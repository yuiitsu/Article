# Python微信APIv3签名

### 1. 主要代码

```python
import time
import random
from Cryptodome.PublicKey import RSA
from Cryptodome.Signature import pkcs1_15
from Cryptodome.Hash import SHA256
from base64 import b64encode

mchid = '商户ID'
serial_no = '商户API证书序列号'
private_key = "-----BEGIN PRIVATE KEY-----\n" \
              "NIIEvgIBADANBgkqhkiG9w0B3QEFAA5CBKgwggSkAgEAAoIBAQC2l/hfX5g6qjo7" \
              "BcqTa+LldQpd7m4ZZEKT2BxUpYPNJ/YoZO1c3qB/+CfQQlKJYPseOsTnUeuAZ2AG" \
              "z2LWuh/36gmiFX2Nw1pI0AkVwec8pB1+IqLGHzRsAaDJ++2J88TcWOuM5lOcBr6I" \
              "VJOyl1ghYB4Dnuk2uid46rAZwWL3L2FZ+ktfdtePt4zOQrpILRuDNzR6TOcd1JER" \
              "ylt8iDyoVTr69ALnQdhoNnofnrBK3R4sY5ON1BR01DpZM8yX8mmb6yOLjp1VoP+5" \
              "n8pCPVAqZJEbwfDH90naeMZLYfoCvdaJhxHLblviZN7v4VW6Fx/W43O9sRTNPukU" \
              "W5pLBP57AgMBAAECggEBAI9U6iZLzx62A7HTUPq6ZMkEQB2OEyUhe9W8fjjAGJ9R" \
              "8DwzRdRx+gGaVf54IXwvwdAwB+MhfkE0ZL/Tyd2PC4s7j0ZJol5G7Ddd/tOye4cx" \
              "uOkL3USyuB7UhFgpx4RT88OYlYbsQtOmw6gW5D376bWBUu46rw1Dwbp8V7JQCRTJ" \
              "K+vbMj1ytgP10fg/hTQNe7RWmd+KC+OGZbBH6MxybLoR+BOwQQpwGEgoEoY9Rmlm" \
              "E/8poPvdY/PLIaS2ca/rsgJ/ZhiSx6BL0DAqVkKtM9U9T0LRoZmJcCp6YgLKGA0X" \
              "BrnbMXczFHFVfjmK1jw4UqWbPICsrbhF4dtIuSqZbDECgYEA4M8J4T/grNaNhQTc" \
              "x6q/ylFZB3I8Qoih2YeLx7/JOBEL66jNJhHVrgyF8JChlL0J8RGMg8KdxzZ0OTEw" \
              "Q5ZK4mo93cxwqFh87X4KUNaicNveCsxuEL80Bbhst7NmJKi5T2VQMR9kFd+eTwx8" \
              "c5aOHX5W4RIogKyNHqvCdEqHohMCgYEAz+1/NfRtqI2F68olrtdlbReaHtlCZxRJ" \
              "3XhaIT5OGjXE/0upRRx6EgDh4f+wvqpFdlutPHFTSz4H/5rBGyZ7NJ5D44WO3Z7K" \
              "SdY3YJ+5J6Nu+2OX6FGZWmqyW9kgYRthzClqNduBWbgu0gRC8R8wLg1W7xe9HN5e" \
              "bc0HnhnBfvkCgYBSNMNbH/2rlkVn1/BX/yNk+zxAEdDhT49HuV4u6/3Lx8gBI9fo" \
              "zOrDW4b7AhhkCICDK7SjVd5WQ55ab5dDj8jQZK384Q5tMPZ17fodt27tMClQ75Js" \
              "A08lrFvtDOg6DbK9ysF5RQ5XRU9hfqJfrjVHqbRhVz+CVhbAmXRhDAPvC5KBgQCX" \
              "kXuCvCvXi1qNF+1CN3eS/3p0dFEITOzPSXUB+KX8SyfQJbo9S9XcG9KM6NNRGVPL" \
              "RGbSwZVDKvOvqoKLpRB4ucmpJ+mNubuh+Uqi36ubrnIvRFkum5TbKR3dADivMMOo" \
              "jKQEoH75BN70bvDRTbfUShsN8ZAA5EQXbDbaU9IOGQKBgEr+xQMksLCif8GrwlfJ" \
              "Vrf3uyXTyY1QVRAGy9UIiM0RnsWDaGcRgNaz1K9YV3gqm7qHe2y8fzB0PLLw2TzY" \
              "GUwxIttKpnEVlX1M59nmGFmQq08YNC+JRHZrvmmB0MyEBSA9lg0ugASdQQC5PGYa" \
              "Rnn7p8VlUEauVmFpF6BTVZLQ\n" \
              "-----END PRIVATE KEY-----"
timestamp = str(int(time.time()))
nonce_str = random.randint(100000, 10000000)

def sign_str(method, url_path, request_bdy):
  """
  生成欲签名字符串
  """
	sign_list = [
    method,
    url_path,
    timestamp,
    nonce_str,
    request_body
  ]
  return '\n'.join(sign_list) + '\n'


def sign(sign_str):
  """
  生成签名
  """
	rsa_key = RSA.importKey(private_key)
	signer = pkcs1_15.new(rsa_key)
	digest = SHA256.new(sign_str.encode('utf8'))
	sign = b64encode(signer.sign(digest)).decode('utf8')
  return sign


def authorization(method, url_path, request_body):
  """
  生成Authorization
  """
  sign_str = sign_str(method, url_path, request_body)
  sign = sign(sign_str)
  authorization = 'WECHATPAY2-SHA256-RSA2048  ' \
            'mchid="{mchid}",' \
            'nonce_str="{nonce_str}",' \
            'signature="{sign}",' \
            'timestamp="{timestamp}",' \
            'serial_no="{serial_no}"'.\
            format(mchid=mchid,
                   nonce_str=nonce_str,
                   sign=sign,
                   timestamp=timestamp,
                   serial_no=serial_no
                  )
  return authorization


if __name__ == '__main__':
  print(authorization('POST', '/v3/marketing/favor/users/openid/coupons'), '{"stock_id":"123","stock_creator_mchid":"1302430101","out_request_no":"20190522_001","appid":"your appid"}')
  # Authorization WECHATPAY2-SHA256-RSA2048 mchid="1312030806",nonce_str="f0wwnSIuQN8yDr0U4bYKNmUgALcMUCLM",signature="dcFTPfaAewd+UXuXv+VA+KeGW1coUG68PtklWtsMiFGal5GxiljGUVGV60gBnIo2La1R3cxf7mOb62q7xoab9mP1SZ5dP8L+amQ9vyl+ZYTaJOg31vtkDwMU0ILNqy96SuKy+5/Q2NSCQU0fBLMWU11vbSoA2ycEsCjDEknc8Hiw+vyKkV6iGyUNBMizfwZhJWdRcWDWeyxAy0rsaZKVOpeEyJ2xPQnLX8uB+gqCIO5+vE8KYjseXPGun+Zr6i5gl7i0O/BdBfY4BDRAZsrF5v7LikptEbRwJ8+1IevIT5LaUc5J5BnGM009BuzsZzK8cphhKvepVmA8Gy0gWvfDeA==",timestamp="1592375315",serial_no="14DFFAAAA79AF4BD56CC1O55E06246E95D3PAAS0"
```

### 2. 坑

- 私钥，如果不是直接读取文件，使用字符串形式，需要保持格式，在`-----BEGIN PRIVATE KEY-----`后和`-----END PRIVATE KEY-----`之前需要加上`\n`。不然容易报`valueerror: rsa key format is not supported`或是`ValueError: Not a valid PEM pre boundary`.
- **生成欲签名字符串时，最后一定要加上`\n`，不然永远是签名错误。即使`request_body`为空，也得加上。**
- 因为headers参数为Authoriaztion，如果Python使用的http client带有auth_username和auth_password或类似认证参数，切记这两项不要有值，一般为None，即使传空，也不行，因为http client会创建一个Authoriaztion，然后不管怎么修改headers里的Authoriaztion都会得到报错信息：`Http头Authoriaztion认证类型不正确`

