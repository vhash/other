#### keepalived故障切换配置
官方文档 <https://keepalived.org/manpage.html>  
参考文档 <https://www.cnblogs.com/clsn/p/8052649.html>  
keepalived可以实现主备服务器故障切换转移、负载均衡。  

##### 以下为CentOS7.5的安装和配置
安装`yum install keepalived`
配置文件 `/etc/keepalived/keepalived.conf`

主节点`192.168.1.2` 备节点`192.168.1.3` 虚拟ip`192.168.1.4`

主节点配置:
```
global_defs {
   router_id node01 # 节点标识，局域网内唯一
}

vrrp_instance VI_1 {
    state BACKUP # 初始状态 MASTER | BACKUP, 此处只是说明，不重要,但是如果设置了nopreempt,此处必须是BACKUP
    nopreempt # 设置此项的话，当优先级高的节点连接后，不会抢占原来的master节点，否则会自动切换到优先级高的节点
    interface eth0
    virtual_router_id 51 # 同一个集群id一致
    
    priority 150 # 优先级决定是主还是备，**主节点必须比备节点高50**
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111 # 认证，主备必须相同
    }
    virtual_ipaddress {
        192.168.1.4/24 # 虚拟ip，主备必须相同
    }
}
```

备节点配置:
```
global_defs {
   router_id node02 # 节点标识，局域网内唯一
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51 # 同一个集群id一致
    
    priority 100 # 优先级决定是主还是备，**主节点必须比备节点高50**
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111 # 认证，主备必须相同
    }
    virtual_ipaddress {
        192.168.1.4/24 # 虚拟ip，主备必须相同
    }
}
```

启动 `sudo systemctl start keepalived`  
查看是否启动成功 `ip a` 返回:  
```
eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether ff:ff:ff:ff:ff:ff brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.2/24 brd 192.168.1.1 scope global eth0
    ****inet 192.168.1.4/24 scope global secondary eth0:1****
    inet6 ffff::fff:ffff:ffff:ffff/64 scope link 
       valid_lft forever preferred_lft forever
```

问题： 
1. 主节点宕机后没有自动切换到备节点:
可能是防火墙的问题  
VRRP是用IP多播的方式来实现主备之间的通信,默认多播地址（224.0.0.18)，所以需要设置防火墙  
主备节点都运行以下命令  
添加规则 `sudo firewall-cmd --direct --permanent --add-rule ipv4 filter INPUT 0 --in-interface em1 --destination 224.0.0.18 --protocol vrrp -j ACCEPT`  
重载防火墙规则 `sudo firewall-cmd --reload`  

