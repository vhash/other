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
