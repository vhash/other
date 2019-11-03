#### yield用法
参考链接：<https://blog.csdn.net/mieleizhi0522/article/details/82142856>

#### supervisor安装和配置
supervisor用来管理进程  
官方文档: <http://supervisord.org/configuration.html>  
参考：<https://www.liaoxuefeng.com/article/895919885120064>

##### CentOS7.5安装
`yum install supervisor`  

在`/etc/supervisord.d/`中添加需要管理的进程配置文件：  
比如需要管理的是一个gunicorn启动的python app(gunicorn配置需要daemon为False)  
则在`/etc/supervisord.d/`中新建文件test.ini  
```
[program:test]
autostart=true # 当supervisor启动时，启动该进程
autorestart=true # 该进程异常退出时，supervisor重启该进程
directory=/home/test/test/backend # 在哪个目录执行命令
command=/home/test/.pyenv/versions/test/bin/gunicorn manage:app -c app.guni
user=test
```

##### supervisordctl常用命令
help # 查看帮助  
status # 查看程序状态  
stop program_name # 关闭 指定的程序  
start program_name # 启动 指定的程序  
restart program_name # 重启 指定的程序  
tail -f program_name # 查看 该程序的日志  
update # 重启配置文件修改过的程序（修改了配置，通过这个命令加载新的配置)  
