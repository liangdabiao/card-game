---
layout: post
title: 面向属性基加密的 X.509 身份
date: 2023-11-19 00:00:00.000000000 -00:00
permalink: identity-for-attribute-based-encryption/
author: Benny Cheung
artwork:
  author: Dmitriy Bragin
tags:
  - PKI
  - Identity
  - Cryptography
  - Attribute-based Encryption
  - Python
category: post
comments: true
image: images/identity-for-attribute-based-encryption/cover2x.jpg
images:
  cover: images/identity-for-attribute-based-encryption/cover.jpg
  header: images/identity-for-attribute-based-encryption/header.jpg
published: true
series: "Blockchain Healthcare"
series_order: 3
---

<!--excerpt.start-->
在物理世界，我们信任知名组织——包括政府——颁发的身份证件。验证过程是对卡片真实性的目视检查。进入数字领域，我们依赖公钥基础设施（PKI）来安全地识别参与者。这正是支撑当今互联网商业的关键密码技术！再与先进的属性基加密相结合，我们将看到如何用 PKI 身份支撑对受保护记录的灵活访问控制。
<!--excerpt.end-->

----
> [2024/10/05] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/identity-for-attribute-based-encryption/Podcast_Personal_Identity_ABE.mp3
{% include open-embed.html %}
----

![医疗区块链]({{ site.baseurl }}/images/identity-for-attribute-based-encryption/ManyFaces_Mosaicos_x750.jpg) 

*图：Kandinsky 艺术风格迁移到众多面孔的图像*

我们之前的文章 ["Attribute-based Encryption for Healthcare Blockchain"]({{ site.baseurl }}/attribute-based-encryption-for-healthcare-blockchain)
讲解了如何把属性基加密（ABE）应用于电子健康记录（EHR），
帮助规划安全系统实现的路线图。
不过，由于聚焦 ABE，我们只用一小段话触及了参与者的身份与属性。
我们假定了读者具备公钥基础设施（PKI）、X.509 证书和证书颁发机构（CA）的知识。
此外，应用层属性被"神奇地"注册进了 X.509 证书。
参与者的身份与属性受到完整保护与认证，即没有人能伪造身份与属性。现在是时候更详细地解释这一密码学组件了。

继续我们的探索，我们将采用实践路线：演示如何用 Python `cryptography` 生成带自定义属性的 X.509 证书；随后，我们使用 `charm-crypto` 框架的混合适配器，配合 X.509 自定义属性执行 CP-ABE（密文策略）。

## Python 安装

### 虚拟环境
使用隔离的 Python 虚拟环境可以让你免于安装带来的头痛与灾难。
`crypto`（或你自己起的名字）是虚拟环境的名称，`python=3.5` 是 Python 版本。

```bash
conda create -n crypto python=3.5
```

按 y 继续。这会把 Python 版本及所有相关的 `anaconda` 打包库安装到 `{path_to_anaconda_location}/envs/crypto

然后用以下命令激活虚拟环境，

```
source activate crypto
```

### cryptography 模块
我们的实验从 [cryptography](https://cryptography.io/en/latest/) 模块开始，
`cryptography` 既包含高层配方，也包含常用密码算法（对称密码、消息摘要、密钥派生函数）的低层接口。我们依赖 `cryptography` 的许多特性来演示 X509 证书处理。

确保 `cryptography` 安装在 virtualenv 内。

```
pip install cryptography
```

### Charm 框架
我们的实验使用基于 [charm-crypto](http://charm-crypto.io/) 框架的属性基加密 Python 实现 [[AGM13]](#AGM13)。
Charm 为快速构建高级密码系统原型而设计。它是一个工程良好的框架，采用混合设计：性能密集的数学运算用原生 C 模块实现，而密码系统本身用可读的高级语言编写。Charm 还提供了许多新组件，方便快速开发新方案与协议。我们就是这样做 CP-ABE 实验的。

从源码安装 Charm 很直接。首先，确认你已安装以下依赖：
* [GMP 5.x](http://gmplib.org/)
* [PBC](http://crypto.stanford.edu/pbc/download.html) 
* [OPENSSL](http://www.openssl.org/source/)

确保 Charm 从 virtualenv 安装。

注意：`--enable-darwin` 用于 MacOS 安装

```
./configure.sh --enable-darwin
make install
```

验证安装是否可用，
```
make test
```

## 公钥基础设施
理解公钥基础设施（PKI）标准将帮助我们设计各种网络参与者之间的安全通信。PKI 使我们能够为参与者提供身份，并确保系统上的消息得到恰当的认证。

PKI 有四个关键要素：
* 公钥与私钥
* 证书颁发机构
* 数字证书
* 证书吊销列表（*本文不讨论*）

![PKI 中的身份]({{ site.baseurl }}/images/identity-for-attribute-based-encryption/Identity_PKI.jpg)
*图片来源：Hyperledger Fabric [文档](https://hyperledger-fabric.readthedocs.io/en/release-1.4/identity/identity.html)* 

从证书颁发机构（CA）获取证书时，通常的流程是：

* 生成私钥/公钥对。
* 创建证书请求（CSR），用你的密钥签名（以证明你拥有该密钥）。
* 你把 CSR 交给 CA（而不是私钥）。
* CA 验证你拥有想申请证书的资源（例如域名和/或属性）。
* CA 给你一张由其签名的证书，其中标识了你的公钥，以及你通过认证的资源。

我们将在以下各节用实验性的 Python 代码来理解上述各个组件。

### 认证、公钥与私钥
认证与消息完整性是安全通信中的重要概念。认证要求交换消息的各方确信特定消息创建者的身份。消息具有"完整性"意味着它在传输过程中未被修改。

PKI 认证机制依赖数字签名，让参与者能对其消息做数字签名。数字签名保证被签名消息的完整性，并表明消息来自特定参与者而非他人。
在公钥密码学中，数字签名机制要求各方持有两个密码学上相互关联的密钥：一个广泛公开、充当认证锚点的公钥，以及一个用于对消息产生数字签名的私钥。数字签名消息的接收者可以验证所收消息的来源与完整性：检查所附签名在预期发送者的公钥下是否有效。

私钥与相应公钥之间唯一的关系，是让安全通信成为可能的密码学魔法。密钥之间唯一的数学关系是：私钥可用于对消息产生签名，而只有相应的公钥才能匹配该签名，且只对同一条消息成立。

如果我们想从典型的 CA 获取证书，需要生成一个私钥——RSA 密钥（这是目前网络上最常见的密钥类型）：

```python
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import rsa

# Generate our key
key = rsa.generate_private_key(
  public_exponent=65537,
  key_size=2048,
  backend=default_backend()
  )

# Write our key to disk for safe keeping
# Alternative, encryption_algorithm=serialization.BestAvailableEncryption(b"passphrase")
# but this required password when the private key is used, not ideal for automation.
with open("key.pem", "wb") as f:
  f.write(key.private_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PrivateFormat.TraditionalOpenSSL,
    encryption_algorithm=serialization.NoEncryption(),
  ))
```

私钥以 PEM 文件格式保存到 `key.pem`。我们应妥善保管并保护这个文件。这个文件不应传输到任何其他地方。下一步是创建证书请求（CSR），由私钥签名。 

### 证书颁发机构
如前所述，参与者可以通过由系统信任的机构颁发的数字身份参与安全系统。在最常见的情况下，数字身份（或简称身份）的形式是经过密码学验证、符合 X.509 标准并由证书颁发机构（CA）颁发的数字证书。

证书颁发机构向不同参与者发放证书。这些证书由 CA 数字签名，并把参与者的*公钥*（以及可选的一组属性，我们稍后再谈）绑定在一起。因此，如果信任该 CA（通过知晓其公钥），就可以在验证 CA 对参与者证书的签名之后，信任该特定参与者与证书所含公钥绑定，并拥有其中所含的属性。

![PKI 中的身份]({{ site.baseurl }}/images/identity-for-attribute-based-encryption/Certificate_Authority.jpg)
*图片来源：Hyperledger Fabric [文档](https://hyperledger-fabric.readthedocs.io/en/release-1.4/identity/identity.html)*

如果你已经生成了*私钥*（在上一步），我们可以用 `load_pem_private_key()` 加载它。
接下来我们需要生成证书签名请求。一份典型的 CSR 包含几个细节：

* 关于我们公钥的信息（包括整个主体的签名）。
* 关于我们是谁的信息。
* 关于这张证书用于哪些域名的信息。

```python
from cryptography import x509
from cryptography.x509.oid import NameOID
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives import serialization

# load the saved private key
pem_key = open('key.pem', 'rb').read()
key = serialization.load_pem_private_key(pem_key, password=None, backend=default_backend())

# Generate a CSR
csr = x509.CertificateSigningRequestBuilder().subject_name(x509.Name([
# Provide various details about who we are.
  x509.NameAttribute(NameOID.COUNTRY_NAME, u"CA"),
  x509.NameAttribute(NameOID.STATE_OR_PROVINCE_NAME, u"Ontario"),
  x509.NameAttribute(NameOID.LOCALITY_NAME, u"Toronto"),
  x509.NameAttribute(NameOID.ORGANIZATION_NAME, u"Medical Record Company"),
  x509.NameAttribute(NameOID.COMMON_NAME, u"medicalrecord.com"),
  ])).add_extension(
    x509.SubjectAlternativeName([
      # Describe what sites we want this certificate for.
      x509.DNSName(u"www.medicalrecord.com"),
    ]),
  critical=False,
  # Sign the CSR with our private key.
  ).sign(key, hashes.SHA256(), default_backend())

# Write our CSR out to disk.
with open("csr.pem", "wb") as f:
  f.write(csr.public_bytes(serialization.Encoding.PEM))
```

CSR 以 PEM 文件格式保存到 `csr.pem`。下一步是把 CSR 发给 CA（而不是私钥）。
CA 验证资源所有权（例如域名和/或属性），然后签名并颁发所请求的证书。

### 数字证书——X.509
在 EHR 场景中，每个希望与系统交互的参与者都需要一个身份。可以使用一个或多个 CA 从数字视角定义参与者。正是 CA 为参与者获得可验证的数字证书提供了基础。数字证书是一份文档，包含与参与者相关的一组属性。最常见的证书类型是符合 X.509 标准的证书，它允许在证书结构中编码参与者的身份细节。

证书可以被广泛传播，因为它们不包含任何私钥。因此它们可以用作信任锚点，认证来自不同参与者的消息。CA 自己也有证书，并被广泛公开。这让其他验证方 CA 能够验证参与者的证书。证书连同全部属性由 CA 数字签名，因此任何篡改都会使证书失效。把 X.509 证书想象成一张不可更改的数字身份证。

把之前生成的 CSR `csr.pem` 移到 CA 一侧，CA 需要验证然后签名它，生成正式的证书。

```python
from cryptography import x509
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import hashes, serialization
import datetime
import uuid

pem_csr = open('csr.pem', 'rb').read()
csr = x509.load_pem_x509_csr(pem_csr, default_backend())

# using internal CA certificate and private key
pem_cert = open('ca.crt', 'rb').read()
ca = x509.load_pem_x509_certificate(pem_cert, default_backend())
pem_key = open('ca.key', 'rb').read()
ca_key = serialization.load_pem_private_key(pem_key, password=None, backend=default_backend())

# should perform CSR validation
builder = x509.CertificateBuilder()
builder = builder.subject_name(csr.subject)
builder = builder.issuer_name(ca.subject)
builder = builder.not_valid_before(datetime.datetime.now())
builder = builder.not_valid_after(datetime.datetime.now() + datetime.timedelta(30)) # 30 days
builder = builder.public_key(csr.public_key())
builder = builder.serial_number(int(uuid.uuid4()))
for ext in csr.extensions:
    builder = builder.add_extension(ext.value, ext.critical)

certificate = builder.sign(
    private_key=ca_key,
    algorithm=hashes.SHA256(),
    backend=default_backend()
)

with open('my.crt', 'wb') as f:
    f.write(certificate.public_bytes(serialization.Encoding.PEM))

```

用 `openssl` 打印 `my.crt`——X.509 证书文件——的文本版本，

```
openssl x509 -text -noout -in my.crt
```

给定 X.509 证书的文本输出应形如下，注意 `Subject` 部分定义了证书持有者的标准属性。

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            e1:99:7e:e7:1d:6d:48:b9:90:a4:f0:9f:8e:60:cc:e1
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=CA, ST=Ontario, O=Jonah Group Ltd., OU=Consulting, CN=Jonah Group CA/emailAddress=bcheung@jonahgroup.com
        Validity
            Not Before: Aug 18 09:35:58 2020 GMT
            Not After : Sep 17 09:35:58 2020 GMT
        Subject: C=CA, ST=Ontario, L=Toronto, O=Medical Record Company, CN=medicalrecord.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:b6:a3:22:fb:20:4c:70:e3:94:4a:bd:ae:0f:b8:
                    fe:61:2f:37:b8:a7:22:f5:6c:1e:bd:84:0a:0d:bf:
                    b8:0e:92:a5:6e:69:bc:99:14:c7:3d:77:8d:a3:ea:
                    b7:be:f9:e4:86:e2:2d:c0:3f:27:38:92:e0:45:98:
                    61:7f:0e:65:b2:02:11:db:d8:00:6d:cc:fa:57:43:
                    09:d0:27:af:12:a4:f6:0d:de:74:a6:c3:20:00:6b:
                    cd:d8:f6:5c:2c:3a:d1:e0:7d:cf:f5:40:80:33:e6:
                    ad:cb:3d:26:f9:87:41:d8:e2:cb:80:52:c1:27:80:
                    ca:98:68:93:b8:7e:4e:04:b7:4e:fc:fb:e5:63:8c:
                    c1:62:b8:76:62:d2:5d:33:7e:1c:35:12:cd:ba:17:
                    e6:9e:1e:f9:3c:54:8c:96:bc:18:e9:a1:27:ae:b2:
                    70:6e:be:c3:41:5b:8f:0d:f7:7d:25:42:6d:c3:5c:
                    9a:2e:4b:4a:2c:ff:77:ec:d3:e6:06:03:dc:0d:f4:
                    26:51:b3:16:2a:93:ba:b2:88:9a:71:d1:0f:6c:d1:
                    ff:87:4c:8c:50:eb:1e:29:69:e0:48:14:e5:89:df:
                    5e:ee:40:2a:ad:83:b1:7d:06:90:3e:1d:54:60:4c:
                    ea:83:cc:bc:71:d7:bd:92:c6:67:71:ac:16:d0:7d:
                    7d:2b
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Alternative Name: 
                DNS:www.medicalrecord.com
    Signature Algorithm: sha256WithRSAEncryption
         7a:cc:9e:75:74:aa:fb:8f:4c:16:70:bf:f2:61:1c:40:6c:9f:
         44:82:33:be:76:b0:eb:b2:74:61:04:36:ed:ce:a1:c8:d0:e6:
         51:cb:14:4a:50:a2:f7:f7:3f:97:ef:25:8c:39:e9:1e:7a:8a:
         10:06:49:05:6c:70:e6:cb:61:cd:f4:91:5b:7b:cb:4a:71:f9:
         11:49:e2:da:55:c7:17:89:65:29:9e:d0:2a:79:34:e7:79:02:
         9e:c4:70:88:5a:f4:d8:53:aa:d5:67:47:45:32:2e:0b:c7:45:
         d1:b3:08:f7:0d:2c:33:94:65:5a:91:43:e9:b1:4a:6a:8e:b7:
         6d:b4:8b:b8:53:c9:fb:e1:fd:88:1c:85:00:3b:db:eb:5a:96:
         06:a0:fa:89:70:ff:c6:27:29:1b:5d:e0:8e:93:05:47:9e:3e:
         c6:2a:51:f9:d0:10:a4:26:d4:84:96:1a:6a:bd:9f:f7:a5:f2:
         3f:17:5d:58:4c:79:87:b4:0c:9a:a5:43:72:cb:9a:e1:40:07:
         4d:f7:42:c0:36:e9:b4:e9:2f:6e:f2:6b:52:93:54:28:7e:0f:
         67:bc:f0:98:30:c1:44:32:9f:1f:d2:81:43:41:30:9c:4e:57:
         48:8a:e2:8a:0b:c9:7a:95:55:7d:a1:fc:15:39:33:c2:a1:8a:
         c7:91:3c:77
```

## 带自定义属性的自签名证书
大多数时候我们想要一张由 CA 签名的证书，以建立信任。
但有时我们想创建自签名证书。自签名证书不是由 CA 颁发的，
而是由与其内嵌公钥相对应的私钥签名，即自签名。

这意味着别人不信任这些证书，但也意味着它们可以非常容易地颁发。
一般而言，自签名证书的唯一用例是测试：我们不需要任何人信任我们的证书。接下来就是带着自定义属性做 CP-ABE 实验的方式。

在下面的示例代码中，我们在证书里加入了一个 X.509 非标准扩展 `1.2.3.4.5.6.7.8.1`，其 JSON 值为 `{"Role":"PATIENT", "ID":"123456"}`，用以编码自定义属性。然后我们生成证书本身：

```python
from cryptography import x509
from cryptography.x509.oid import NameOID
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives import serialization
import datetime

# load the saved private key
pem_key = open('key.pem', 'rb').read()
key = serialization.load_pem_private_key(pem_key, password=None, backend=default_backend())

# Various details about who we are. For a self-signed certificate the
# subject and issuer are always the same.
subject = issuer = x509.Name([
  x509.NameAttribute(NameOID.COUNTRY_NAME, u"CA"),
  x509.NameAttribute(NameOID.STATE_OR_PROVINCE_NAME, u"Ontario"),
  x509.NameAttribute(NameOID.LOCALITY_NAME, u"Toronto"),
  x509.NameAttribute(NameOID.ORGANIZATION_NAME, u"Medical Record"),
  x509.NameAttribute(NameOID.COMMON_NAME, u"Amy Grant"),
  ])

cert = x509.CertificateBuilder().subject_name(
  subject
).issuer_name(
  issuer
).public_key(
  key.public_key()
).serial_number(
  x509.random_serial_number()
).not_valid_before(
  datetime.datetime.utcnow()
).not_valid_after(
  # Our certificate will be valid for 10 days
  datetime.datetime.utcnow() + datetime.timedelta(days=10)
).add_extension(
  x509.SubjectAlternativeName([x509.DNSName(u"localhost")]),
    critical=False,
).add_extension(
  x509.BasicConstraints(ca=False, path_length=None), critical=True,
).add_extension(
  x509.UnrecognizedExtension(
    x509.ObjectIdentifier('1.2.3.4.5.6.7.8.1'),
    b'{"Role":"PATIENT", "ID":"123456"}'
  ),
  critical=True,
  # Sign our certificate with our private key
  # don't sign with SHA1() anymore, it is deplicated
).sign(key, hashes.SHA256(), default_backend())

# Write our certificate out to disk.
with open("my_self.crt", "wb") as f:
  f.write(cert.public_bytes(serialization.Encoding.PEM))

```


用 `openssl` 打印 `my_self.crt`——自签名 X.509 证书文件——的文本版本，
可以看到 `Issuer` 与 `Subject` 是同一实体。


```
openssl x509 -text -noout -in my_self.crt
```

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            13:ee:bf:1e:ae:0c:a5:b2:c8:22:25:6a:0f:39:62:dc:b0:fd:2e:4a
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=CA, ST=Ontario, L=Toronto, O=Medical Record, CN=Amy Grant
        Validity
            Not Before: Aug 18 01:25:31 2020 GMT
            Not After : Aug 28 01:25:31 2020 GMT
        Subject: C=CA, ST=Ontario, L=Toronto, O=Medical Record, CN=Amy Grant
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:b6:a3:22:fb:20:4c:70:e3:94:4a:bd:ae:0f:b8:
                    fe:61:2f:37:b8:a7:22:f5:6c:1e:bd:84:0a:0d:bf:
                    b8:0e:92:a5:6e:69:bc:99:14:c7:3d:77:8d:a3:ea:
                    b7:be:f9:e4:86:e2:2d:c0:3f:27:38:92:e0:45:98:
                    61:7f:0e:65:b2:02:11:db:d8:00:6d:cc:fa:57:43:
                    09:d0:27:af:12:a4:f6:0d:de:74:a6:c3:20:00:6b:
                    cd:d8:f6:5c:2c:3a:d1:e0:7d:cf:f5:40:80:33:e6:
                    ad:cb:3d:26:f9:87:41:d8:e2:cb:80:52:c1:27:80:
                    ca:98:68:93:b8:7e:4e:04:b7:4e:fc:fb:e5:63:8c:
                    c1:62:b8:76:62:d2:5d:33:7e:1c:35:12:cd:ba:17:
                    e6:9e:1e:f9:3c:54:8c:96:bc:18:e9:a1:27:ae:b2:
                    70:6e:be:c3:41:5b:8f:0d:f7:7d:25:42:6d:c3:5c:
                    9a:2e:4b:4a:2c:ff:77:ec:d3:e6:06:03:dc:0d:f4:
                    26:51:b3:16:2a:93:ba:b2:88:9a:71:d1:0f:6c:d1:
                    ff:87:4c:8c:50:eb:1e:29:69:e0:48:14:e5:89:df:
                    5e:ee:40:2a:ad:83:b1:7d:06:90:3e:1d:54:60:4c:
                    ea:83:cc:bc:71:d7:bd:92:c6:67:71:ac:16:d0:7d:
                    7d:2b
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Alternative Name: 
                DNS:localhost
            X509v3 Basic Constraints: critical
                CA:FALSE
            1.2.3.4.5.6.7.8.1: critical
                {"Role":"PATIENT", "ID":"123456"}
    Signature Algorithm: sha256WithRSAEncryption
         a0:20:f3:6a:aa:45:a4:00:0c:ba:ce:7f:e1:a3:15:c7:43:19:
         02:ae:71:10:ee:5b:d1:17:a6:ee:fc:ff:0c:b0:25:19:f0:dd:
         da:1b:7c:77:c5:9c:fc:44:0d:84:5d:eb:8d:ef:8d:26:95:2a:
         fc:33:a8:78:89:38:4c:08:0e:c3:f5:8e:46:06:51:1b:d3:0e:
         16:fd:00:99:88:04:f6:67:cf:f8:b8:cd:18:63:59:95:04:00:
         74:49:e5:2d:f7:f2:95:9a:69:ca:64:13:ea:94:c7:7d:0a:ff:
         e1:29:74:71:14:42:49:ec:f7:65:b1:f8:0b:dc:92:c7:e1:37:
         a9:65:44:1b:d0:73:71:f7:b5:4f:cf:56:d3:20:2d:d4:5c:db:
         6b:ce:dc:55:ba:ac:61:ff:0d:53:5b:a2:df:4f:80:6f:78:73:
         93:7d:c7:22:8d:e7:51:8a:ef:15:d9:d3:1c:01:8e:65:c8:77:
         41:95:d4:8d:e0:1f:b3:84:27:c2:03:65:0e:e2:06:ab:5e:3a:
         ae:5c:3e:be:7f:44:fd:20:45:1d:38:d0:0b:0d:8c:d1:62:58:
         a7:fe:77:2e:38:b9:80:da:f2:24:75:a1:c2:04:18:4f:e5:7a:
         63:27:a4:62:06:d7:bf:f4:fe:e0:9f:bc:61:dc:59:e4:d2:03:
         0e:40:8e:ce
```

我们可以通过解码证书扩展来测试，

```python
import json

for ext in cert.extensions:
    print('-'*8)
    print('oid:', ext.oid)
    print('critical:', ext.critical)
    if (ext.oid.dotted_string == '1.2.3.4.5.6.7.8.1'):
        obj = json.loads(ext.value.value.decode('utf-8'))
        print('Custom ABAC (attribute-based access control)')
        print('Role:', obj['Role'])
        print('ID:', obj['ID'])
    else:
        print('value:', ext.value)
```

```
--------
oid: <ObjectIdentifier(oid=2.5.29.17, name=subjectAltName)>
critical: False
value: <SubjectAlternativeName(<GeneralNames([<DNSName(value='localhost')>])>)>
--------
oid: <ObjectIdentifier(oid=2.5.29.19, name=basicConstraints)>
critical: True
value: <BasicConstraints(ca=False, path_length=None)>
--------
oid: <ObjectIdentifier(oid=1.2.3.4.5.6.7.8.1, name=Unknown OID)>
critical: True
Custom ABAC (attribute-based access control)
Role: PATIENT
ID: 123456
```

## 用 Charm 的混合适配器执行 CP-ABE
我们之前来自 ["Attribute-based Encryption for Healthcare Blockchain"]({{ site.baseurl }}/attribute-based-encryption-for-healthcare-blockchain) 的 CP-ABE 示例中有几个问题，都关乎随机 $$GT$$ 消息。读者期望能用 CP-ABE 加密任意明文消息。我们将提供一个扩展示例，说明如何加密任意明文消息并用证书属性解密（参考[加密交换上的答案](https://crypto.stackexchange.com/questions/19310/what-is-msg-group-randomgt-in-charm-encryption-schemes/19312)）。

在真实世界的应用中，属性基加密（ABE）与对称密码结合使用，
因为 ABE 只能加密群元素。在这里就是乘法群 $$GT$$。
用群元素表示文本消息（比特串）时，可表示的比特数是有限的，因为群的大小派生自一个素数。在 Charm 中，很可能是 512 位素数。如果我们有一个把比特串映射到群元素再映射回来的函数，我们也无法表示大于任何群元素的消息。

Charm 在 ABE 语境下提供了**混合适配器**（hybrid adapter）。文本消息用 AES 加密，AES 密钥派生自随机的 $$GT$$ 元素。

以下代码演示如何使用 Charm 的混合适配器（`HybridABEnc`）执行 CP-ABE 方案 [[BSW07]](#BSW07)。
假设我们使用的访问策略为 `((PATIENT and 123456) or (PRACTITIONER and 9876543))`。

* 读取患者证书并解码包含自定义属性的扩展。
* 患者证书的自定义属性为 `['PATIENT', '123456']`，因此患者的密钥拥有满足策略的足够属性。密文被允许解密。

```python
from cryptography import x509
from cryptography.hazmat.backends import default_backend
from charm.toolbox.pairinggroup import PairingGroup
from charm.adapters.abenc_adapt_hybrid import HybridABEnc
from charm.schemes.abenc.abenc_bsw07 import CPabe_BSW07
import json

# preparing the hybrid adapter for CP-ABE
groupObj = PairingGroup('SS512')
cpabe = CPabe_BSW07(groupObj)
hyb_abe = HybridABEnc(cpabe, groupObj)
(pk, mk) = hyb_abe.setup()

access_policy = '((PATIENT and 123456) or (PRACTITIONER and 9876543))'
message = "Personal Secret: high blood pressure and diabetic"
# ct is the encrypted message with the access policy
ct = hyb_abe.encrypt(pk, message, access_policy)

# pretend that we are in the secured server, reading the participant's certificate
pem_cert = open('my_self.crt', 'rb').read()
cert = x509.load_pem_x509_certificate(pem_cert, default_backend())

# fetching the custom attributes stored in the extension 1.2.3.4.5.6.7.8.1
for ext in cert.extensions:
    if (ext.oid.dotted_string == '1.2.3.4.5.6.7.8.1'):
        obj = json.loads(ext.value.value.decode('utf-8'))
        attr = [value for key, value in obj.items()]
        break

# using the patient attributes to generate the secret key
print('patient attributes:', attr)
sk = hyb_abe.keygen(pk, mk, attr)

# using the secret key
# CP-ABE will check if the attributes are sufficient to decrypt the message
hyb_abe.decrypt(pk, sk, ct)
```

如果一切按计划进行，你应该能看到该消息。

```
patient attributes: ['PATIENT', '123456']
b'Personal Secret: high blood pressure and diabetic'
```

Amy 用她自己的 X.509 身份成功解密了她自己的医疗记录！Amy 的健康状况是安全的：只有她自己，以及潜在地、证书中带有 `{"Role":"PRACTITIONER", "ID":"9876543"}` 属性的从业者才能解密。

## 参考文献

* <a name="CRYPTO">[[CRYPTO]](https://cryptography.io/en/latest/x509/)</a> Cryptography online documentation - X.509 module
* <a name="BSW07">[[BSW07]](https://www.cs.utexas.edu/~bwaters/publications/papers/cp-abe.pdf)</a> John Bethencourt, Amit Sahai, and Brent Waters. "Ciphertext-policy attribute-based encryption." In Security and Privacy, 2007. SP'07. IEEE Symposium on, pp. 321-334. IEEE, 2007.
* <a name="AGM13">[[AGM13]](https://www.cs.purdue.edu/homes/clg/files/Charm.pdf)</a> Akinyele, J.A., Garman, C., Miers, I. et al. Charm: a framework for rapidly prototyping cryptosystems. J Cryptogr Eng 3, 111–128 (2013).
* <a name="MD16">[[MD16]](https://www.youtube.com/watch?v=x_OWvcC8YY0>)</a> Marlon Dutra, Empowering X.509 Certificate Management with Python, PyCon Austrialia 2016.
