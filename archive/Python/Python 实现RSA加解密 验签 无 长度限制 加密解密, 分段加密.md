# 系列文章目录


---

**Table of Contents**

- [系列文章目录](#系列文章目录)
- [为什么要分段加密](#为什么要分段加密)
- [代码示例](#代码示例)
- [测试](#测试)
- [测试结果](#测试结果)

---


# 为什么要分段加密

加密的字段长短规则如下：

加密的 plaintext 最大长度是 证书key位数/8 - 11, 例如1024 bit的证书，被加密的串最长 1024/8 - 11=117,

那么对于 2048bit的证书，被加密的长度最长2048/8 - 11 =245,

解决办法是 分块 加密，然后分块解密就行了，
因为 证书key固定的情况下，加密出来的串长度是固定的。
也就是说，如果使用2048bit的证书，并且被加密的字符段是小于245个，那么被加密出来的字符长度是344个，以此类推，被加密的字符串可以是688个，1032个等。

# 代码示例

```python
from Crypto.Cipher import PKCS1_v1_5 as Cipher_pkcs1_v1_5
from Crypto.Signature import PKCS1_v1_5

from Crypto.PublicKey import RSA
from Crypto.Hash import SHA256
import base64


class RsaUtil:

    def __init__(self, pub_key, pri_key):
        self.pri_key_obj = None
        self.pub_key_obj = None
        self.verifier = None
        self.signer = None
        if pub_key:
            pub_key = RSA.importKey(base64.b64decode(pub_key))
            self.pub_key_obj = Cipher_pkcs1_v1_5.new(pub_key)
            self.verifier = PKCS1_v1_5.new(pub_key)
        if pri_key:
            pri_key = RSA.importKey(base64.b64decode(pri_key))
            self.pri_key_obj = Cipher_pkcs1_v1_5.new(pri_key)
            self.signer = PKCS1_v1_5.new(pri_key)

    def public_long_encrypt(self, data, charset='utf-8'):
        data = data.encode(charset)
        length = len(data)
        default_length = 117
        res = []
        for i in range(0, length, default_length):
            res.append(self.pub_key_obj.encrypt(data[i:i + default_length]))
        byte_data = b''.join(res)
        return base64.b64encode(byte_data)

    def private_long_decrypt(self, data, sentinel=b'decrypt error'):
        data = base64.b64decode(data)
        length = len(data)
        default_length = 128
        res = []
        for i in range(0, length, default_length):
            res.append(self.pri_key_obj.decrypt(data[i:i + default_length], sentinel))
        return str(b''.join(res), encoding = "utf-8")

    def sign(self, data, charset='utf-8'):
        h = SHA256.new(data.encode(charset))
        signature = self.signer.sign(h)
        return base64.b64encode(signature)

    def verify(self, data, sign,  charset='utf-8'):
        h = SHA256.new(data.encode(charset))
        return self.verifier.verify(h, base64.b64decode(sign))

```

# 测试

```python
from rsa_util import RsaUtil


data = "{\"0\":\"0\",\"1\":\"1\",\"10\":\"10\",\"11\":\"11\",\"12\":\"12\",\"13\":\"13\",\"14\":\"14\",\"15\":\"15\",\"16\":\"16\",\"17\":\"17\",\"18\":\"18\",\"19\":\"19\",\"2\":\"2\",\"3\":\"3\",\"4\":\"4\",\"5\":\"5\",\"6\":\"6\",\"7\":\"7\",\"8\":\"8\",\"9\":\"9\"}"

pub_key = '''
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCQFA4YVQZatJAyO7TsuzkWE8dz
17qi8GuOCnegKbKd6alLXkDzKhVG3kd3GijouHtlqsm2zFCK7K+I5MUu8Fuk23OE
wIVZn9StltjLzJ1hB1AZC1/NCoCFZG5T2+AaQolrw8LvPS5jH2TuYQf7oLDHR88B
KJgV/tZlr22Jicqm0wIDAQAB
'''
pri_key = '''
MIICWwIBAAKBgQCQFA4YVQZatJAyO7TsuzkWE8dz17qi8GuOCnegKbKd6alLXkDz
KhVG3kd3GijouHtlqsm2zFCK7K+I5MUu8Fuk23OEwIVZn9StltjLzJ1hB1AZC1/N
CoCFZG5T2+AaQolrw8LvPS5jH2TuYQf7oLDHR88BKJgV/tZlr22Jicqm0wIDAQAB
AoGAMP6A5IlVRdcNCef/2Fi6SuWi96OuleYHzR+GGnLTiJuCtFxy3b27yoOf7cJ5
ktnZLHNtcLn90aA2+OhCnXmiz+M9PNArzfvtDoAKMlM9UEpBjGW/QYPkcHgnKOs9
utAr4OnPB9PFdvCuwya4P8AL/7kpjSW+4zQpUT459BlJFxECQQDYUnQQgyR3CZiG
Pj9vPfmmFmogpZpJTG9zAuOjOCxa5BQvV4iKhk6pkQAaVsjc7WMobEIhLqXn/I8E
ldsqIPj1AkEAqoFZULpjke8CQm0rmr2UdbhU74KKYzeS2KKKc/2TdQUzTqvBdY2+
VCyc0Ok6BWctBHfsu4FR6YpDYsg3QwvjpwJAEHeuaDdjhkBPwSBp+dDw+UjJiXSx
2xSbg1jb9WfoUH7+XmA+f7UbteLY7ChhIBheLQyYuCfx70gVpxa1WW6rJQJAEahR
mpWi6CMLZduub1kAvew4B5HKSRohQAQdOIPjOHQwaw5Ie6cRNeBk4RG2K4cS12qf
/o8W74udDObVKkFZ8wJAPL8bRWv0IWTlvwM14mKxcVf1qCuhkT8GgrG/YP/8fcW8
SiT+DifcA7BVOgQjgbTchSfaA+YNe7A9qiVmA+G4GQ==
'''

rsa_util = RsaUtil(pub_key, pri_key)
print(f'原文: {data}')

encrypt = rsa_util.public_long_encrypt(data)
print(f'加密: {encrypt}')

decrypt_str = rsa_util.private_long_decrypt(encrypt)
print(f'解密: {decrypt_str}')

sign = rsa_util.sign(data)
print(f'sign: {sign}')

verify = rsa_util.verify(decrypt_str, sign)
print(f'verify: {verify}')

```

# 测试结果

```python
原文: {"0":"0","1":"1","10":"10","11":"11","12":"12","13":"13","14":"14","15":"15","16":"16","17":"17","18":"18","19":"19","2":"2","3":"3","4":"4","5":"5","6":"6","7":"7","8":"8","9":"9"}
加密: b'BXzccjfWEyF061Beh+bhqi0P88jMeB3eI84/mKgNVrVQFIh6wa209xzQLRCfoVJPN16T1T8jtORisdLXFTmdgH2cXHiorIJfYo+y64aJ8jg1hGe2zmUfFws2WcKttTGluhI8x3xbO2wZi1aHS8k8uxUR0bAWu+LHWcBABqGD3DUQf0AJ/graML1G1hIVYVd0JG6+wZnnB5o6wLrViAh7T2TiVtRVzA4dZ+z73mibDzSohHXA5zDb1XAO0RgATBnpMda0krMgZPOVZHhBRlz6PvUcYcv1+kUTsE/bqTDXOPiQJ2H7bnvTAA2ApoM5CRYFAuhKs5FHSHXhOc6K1uOHpQ=='
解密: {"0":"0","1":"1","10":"10","11":"11","12":"12","13":"13","14":"14","15":"15","16":"16","17":"17","18":"18","19":"19","2":"2","3":"3","4":"4","5":"5","6":"6","7":"7","8":"8","9":"9"}
sign: b'Ho2uhOjKx4GRXulOq/x/owEB01rzztc7WmxTDxjxUzDf7U1cNsp1R7+ahwQuUnGR+lL8RX8G9PxYW/aofc0ApFqIanzEhxbVMsVk87V4M+a/ZEXOJuAgw/0U8IauuMtHl+zaI0KY+TDgrx1rXq5razqU2sU0Ps+y+ILytEK723k='
verify: True
```