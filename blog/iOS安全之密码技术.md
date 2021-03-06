![图片来自网络](https://upload-images.jianshu.io/upload_images/1159872-15656f71ba6c714c.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

工作或者面试过程中我们总会接触到密码相关的技术和问题，本篇文章是我对近期阅读《图解密码技术》的总结，这本书很值得推荐。那本篇文章是否适合你呢，如果下面的问题你都了解这篇文章暂时不适合你。

- 工作中你用到过哪些加密算法？
- 分组密码的模式有哪些？（分组密码是如何迭代的？）
- 对称加密（共享秘钥密码）和非对称加密（公钥密码）那个快？
- 说说混合密码系统的具体实现？
- SSL/TLS用到哪些密码相关技术？
- 区块链用到了哪些密码相关技术？
- 单项散列函数、消息认证码、数组签名、证书分别是为了解决哪些问题？
- ......


本文章只对我认为重要的知识点做简单介绍和梳理，想详细了解可以阅读原书籍。

开始之前我们先明确一个基本的概念。

**加密**，是以某种特殊的**算法**改变原有的信息数据，使得未授权的用户即使获得了已加密的信息，但因不知解密的方法，仍然无法了解信息的内容。所以Hash算法并不能称为加密算法，因为操作者本人都无法解密成原数据。



### 密码技术

根据秘钥的使用方法可以将密码分为：对称密码和公钥密码两种。**对称密码（共享秘钥密码）**是指加密和解密时使用同一秘钥的方式；**非对称密码（公钥密码）**是指加密和解密时使用不同秘钥的方式。

#### 对称密码

常用的对称密码算法主要有DES、三重DES、AES、Rijndael、APPTagId

##### 比特序列的异或

密码技术中经常会用到比特序列的**异或运算**`xor`,用符号表示⊕，a⊕b = (¬a ∧ b) ∨ (a ∧¬b)，如果a、b两个值不相同，则异或结果为1。如果a、b两个值相同，异或结果为0。密码学中用到异或运算的一个特性是：如果有比特序列a、b、c ，如果a⊕b=c则 c⊕b=a。

|      | 0101（a） |
| ---- | --------- |
| ⊕    | 0011（b） |
| 结果 | 0110（c） |

逆运算：

|      | 0110（c）  |
| ---- | ---------- |
| ⊕    | 0011（b）  |
| 结果 | 0101 （a） |

有没有发现什么我们把a当做源数据，b当做秘钥，c当做加密后的密文，a⊕b相当于加密。c和b异或后又变成了a。这不就是一个很简单的对称加密嘛。

##### DES

DES现在已经不是一种安全的加密方法,已经能够被暴力破解。DES的基本机构是Feistel网络（Feistel结构、Feistel密码）。Feistel网络中加密的步骤称为轮，整个过程就是进行若干次的轮循环。DES是一种16轮循环的Feistel网络。下面看Feistel网络中的轮。

![1001.png](https://upload-images.jianshu.io/upload_images/1159872-9372aac064c50b22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




我们先看其中一轮的步骤，上边是三轮的结构图。

1. 将输入数据等分成左右两部分。
2. 将输入的右侧直接发送给输出的右侧。
3. 将输入的右侧发送给轮函数。
4. 轮函数根据右侧数据和子秘钥，计算出一串随机序列。
5. 将上一步得到的比特序列与左侧数据进行XOR(异或)运算，并将结果作为加密后的左侧。

这样第一轮下来虽然右半部没有加密，但是第二轮就会变成左半部分参与加密。

DES的解密使用相同结构的逆运算就行了。

##### 三重DES

三重DES是为了加强DES的强度，将DES重复3次，得到的一种密码算法，也叫做TDES或者3DES。

3DES并不是进行进行三次DES加密（加密-> 加密 ->加密）,而是加密-> 解密->加密的过程。当然每一步秘钥不同。当一二步秘钥相同时，三重DES也可以当做DES使用，因为前两部加密、解密又变成了原文，相当于只进行了一次DES。

##### AES

AES是为了取代DES而生的一种对称密码算法。他并没有使用Feistel网络，而是采用了Rijndael算法。Rijndael算法中一轮就经过了逐字节替换、平移行、混合列、与轮秘钥进行异或。

![1002.png](https://upload-images.jianshu.io/upload_images/1159872-4c375ab19bbafe52.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)






#### 分组密码模式

我们知道DES和3DES一次只能加密64比特长度，AES一次可加密128或者192、256比特。但是要加密的原文远不止这么短，那怎么加密的？这时候就要对分组密码进行迭代，而迭代的方法就叫做分组密码的模式。

分组密码的模式主要有以下五种：ECB(电子密码本模式)、CBC(密码分组链接模式)、CFB(密文反馈模式)、OFB(输出反馈模式)、CTR(计算器模式)。

##### ECB

ECB模式全名是电子密码本模式，这种模式就是将分组分别加密然后拼接得到密文。因为存在弱点因此通常不被使用。

#####CBC

Cipher Block Chaining mode, 密码分组链接模式。首先将明文分组与前一个密文分组进行 XOR 运算，然后再进行加密。【SSL/TLS 有用到】

#####CFB

Cipher FeedBack mode, 密文反馈模式。前一个密文分组会被送到密码算法的输入端

#####OFB

Output FeedBack mode, 输出反馈模式。密码算法的输出会反馈到密码算法的输入端。

#####CTR

CounTeR mode, 计数器模式。通过将逐次累加的计数器进行加密来生成密钥流的密码。



现在推荐使用CBC和CTR模式。

#### 非对称加密

在对称密码中，加密和解密的秘钥相同，所以需要先向发送者配送秘钥。秘钥配送过程就可能受到攻击者窃取或者篡改，为了解决秘钥配送问题，非对称密码应运而生。在公钥密码中，加密秘钥和解密秘钥是不同的，只要拥有了加密秘钥，任何人都可以加密，但只有解密秘钥才能解密。

##### RSA

RSA是一种公钥密码算法。

- 加密

在RSA中明文、密文和秘钥都是数字。RSA的加密过程可以用下列公式表示：

```
密文 = 明文 E mod N 
```

密文是代表明文的数字的E次方对N求余的结果。E和N的组合就是公钥。

- 解密

解密公式如下：

```
明文 = 密文 D mod N
```

密文的D次方对N求余就是明文。D和N的组合是私钥。

当然这里的E、D、N需要满足一定的规范的。具体计算秘钥对就不在此赘述，想了解的可以查看数据或者上网了解。

RSA是利用大整数的质因数分解的困难度来保证其健壮和攻击困难度的。

##### 其他非对称加密

当然除了RSA外，非对称加密算法还有ELGamal、Rabin、椭圆曲线密码。

#### 混合密码系统

混合密码系统章节的主题就表明了它的用途：用对称密码提高速度，用公钥密码保护会话秘钥。这里的会话秘钥就是对称密码使用的秘钥。

公钥密码当然也不是完美的，他有两个问题：

1. 公钥密码的处理速度远远低于对称密码。
2. 公钥密码难以抵御中间人攻击。

混合密码系统能解决上边第一个问题，要解决第二个问题，就需要我们下一篇文章介绍的公钥认证技术。

混合密码系统就是将对称密码和公钥密码优势相结合的方法。混合密码系统加密过程如下图：

![1003.png](https://upload-images.jianshu.io/upload_images/1159872-e2f4847ce90edc7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




- 会话秘钥一般是通过伪随机数生成器产生，是为本次通信而生成的秘钥。
- 会话秘钥是对称密码的秘钥，同时也是公钥密码的明文。

### 认证技术

#### 单向散列函数

单向散列函数时为了检验消息的**完整性**而生的。它也叫做消息摘要函数、哈希函数或者杂凑函数。

单向散列函数输出的散列值也叫做消息摘要或者指纹。

#####单向性

单向散列函数必须具备单向性：无法通过散列值反算出消息的性质。

##### 应用

- 检测软件是否被篡改
- 基于口令的加密：将口令和盐混合后计算其散列值，然后将散列值作为加密的秘钥。
- 消息认证码（下面详细介绍）
- 数字签名
- 伪随机数生成器
- 一次性口令

##### 具体单向散列函数

在讲解具体散列函数之前，我们先明确两个概念：强抗碰撞性和弱抗碰撞性。

- 强抗碰撞性：要找到散列值相同的两条不同的消息时非常困难的。
- 弱抗碰撞性：要找到和已知消息具有相同散列值的另外一条消息是非常困难的。

单向散列函数的有MD4、MD5、SHA-1、SHA-256、SHA-384、SHA-512

**MD4、MD5**

MD4和MD5都是产生128比特的散列值。

MD4已经不安全被攻破，MD5也已失去强抗碰撞性，也就是说能产生具有相同散列值的两条不同消息。

**SHA-1**

SHA-1的强抗碰撞性2005年也已被攻破。

**SHA-2**

SHA-256、SHA-384、SHA-512统称为SHA-2，SHA-2目前是安全的。

所以一些重要的信息建议使用SHA-2散列函数。

#### 消息认证码

单向散列函数虽然能检测消息的完整性，但是确不能保证消息时有发送者发送的。

**消息认证码**就是为了保证消息的完整性和防伪造。它是一种确认完整性并进行认证的技术。消息认证码简称：**MAC**。

消息认证码计算必须具有发送者和接受者共享秘钥，没有秘钥无法计算，消息认证码正是利用这一特性来完成认证的。

消息认证码的使用步骤如下：

![1004.png](https://upload-images.jianshu.io/upload_images/1159872-98c1acc92e72a57e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




消息认证码同样具有秘钥配送问题，解决方法和对称密码一样可以采用公钥密码、Diffie-Hellman秘钥交换、秘钥配送中心或者其他安全方式发送秘钥。

##### 应用场景

- SWIFT(环球银行金融电信协会)银行间交换信息用到了消息认证码
- IPsec，IP协议中对通信内容的认证和完整性教研采用的消息认证码
- SSL/TLS（下面章节会详细介绍）

##### HMAC

HMAC是一种使用单向散列函数来构造消息认证码的方法，HMAC中的H是Hash的意思。HMAC的步骤如下：

![1005.png](https://upload-images.jianshu.io/upload_images/1159872-3fc42705e5ac0794.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




#### 数字签名

消息认证码是能解决完整性和防止伪造却无法防止发送者否认，这个时候需要数字签名登场了。数字签名相当于现实世界中的盖章，使用它可以识别篡改和伪装，还能防止否认。

消息认证码之所以不能防止否认是因为发送者和接受者使用的是同一个秘钥，正因为是同一个秘钥，消息认证码双方都可以计算，对第三方来说无法确认消息是谁生成的。数字签名就不同了，它使用的是不同的秘钥。

- 生成消息签名：这一行为一般是消息发送方来完成的，也成为`对消息签名`。根据消息内容计算数字签名的值，也意味着“我认可该消息的内容”。
- 验证数字签名：一般由消息接受者来完成，也可以由验证消息的第三方来完成。

数字签名对签名秘钥和验证秘钥进行了区分，使用验证秘钥是无法进行签名的。因此签名秘钥只能由签名的人持有，而验证秘钥则是任何需要验证的人都可以持有。是不是觉得似曾相识，没错这就是我们上面介绍的公钥密码。数字签名正是把公钥密码“反过来用”实现的。看下面对比表：

|            | 私钥                 | 公钥                       |
| ---------- | -------------------- | -------------------------- |
| 公钥密码   | 接受者解密时使用     | 发送者加密时使用           |
| 数字签名   | 签名者生成签名时使用 | 验证着验证签名时使用       |
| 谁持有秘钥 | 个人持有             | 只要需要，任何人都可以持有 |

数字签名中用私钥加密相当于生成签名，用公钥解密相当于验证签名。数字签名的方法有两种：

- 直接对消息进行签名（耗时，因为公钥密码本来就非常慢）
- 对消息的散列值签名（快）

**数字签名无法解决的问题**

用数组签名即可以识别篡改和伪装，还能防止否认。然而要正确使用数字签名有一个大前提，那就是数字签名的公钥必须属于真正的发送者。现在我们仿佛陷入一个死循环-数字签名是用来识别篡改、伪装和防止否认的，但是为此我们又必须从没有被伪装的发送者得到没有被篡改的公钥才行。为了确认我们的得到的公钥是合法的，我们需要使用证书。

####证书

公钥证书：里边记有个人信息以及此人的公钥，并有认证机构施加数字签名。公钥证书也简称证书。

认证机构：能够认定“公钥确实属于此人”并能够生成数字签名的个人或者组织。也是对证书进行管理的人。

下面看一个简单的认证示例图：

![1006.png](https://upload-images.jianshu.io/upload_images/1159872-ee022557e66a152a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 认证机构

为了协助认证机构工作，有时还包括注册机构和仓库。注册机构负责公钥注册和本人身份认证。仓库是保存证书的数据库。

认证机构的作用：

- 生成秘钥对
- 注册证书
- 作废证书与CRL

>CRL是认证机构宣布作废证书的一览表。

##### 证书的层级结构

既然用户的公钥由认证机构添加数字签名进行认证，那认证机构又由谁来认证呢？对于认证机构可以由其他认证机构施加数字签名，进行验证，即生成一张认证机构的公钥证书。

一个认证机构来验证另外一个认证机构的公钥，这样的关系可以迭代好几层，也就形成了证书的层级机构。

你肯定会有疑问最高级认证机构的公钥（根证书）由谁来认证呢？一般有两种方式

1. 根认证机构对自己公钥进行数字签名也叫自签名
2. 根认证机构的公钥由低级认证机构认证形成环形认证

> PKI:公钥基础设施



### 应用场景

下面介绍密码技术在以下三种场景中的使用，PGP密码软件、SSL/TLS、区块链技术。

#### PGP

PGP是由Philip Zimmermann个人编写的密码软件，PGP是Pretty Good Privary的缩写。

PGP的功能

- 对称密码

  支持AES、IDES、CAST、三重DES、Blowfish、Twofish等，分组密码模式采用CFB模式。

- 公钥密码

  公钥密码支持RSA、ELGamal等，PGP不是对明文直接加密，而是使用的混合密码系统来进行加密操作。

- 数字签名

- 单向散列函数

  支持MD5、SHA-1、RIPEMD-160等

- 证书

- 压缩

- 文本数据

- 大文件的拆分和拼合

- 钥匙串管理

PGP混合密码系统加密基本步骤和我们前面讲的相同，只是多了消息的压缩以及二进制->文本转换这两个步骤，具体步骤如下图：

![1007.png](https://upload-images.jianshu.io/upload_images/1159872-677128294f3ccd5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### SSL/TLS

我们都知道HTTP协议是不安全的，现在推荐使用HTTPS协议通信，HTTPS相比HTTP多了一层SSL/TLS。SSL是TLS的前身，SSL协议由于存在安全漏洞，现在使用的是TLS1.1、TLS1.2协议。

SSL/TLS提供了一种密码通信的框架，这意味着它使用的对称密码、公钥密码、数字签名、单向散列函数等技术可以像零件一样进行替换。

##### 协议内容

TLS协议是由TLS记录协议和TLS握手协议这两层协议叠加而成。底层TLS记录协议负责加密，上层的TLS握手协议负责加密以外的其他各种操作。

**握手协议**

上层握手协议又分成4个子协议：握手协议、密码规格变更协议、警告协议和应用数据协议，如下图：

![1008.png](https://upload-images.jianshu.io/upload_images/1159872-c32c3e20dad40e81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 握手协议：负责在客户端和服务器之间协商决定密码算法和共享秘钥。
- 密码规格变更协议：负责向通信对象传达变更密码方式的信号。
- 警告协议：负责在发生错误时将错误传达给对方。
- 应用数据协议：将TLS上面承载的应用数据传达给通信对象的协议。

**TLS记录协议**

TLS协议负责消息的压缩、加密以及数据的认证。

##### 小结

具体客户端和服务器之间开始连接步骤由于篇幅原因就不在此详细介绍，想了解的可以上网搜索或者阅读原书籍。

我们来看看TLS用到了哪些密码技术。

| 密码技术            | 作用                                               |
| ------------------- | -------------------------------------------------- |
| 公钥密码            | 加密预备主密码                                     |
| 单向散列函数        | 构成伪随机数生成器                                 |
| 数字签名            | 验证服务器和客户端的证书                           |
| 伪随机数生成器      | 生成预备主密码、根据主密码生成密码、生成初始化向量 |
| 对称密码（CBC模式） | 确保片段的机密性                                   |
| 消息认证码          | 确保片段的完整性并进行认证                         |

####比特币

区块链就是保存比特币全部交易记录的公共账簿。

关于区块链相关的概念：比特币、比特币地址、钱包、区块、挖矿、矿工、工作量证明等，想详细了解推荐阅读阮一峰的http://www.ruanyifeng.com/blog/2017/12/blockchain-tutorial.html这篇文章。我们说说它用到的密码技术。

- 散列值

  工作量证明就是通过计算符合要求的散列值来实现的。每个区块中也包括当前区块的散列值和上一个区块的散列值。

- 数字签名技术

  在创建交易时运用了数字签名技术，比特币的数字签名算法是椭圆曲线DSA。

- 公钥密码

  钱包地址使用的就是公钥密码技术


### 总结

《图解密码技术》是一本通俗易懂的书籍，强烈推荐阅读。本人也是第一次写读书笔记，有不完善的地方欢迎留言指正。

----------

更多iOS、逆向、ReactNative开发文章请专注微博或者微信公众账号：乐Coding，微信扫描下方二维码也可关注。

![lecoding](http://upload-images.jianshu.io/upload_images/1159872-1bb43add674a10d0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)icon.jpg


