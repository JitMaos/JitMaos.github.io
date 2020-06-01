---
title: Android基础『V1V2V3签名』
tags: [Android]
---

---

基础概念

签名：在 APK 中写入一个「指纹」。指纹写入以后，APK 中有任何修改，都会导致这个指纹无效，Android 系统在安装 APK 进行签名校验时就会不通过，从而保证了安全性。

摘要算法：<font color="#dd0000">**使用一段简单的看上去随机的不可逆向的固定长度的字符串来表示一个文件的唯一性。**</font>常见的摘要算法如MD5(128个比特位)、SHA-1算法(160/192/256个比特位)。

公钥密码体制：也称非对称算法，特点是**公钥是公开的**，私钥是保密的。常见的如：RSA。

展开讨论一下RSA：

1. 加密：公钥加密，私钥解密的过程，称为「加密」。
2. **签名：私钥加密，公钥解密的过程，称为「签名」。**加签用来证明这段密文是有公钥对应的私钥拥有者发出的。

网络传输一般不会对整个文件进行签名，因为非对称加密效率比较低。所以一般使用<font color="#dd0000">**摘要算法+RSA算法**</font>实现，传输使用RSA的私钥对文件的摘要信息进行加密作为摘要；接受者接收到文件后，对文件使用相同的摘要算法对文件求摘要值，然后再使用公钥解密加密过得摘要，比较两个摘要值判断文件是否合法。

数字证书：尽管使用RSA +摘要算法可以实现文件的可靠性检测。但是仍然存在问题，<font color="#dd0000">**问题在于公钥是公开的，**</font>比如某些钓鱼网站，直接下发一个他自己的公钥，那么用户就算验证了文件的合法性。依然会进圈套，因为用户不知道和他交互的是否是正确的对象。

所以需要一个权威的CA机构，用来<font color="#dd0000">**颁发数字证书，用以认证用户手上公开的公钥**</font>，这些权威的CA机构将他们的证书先预置在机器里，然后使用他们的私钥对申请数字证书的公钥以及其他信息(有效期、签名算法、数字签名blalala)进行加密，从而得到一个数字证书。用来验证自己所访问的对象是否是正确的。

---

Android中的签名方案

<font color="#dd0000">**V1**</font>：基于jarsigner(JDK自带工具，使用keystore文件进行签名) 或 apksigner(Android专门提供的，使用pk8、x509.pem进行签名)。keystore和pk8/x509.pem可以相互转换。

签名原理：首先keystore文件包含一个MD5和一个SHA1摘要。**这也是很多开放平台需要我们上传的摘要数据**。

签名APK后会在META-INF文件夹下生产CERT.RSA、CERT.SF、MANIFEST.MF三个文件。

在apk中，/META-INF文件夹中保存着apk的签名信息，一般至少包含三个文件，[CERT].RSA，[CERT].SF和MANIFEIST.MF文件。这三个文件就是对apk的签名信息。

<!--More-->

MANIFEST.MF中包含对apk中除了/META-INF文件夹外所有文件的签名值，签名方法是先SHA1()(或其他hash方法)在base64()。存储形式是：Name加[SHA1]-Digest。
[CERT].SF是对MANIFEST.MF文件整体签名以及其中各个条目的签名。一般地，如果是使用工具签名，还多包括一项。就是对MANIFEST.MF头部信息的签名，关于这一点前面源码分析中已经提到。
[CERT].RSA包含用私钥对[CERT].SF的签名以及包含公钥信息的数字证书。

是否存在签名伪造可能：

修改(含增删改)了apk中的文件，则：校验时计算出的文件的摘要值与MANIFEST.MF文件中的条目不匹配，失败。
修改apk中的文件+MANIFEST.MF,则：MANIFEST.MF修改过的条目的摘要与[CERT].SF对应的条目不匹配，失败。
修改apk中的文件+MANIFEST.MF+[CERT].SF，则：计算出的[CERT].SF签名与[CERT].RSA中记录的签名值不匹配，失败。
修改apk中的文件+MANIFEST.MF+[CERT].SF+[CERT].RSA，则：由于证书不可伪造，[CERT].RSA无法伪造。

---

<font color="#dd0000">**V2**</font>：7.0新增的

为啥要做V2，因为V1存在问题：

- 签名校验速度慢

  校验过程中需要对apk中所有文件进行摘要计算，在 APK 资源很多、性能较差的机器上签名校验会花费较长时间，导致安装速度慢。

- 完整性保障不够

  <font color="#dd0000">**META-INF 目录用来存放签名，自然此目录本身是不计入签名校验过程的，可以随意在这个目录中添加文件，比如一些快速批量打包方案就选择在这个目录中添加渠道文件**</font>。(**所以在V2上这种多渠道打包的方式生效了**)

V2签名后的包会被分为四部分 

 	1. Contents of ZIP entries（from offset 0 until the start of APK Signing Block） 
 	2. APK Signing Block 
 	3. ZIP Central Directory 
 	4. ZIP End of Central Directory

**新应用签名方案的签名信息会被保存在区块2（APK Signing Block）**中， 而区块1（`Contents of ZIP entries`）、区块3（`ZIP Central Directory`）、区块4（`ZIP End of Central Directory`）是受保护的，**在签名后任何对区块1、3、4的修改都逃不过新的应用签名方案的检查**。

1. 对新的应用签名方案生成的APK包中的ID-value进行扩展，提供自定义ID－value（渠道信息），并保存在APK中
2. <font color="#dd0000">**而APK在安装过程中进行的签名校验，是忽略我们添加的这个ID-value的，这样就能正常安装了**</font>
3. 在App运行阶段，可以通过ZIP的`EOCD（End of central directory）`、`Central directory`等结构中的信息（会涉及ZIP格式的相关知识，这里不做展开描述）**找到我们自己添加的ID-value，从而实现获取渠道信息的功能**(需要自制一个读取渠道信息的工具)

---

<font color="#dd0000">**V3**</font>：9.0新增的

**格式大体和 v2 类似，在 v2 插入的签名块（Apk Signature Block v2）中，又添加了一个新快（Attr块）**。

在这个新块中，会记录我们之前的签名信息以及新的签名信息，以<font color="#dd0000">**密钥转轮的方案，来做签名的替换和升级。这意味着，只要旧签名证书在手，我们就可以通过它在新的 APK 文件中，更改签名**</font>。

v3 签名新增的新块（attr）存储了所有的签名信息，由更小的 Level 块，以**链表**的形式存储。

其中每个节点都包含用于为之前版本的应用签名的签名证书，最旧的签名证书对应根节点，系统会让每个节点中的证书为列表中下一个证书签名，从而为每个新密钥提供证据来证明它应该像旧密钥一样可信。

这个过程有点类似 CA 证书的证明过程，已安装的 App 的旧签名，确保覆盖安装的 APK 的新签名正确，将信任传递下去。

注意：**签名方式只支持升级不支持降级，如安装了V2的包，不能覆盖替换为V1的包。**

参考

[Android App签名(证书)校验过程源码分析](https://blog.csdn.net/u010651541/article/details/52652690)

[新一代开源Android渠道包生成工具Walle](https://tech.meituan.com/2017/01/13/android-apk-v2-signature-scheme.html)

[Android 签名机制 v1、v2、v3](https://blog.csdn.net/singwhatiwanna/article/details/103142836)