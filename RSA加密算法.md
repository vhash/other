RSA加密算法是一种非对称加密算法<https://blog.csdn.net/cynophile/article/details/79642498>   
公钥持有者: A和C   
私钥持有者: B

pip install pycryptodome
##### 签名
为了保证消息确实是B发出的，B需要用私钥对消息进行签名   
B —— 明文 ——（计算hash值）—— hash值 ——（使用私钥加密）—— 签名   

A —— 签名 ——（使用公钥解密）—— hash值1 \   
　　　　　　　　　　　　　　　　　　　　　　比对 hash值1和2   
A —— 明文 ——（计算hash值 ）—— hash值2 /   

```
from Cryptodome.PublicKey import RSA
import base64
from Cryptodome.Signature import PKCS1_v1_5
from Cryptodome.Hash import SHA256

def gen_sign(message, pri_key):
    if pri_key.startswith('-----'):
        rsa_key = RSA.importKey(pri_key)
    else:
        rsa_key = RSA.importKey(base64.b64decode(pri_key))

    signer = PKCS1_v1_5.new(rsa_key)
    # 使用SHA256算法计算hash值
    hash_obj = SHA256.new(message.encode())
    signature = signer.sign(hash_obj)
    # base64 编码，转换为unicode表示并移除回车
    sign = base64.b64encode(signature).decode().replace("\n", "")
    return sign

def verify_sign(message, pub_key, sign):
    if pub_key.startswith('-----'):
        rsa_key = RSA.importKey(pub_key)
    else:
        rsa_key = RSA.importKey(base64.b64decode(pub_key))

    verifier = PKCS1_v1_5.new(rsa_key)
    # 使用SHA256算法计算hash值
    h = SHA256.new(message.encode())
    return verifier.verify(h, base64.b64decode(signature.encode()))
```
##### 加解密
A给B发消息，为了保证消息内容不泄露，A需要用B的公钥对消息进行加密   
B给A发消息的话，不能用B的私钥加密，因为C手里的公钥能解密消息。所以B给A发消息必须用A的公钥   

A —— 明文 ——（使用B的公钥加密）—— 密文   

B —— 密文 ——（使用私钥解密）—— 明文   

```
from Cryptodome import Random
from Cryptodome.PublicKey import RSA
import base64
from Cryptodome.Cipher import PKCS1_v1_5 as Cipher_PKCS1_v1_5

def encrypt(message, pub_key):
    if pub_key.startswith('-----'):
        rsa_key = RSA.importKey(pub_key)
    else:
        rsa_key = RSA.importKey(base64.b64decode(pub_key))

    cipher = Cipher_PKCS1_v1_5.new(rsa_key)
    return base64.b64encode(cipher.encrypt(message.encode())).decode()

def decrypt(encrypt_text, pri_key):
    if pri_key.startswith('-----'):
        rsa_key = RSA.importKey(pri_key)
    else:
        rsa_key = RSA.importKey(base64.b64decode(pri_key))

    cipher = Cipher_PKCS1_v1_5.new(rsa_key)
    return cipher.decrypt(base64.b64decode(encrypt_text), Random.new().read(10)).decode()
```

##### 注意
> 如果我们把`message`字符串的长度增加到很长，例如1M，这时，执行RSA加密会得到一个类似这样的错误：`data too large for key size`，这是因为RSA加密的原始信息必须小于Key的长度。那如何用RSA加密一个很长的消息呢？实际上，RSA并不适合加密大数据，而是先生成一个随机的AES密码，用AES加密原始信息，然后用RSA加密AES口令，这样，实际使用RSA时，给对方传的密文分两部分，一部分是AES加密的密文，另一部分是RSA加密的AES口令。对方用RSA先解密出AES口令，再用AES解密密文，即可获得明文。
