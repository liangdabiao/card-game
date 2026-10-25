---
layout: post
title: 面向医疗区块链的属性基加密
date: 2019-04-28 00:00:00.000000000 -00:00
permalink: attribute-based-encryption-for-healthcare-blockchain/
author: Benny Cheung
artwork:
  author: Style of Byzantine mosaics transfer to many faces
tags:
  - Cryptography
  - Attribute-based Encryption
  - Healthcare
  - Blockchain
  - FHIR
category: post
comments: true
image: images/attribute-based-encryption-for-healthcare-blockchain/cover2x.jpg
images:
  cover: images/attribute-based-encryption-for-healthcare-blockchain/cover.jpg
  header: images/attribute-based-encryption-for-healthcare-blockchain/header.jpg
published: true
series: "Blockchain Healthcare"
series_order: 2
---

<!--excerpt.start-->
毫不意外，医疗提供方最大的关切之一就是数据安全。可以说，数据所有权是这个信息时代最重要的资产。患者已经意识到个人信息价值。
因此，医疗提供方需要从集中式所有权转向分布式所有权。现在，患者对自己的健康数据拥有完全控制权，而提供方只在必要时才被授权访问数据。
<!--excerpt.end-->

----
> [2024/10/05] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/attribute-based-encryption-for-healthcare-blockchain/Podcast_Blockchain_ABE.mp3
{% include open-embed.html %}
----

只要私密数据的一部分在使用前被遮蔽，患者就会有动力分享部分健康相关数据，以推动医学研究。
要让这种去中心化的数据所有权与共享行得通，需要一种新技术来保护数据隐私并提供安全的系统。

![医疗区块链]({{ site.baseurl }}/images/attribute-based-encryption-for-healthcare-blockchain/Medical_Blockchain_StyleArt.jpg) 

*图：Kandinsky 艺术风格迁移到医疗区块链图像*

我们之前的文章 ["Why Blockchain for Healthcare?"](https://www.jonahgroup.com/blog/why_blockchain_for_healthcare)（为什么医疗需要区块链？）讲解了如何用区块链与获得授权的各方共享数据。
在本文中，我们将展开医疗数据共享的讨论，
聚焦通过*属性基加密*（Attribute-based Encryption）实现的数据隐私。
当患者决定分享部分健康数据时，数据被保密地记录到区块链上。只有获得授权的各方才能解密共享的数据。

在探索*面向医疗区块链的属性基加密*的概念与实践时，
我们将从系统视角过一遍以下主题，简化数学细节。
当然，对数学倾向的读者，原始论文列在[参考文献](#References)部分，可供深入理解。

## 个人健康记录（PHR）安全
开启这段新共享惯例的旅程之前，
先介绍快速医疗互操作性资源（Fast Healthcare Interoperability Resources，FHIR）——快速高效存储/检索健康数据的行业标准。
FHIR 中的健康数据单元被称为"资源"（Resource）。FHIR 定义了多种这样的资源，如 Patient、Practitioner、Observation、MedicalRequest 等，每种资源可以链接到多个其他资源。
在之前的文章 [FHIR 服务器搭建与运行]({{ site.baseurl }}/fhir-server-up-and-running)中，我们提供了用模拟患者健康数据搭建 FHIR RESTful API 服务的实用教程。一如既往，当我们需要理解一个新概念时，理论与实践平衡的理念总能派上用场。

虽然 FHIR 让健康数据的发布变得容易，
但包含个人身份信息（PII）的数据被无意或恶意地泄露给未经授权的个人或组织，可能造成灾难性后果。
因此，医疗提供方在发布敏感医疗数据时必须遵守联邦和州的政策。
例如，必须仔细研究并执行诸如 [HIPAA 和 HITECH](https://www.hipaajournal.com/relationship-between-hitech-hipaa-electronic-health-medical-records/) 之类的合规政策以备审计。
确保数据安全是迈向合规确定无疑的第一步。

## 属性基加密（ABE）
属性基加密（Attribute-based Encryption，ABE）是一种相对较新的方法，
它重新思考了公钥密码学的概念
[[SW05]](#SW05)[[GPSW06]](#GPSW06)[[BSW07]](#BSW07)[[Waters11]](#Waters11)。
在传统公钥密码学中，消息使用接收方的公钥为特定接收者加密。基于身份的密码学，尤其是基于身份的加密（IBE），改变了公钥密码学的传统理解：它允许公钥是任意字符串，例如接收方的电子邮件地址。

ABE 更进一步，把身份定义为非原子性的一组属性，例如角色和上下文；消息可以针对属性的子集（密钥策略 ABE——KP-ABE）或定义在属性集合上的策略（密文策略 ABE——CP-ABE）来加密。关键问题是：只有当某人持有"匹配属性"的密钥时，才能解密密文，而用户密钥总是由某个可信方颁发。

### 身份与属性
在注册患者或从业者时，可以为它们指定属性，
在其注册到证书颁发机构（CA）时，这些属性会被添加到它们的 X.509 证书中。
属性的示例包括参与网络的各组织共同认可的角色名，例如"Patient"或"Practitioner"。
当智能合约执行时，
它可以在 invoke 或 query 交易之前提取身份与属性。
在简单层面上，这也允许把应用层属性通过 X.509 证书传递
进智能合约。参与者的身份与属性受到完整保护与认证，即没有人能伪造身份与属性。

对于经过认证的参与者，我们可以从其 X.509 证书中找到主体的属性，例如

```
Name;
Organization;
Position;
Role;
...
```

我们还可以添加捕捉操作上下文的应用层属性，例如

```
Action;
IP-address;
Time;
Device;
...
```

有兴趣的读者请参考我们最新的文章 [X.509 Identity for Attribute-based Encryption]({{ site.baseurl }}/identity-for-attribute-based-encryption)，它描述了实践方法，演示如何用 Python cryptography 生成带自定义属性的 X.509 证书；随后，我们使用 charm-crypto 框架的混合适配器，配合 X.509 自定义属性执行 CP-ABE（密文策略）。  

### 密文策略 ABE
在密文策略属性基加密（CP-ABE）中，用户的私钥与一组属性关联，密文则指定了系统定义的属性全集之上的一个访问策略。
当且仅当用户的属性满足相应密文的策略时，用户才能解密该密文。
策略可以用合取、析取和 $$(k,n)$$ 门限门定义在属性之上，
即 $$n$$ 个属性中必须存在 $$k$$ 个。
举例来说，假设属性全集定义为 $${A,B,C,D}$$，
用户 1 获得属性 $${A,B}$$ 的密钥，
用户 2 获得属性 $${D}$$ 的密钥。
如果一个密文按策略 $$(A \land C) \lor D$$ 加密，
那么用户 2 将能够解密，而用户 1 无法解密。

CP-ABE 不像传统公钥密码学那样面向特定用户加密。相反，用户的私钥和密文都与一组属性或属性上的策略相关联。只要用户的私钥与密文之间存在"匹配"，用户就能解密密文。

因此，CP-ABE 可以实现隐式授权，即授权被纳入加密数据之中，只有满足关联策略的人才能解密数据。另一个不错的特性是：用户可以在数据按策略加密之后再获得私钥。所以，加密数据时无需知道实际能解密的用户集合，只需指定允许解密的策略。将来任何获得满足该策略之属性密钥的用户，都能解密这些数据。

### CP-ABE 算法
我们真正想要的，是能够把访问建立在别的东西之上，例如他的位置，或者他是否是与患者关联的从业者。这些被定义为其访问权限的属性，构成基于属性的安全，从而我们可以对解密过程定义细粒度的控制。例如，我们可以规定：某些敏感健康信息只有在患者与从业者都完成身份认证且处于可证明的位置时才能访问。

#### CP-ABE 加密流程
加密流程有以下几个阶段 [[BSW07]](#BSW07)：

* **Setup（建立）**。建立算法不接受除隐式安全参数之外的输入。它输出公共参数 $$(PK)$$ 和主密钥 $$(MK)$$。
* **Encrypt(PK, M, A)（加密）**。加密算法以公共参数 $$(PK)$$、消息 $$(M)$$ 和属性全集之上的访问结构 $$(A)$$ 为输入。算法加密 $$(M)$$ 并产出密文 $$(CT)$$，使得只有拥有一组满足访问结构的属性的用户才能解密该消息。我们假设密文隐式包含 $$(A)$$。
* **Key_Generation(MK, S)（密钥生成）**。密钥生成算法以主密钥（MK）和描述密钥的一组属性 $$(S)$$ 为输入。它输出私钥 $$(SK)$$。
* **Decrypt(PK, CT, SK)（解密）**。解密算法以公共参数 $$(PK)$$、包含访问策略 $$(A)$$ 的密文 $$(CT)$$，以及针对属性集 $$(S)$$ 的私钥 $$(SK)$$ 为输入。如果属性集 $$(S)$$ 满足访问结构 $$(A)$$，算法将解密密文并返回消息 $$(M)$$。
* **Delegate(SK, S˜)（委托）**。委托算法以针对某属性集 $$(S)$$ 的密钥 $$(SK)$$ 和集合 $$S˜ \subset S$$ 为输入。它输出针对属性集 $$(S˜)$$ 的密钥 $$(S˜K)$$。[注：本文不使用该函数]

#### CP-ABE 加密时序
为了消化如何应用这套 CP-ABE 算法，
以下时序图描绘了 CP-ABE 加密流程：

> Python 实现见后文 [ABE with Python](#ABE-Python) 一节

* *阶段 1*：颁发证书——患者与从业者都要参与网络，必须注册并注册加入。证书颁发机构（CA）将向参与者颁发相应的 X.509 证书。每张证书都包含精确描述参与者身份与属性的内容。
* *阶段 2*：准备公钥——系统使用 `Setup` 算法准备主密钥（MK）与公钥（PK）。MK 只有基于属性的访问控制系统知晓，因为它用于生成每个参与者的密钥。PK 分发给每位参与者用于加密其数据。
* *阶段 3*：患者加密数据——患者完成认证并从 FHIR 服务器检索其健康记录（M）。系统使用患者证书，通过 `Key_Generation` 生成患者的密钥。用访问策略（A）通过 `Encrypt` 以公钥（PK）加密健康记录（M）。密文（CT）可以放到区块链（或其他存储系统）上用于数据共享。
* *阶段 4*：从业者解密数据——系统使用从业者证书，通过 `Key_Generation` 生成从业者的密钥。从业者从区块链请求密文（CT）之后，可以用 `Decrypt` 和从业者的密钥解密密文。如果从业者的密钥满足密文策略（A），则返回健康数据（M）；否则解密失败并报错。

![属性基加密时序]({{ site.baseurl }}/images/attribute-based-encryption-for-healthcare-blockchain/ABAC_Cryptography_Seq.jpg)

*图：CP-ABE 加密流程：(1) 患者与从业者收到各自的凭证 (2) ABAC 按各自参与者凭证准备共享密钥 (3) 患者按策略加密数据 (4) 从业者在其密钥匹配策略时解密数据*

### 对 FHIR Patient 资源应用 ABE
例如，可以从 FHIR 服务器检索 `smart-1032702` 的 "Patient" 资源。

```
GET http://localhost:8080/hapi-fhir/baseDstu3/Patient/smart-1032702
```

FHIR 服务器对 `smart-1032702` 返回如下结果，

```json
{
    "fullUrl": "http://localhost:8080/hapi-fhir/baseDstu3/Patient/smart-1032702",
    "resource": {
        "resourceType": "Patient",
        "id": "smart-1032702",
        "name": [
            {
            "use": "official",
            "family": "Shaw",
            "given": [
                "Amy",
                "V"
            ]
            }
        ],
        "telecom": [
            {
            "system": "phone",
            "value": "800-782-6765",
            "use": "mobile"
            },
            {
            "system": "email",
            "value": "amy.shaw@example.com"
            }
        ],
        "gender": "female",
        "birthDate": "2007-03-20",
        "address": [
            {
            "use": "home",
            "line": [
                "49 Meadow St"
            ],
            "city": "Mounds",
            "state": "OK",
            "postalCode": "74047",
            "country": "USA"
            }
        ],
        "generalPractitioner": [
            {
            "reference": "Practitioner/smart-Practitioner-72004454"
            }
        ]
        ...
    }
}
```

智能合约收到患者记录后，可以按策略执行 ABE。
患者记录的密文附带访问策略，密钥则与属性集合关联。
例如，策略 $$A$$ 可以是：*患者记录只能由该特定患者与从业者解密*。

```
Policy A =
(Role=PRACTITIONER AND Id=SMART-PRACTITIONER-72004454) OR (Role=PATIENT AND Id=SMART-1032702)
```

或者一个更大泛化的策略 $$A'$$：*对在某特定医院工作的所有从业者*。

```
Policy A' =
(Org=HOSPITAL AND Role=PRACTITIONER) OR (Role=PATIENT AND Id=SMART-1032702)
```

满足全部条件即可访问一个 FHIR 资源。
在这个模型中，策略的复杂度没有上限。此外，加密可以应用于敏感信息的细粒度控制，例如用不同的策略加密 `birthDate` 或 `SIN` 号码。

延伸阅读：读者可以参考 [[SM17]](#SM17) 与 [[TM17]](#TM17) 了解基于属性的访问控制在医疗资源上的应用。

## 用 Python 做 ABE 实验
从零实现 ABE 颇具挑战，我们转而使用现有的 Python 密码学库和系统来构建实验。
我们的实验使用基于 `Charm` 框架的属性基加密 Python 实现 [[AC17]](#AC17)，Github 仓库：<https://github.com/sagrawal87/ABE>

以下步骤将引导你完成安装与执行。
这些步骤已在 Mac（运行 OS X High Sierra）上验证可行。

### 安装前置依赖
#### PBC
PBC（Pairing-based cryptography，基于配对的密码学）是一个密码学库，网站：<https://formulae.brew.sh/formula/pbc>

```
brew install pbc
...
==> Pouring pbc-0.5.14.high_sierra.bottle.tar.gz
🍺  /usr/local/Cellar/pbc/0.5.14: 35 files, 786KB
```

#### Charm
Charm 是一个快速构建高级密码系统原型的框架 [[AGMPRGR13]](#AGMPRGR13)，Github 仓库：<https://github.com/JHUISI/charm>

从源码安装 Charm 很直接。首先，确认你已安装以下依赖：

* GMP 5.x
* PBC
* OPENSSL

```bash
./configure.sh --enable-darwin
make install
```

```
...
Using /usr/local/anaconda3/lib/python3.5/site-packages
Finished processing dependencies for Charm-Crypto==0.50
```

运行内置测试套件以确保所有加密函数工作正常，

```
make test
```

### 用 Python 做 ABE
以下 `main.py` 经过修改以运行 [[BSW07]](#BSW07) 的 CP-ABE 方案。
策略使用 `((PATIENT and SMART-1032702) or (PRACTITIONER and SMART-PRACTITIONER-72004454))`

* 患者属性列表为 `['PATIENT', 'SMART-1032702']`，`patient_key` 拥有满足策略的足够属性。密文可以成功解密。
* 同样，从业者属性列表为 `['PRACTITIONER', 'SMART-PRACTITIONER-72004454']`，`practitioner_key` 拥有满足策略的足够属性。密文也能成功解密。
* 然而，从业者#2 的属性列表为 `['PRACTITIONER', 'SMART-PRACTITIONER-90000001']`，
`practitioner2_key` 不满足策略。这种情况下密文解密失败。

```python
from charm.toolbox.pairinggroup import PairingGroup, GT
from bsw07 import BSW07


def main():
    # instantiate a bilinear pairing map
    # 'MNT224' represents an asymmetric curve with 224-bit base field
    pairing_group = PairingGroup('MNT224')
    
    # CP-ABE under DLIN (2-linear)
    cpabe = BSW07(pairing_group, 2)

    # run the set up
    (pk, msk) = cpabe.setup()

    # generate a Patient key
    patient_attr_list = ['PATIENT', 'SMART-1032702']
    patient_key = cpabe.keygen(pk, msk, patient_attr_list)

    # generate a Practitioiner key
    practitioner_attr_list = ['PRACTITIONER', 'SMART-PRACTITIONER-72004454']
    practitioner_key = cpabe.keygen(pk, msk, practitioner_attr_list)

    # generate a Practitioiner#2 key
    practitioner2_attr_list = ['PRACTITIONER', 'SMART-PRACTITIONER-90000001']
    practitioner2_key = cpabe.keygen(pk, msk, practitioner2_attr_list)

    # choose a random message pretend to be patient's record
    msg = pairing_group.random(GT)
   
    # generate a ciphertext
    policy_str = '((PATIENT and SMART-1032702) or (PRACTITIONER and SMART-PRACTITIONER-72004454))'
    ctxt = cpabe.encrypt(pk, msg, policy_str)

    # decryption as Patient
    rec_msg = cpabe.decrypt(pk, ctxt, patient_key)
    if rec_msg == msg:
        print ("Successful decryption as a Patient.")
    else:
        print ("Decryption as a Patient failed.")

    # decryption as Practitioner
    rec_msg = cpabe.decrypt(pk, ctxt, practitioner_key)
    if rec_msg == msg:
        print ("Successful decryption as a Practitioner.")
    else:
        print ("Decryption as a Practitioner failed.")

    # decryption as Practitioner#2
    rec_msg = cpabe.decrypt(pk, ctxt, practitioner2_key)
    if rec_msg == msg:
        print ("Successful decryption as a Practitioner#2.")
    else:
        print ("Decryption as a Practitioner#2 failed.")

if __name__ == "__main__":
    main()
```    

运行 `main.py` 之后，我们将看到解密成功。

```
python main.py

Successful decryption as a Patient.
Successful decryption as a Practitioner.
Policy not satisfied.
Decryption as a Practitioner#2 failed..
```

经过这个实验，我们理解了确保患者数据机密性的实用 CP-ABE 技术。
凭借足够的集成技能与区块链知识，我们甚至可以把健康数据记录进 Hyperledger Fabric，投入真实使用。

> **更新 2020/08/17**
> 读者可能期望用 CP-ABE 加密任意明文消息。我们将在
> [X.509 Identity for Attribute-based Encryption]({{ site.baseurl }}/identity-for-attribute-based-encryption) 中提供一个扩展示例，
> 演示如何加密任意明文消息，并用证书属性解密 

## 参考文献

*  <a name="SW05">[[SW05]](https://pdfs.semanticscholar.org/267c/eba2ccc0f0b8872b24ac48c5e8680a04cb89.pdf)</a> Sahai, Amit, and Brent Waters. "Fuzzy identity-based encryption." In Eurocrypt, vol. 3494, pp. 457-473. 2005.
* <a name="GPSW06">[[GPSW06]](https://eprint.iacr.org/2006/309.pdf)</a> Goyal, Vipul, Omkant Pandey, Amit Sahai, and Brent Waters. "Attribute-based encryption for fine-grained access control of encrypted data." In Proceedings of the 13th ACM conference on Computer and communications security, pp. 89-98. ACM, 2006.
* <a name="BSW07">[[BSW07]](https://www.cs.utexas.edu/~bwaters/publications/papers/cp-abe.pdf)</a> John Bethencourt, Amit Sahai, and Brent Waters. "Ciphertext-policy attribute-based encryption." In Security and Privacy, 2007. SP'07. IEEE Symposium on, pp. 321-334. IEEE, 2007.
* <a name="Waters11">[[Waters11]](https://eprint.iacr.org/2008/290.pdf)</a> Waters, Brent. "Ciphertext-policy attribute-based encryption: An expressive, efficient, and provably secure realization." In Public Key Cryptography, vol. 6571, pp. 53-70. 2011.
* <a name="AGMPRGR13">[[AGMPRGR13]](https://eprint.iacr.org/2011/617.pdf)</a> Akinyele, Joseph A., Christina Garman, Ian Miers, Matthew W. Pagano, Michael Rushanan, Matthew Green, and Aviel D. Rubin. "Charm: a framework for rapidly prototyping cryptosystems." Journal of Cryptographic Engineering 3, no. 2 (2013): 111-128.
* <a name="SM17">[[SM17]](https://www.cs.colostate.edu/~subhomuk/Papers/ABAC17.pdf)</a> Subhojeet Mukherjee, et. al., "Attribute Based Access Control for Healthcare Resources", ABAC '17 Proceedings of the 2nd ACM Workshop on Attribute-Based Access Control, pp 29-40.
* <a name="AC17">[[AC17]](https://eprint.iacr.org/2017/807.pdf)</a> Shashank Agrawal, and Melissa Chase. "FAME: Fast Attribute-based Message Encryption." To appear in the Proceedings of the 24th ACM conference on Computer and communications security, 2017.
* <a name="TM17">[[TM17] (video only)](https://www.youtube.com/watch?v=XmvG4jyaxrI.)</a> Tony Mallia, "Attribute Based Access Control (ABAC) in Healthcare", Edmond Scientific Company, Cloud Identity Summit 2017.
