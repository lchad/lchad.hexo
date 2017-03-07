---
title: Android上实现带自签名客户端证书的双向校验HTTPS连接
date: 2016-10-13 21:49
categories: Android
toc: true
---
在阅读文章之前读者应该对一些重要概念有一个基本的认识:
 
**http&&ssl:**
为了提高网络传输的安全性，一般会在比较敏感的部分采用https传输，比        如注册、登录、控制台等。像Gmail、网银、icloud等则全部采用https传输。    https/ssl主要起到两个作用：认证、内容加密传输和数据一致性。经CA签发的证书才起到认证可信的作用，所有有效证书均可以起到加密传输的作用。

**数字证书:**
主要在互联网上的用于身份验证的用途。 安全站点在获得CA（Certificate Authority数字证书认证机构）认证后，获得一个数字证书，以此来标识其合法身份的真实性。数字证书主要分为服务器证书和客户端证书。服务器证书（SSL证书）用来进行身份验证和通信的加密，客户端证书主要用于身份验证和电子签名。找CA申请证书是要收费的。
<!--more-->

----
以下是正文

许多Android应用都在使用RESTful或其他基于HTTP协议的模式来和服务器进行通信。

在Android系统上使用HTTP是很简单的,而且文档也相当完备.在不同的Android系统版本上,HTTPClient和HttpURLConnection都是可以使用的,但是Google官方的建议是在2.3及以上版本使用HttpURLConnection,2.2及更早的版本使用HttpClient。

可是当你需要使用HTTPS的时候,会比使用HTTP稍微麻烦一点.在一个近期的项目中,我们要和一个需要客户端证书校验的HTTPS服务器进行通信,并且服务端证书要是自签名的.这样就变得有些棘手了,这篇文章接下来的部分就会介绍到我们遇到的一些挑战。

如果你想直接看代码的话,可以来这里获取[Github](https://github.com/rfreedman/android-ssl)。当然你需要搭建自己的HTTPS服务器并且生成客户端证书.其中一种方式就是使用[Apache mod_ssl module](http://httpd.apache.org/docs/2.2/mod/mod_ssl.html)。

在Android上使用HTTP或HTTPS之前,要确保你的应用有访问网络的权限,正确姿势如下(在AndroidManifest.xml文件中添加):

    <uses-permission android:name="android.permission.INTERNET"/>



### 挑战1:使用客户端证书

我们的客户端证书是[PKCS 12](http://en.wikipedia.org/wiki/PKCS_12)格式的,文件扩展名为.p12。

为了让我们的应用能够拿到证书,我们使用DDMS工具来把证书文件复制到设备SD卡的根目录下,如果你在使用一台虚拟机的话,也同样可以把证书复制到SD卡根目录下.你也可以通过USB的方式从电脑把证书文件复制到设备SD卡根目录下。

在我们的最终版应用中,我们会把证书复制到应用特有的存储空间中(应该是**/Android/data/package name**这个目录下),之后把SD卡根目录下的原始证书删除掉。

使用客户端证书的关键之处是创建一个包含证书的[KeyStore](http://developer.android.com/intl/zh-cn/reference/java/security/KeyStore.html),然后使用这个KeyStore创建一个自定义的[SSLContext](https://docs.oracle.com/javase/7/docs/api/javax/net/ssl/SSLContext.html)。

当我们拿到了包含客户端证书和证书密码的文件引用的时候,我们把它加载到一个合适的[KeyStore](https://docs.oracle.com/javase/7/docs/api/java/security/KeyStore.html)中。(这里可以参考一下示例源码中的SSLContextFactory.java):

    keyStore = KeyStore.getInstance("PKCS12");
    fis = new FileInputStream(certificateFile);
    keyStore.load(fis, clientCertPassword.toCharArray());
  
现在我们已经拥有了包含客户端证书的KeyStore,我们可以使用它来构建一个SSLContext:

    KeyManagerFactory kmf = KeyManagerFactory.getInstance("X509");
    kmf.init(keyStore, clientCertPassword.toCharArray());
    KeyManager[] keyManagers = kmf.getKeyManagers();
    SSLContext sslContext = SSLContext.getInstance("TLS");
    sslContext.init(keyManagers, null, null);

这个SSLContext可以应用在HTTPUrlConnection的方式与服务端连接的过程中用到:

    String result = null;
    HttpURLConnection urlConnection =null;
    try {
        URL requestedUrl = new URL(url);
        urlConnection = (HttpURLConnection)requestedUrl.openConnection();
        if(urlConnection instanceof HttpsURLConnection) {  
            ((HttpsURLConnection)urlConnection).setSSLSocketFactory(sslContext.getSocketFactory());
        }
        urlConnection.setRequestMethod("GET");   
        urlConnection.setConnectTimeout(1500);
        urlConnection.setReadTimeout(1500);
        lastResponseCode = urlConnection.getResponseCode();
        result = IOUtil.readFully(urlConnection.getInputStream());
        lastContentType = urlConnection.getContentType();
    } catch (Exception ex) {
        result = ex.toString();
    } finally { 
        if (urlConnection != null) {
            urlConnection.disconnect();    
        }
    }

### 挑战2:如何确保自签名证书[**译注2**]是可信任的。


现在我们有了可以和HTTPS服务器通过证书连接的客户端代码了。同时持有着客户端证书,可是这只在服务端的证书是可信任的情况下才有用。实际上,这意味着服务端的证书必须被至少一个主流的认证机构所认证,例如**VeriSign,Thawte,Geotrust,Comodo**等. **CA**(Certificate Authority数字证书认证机构) 认证过的证书都是可以在手机上预置进去的。

但我们的证书是自签名的,这意味着SSLContext中默认的[TrustManager](http://docs.oracle.com/javase/6/docs/api/javax/net/ssl/TrustManager.html)是不会信任服务端证书的,并且也无法达成SSL连接。

如果你尝试着在网络上搜索如何实现自信任证书的HTTPS连接,可能会得到很多错误的建议,绝大多数的答案都是教给开发者直接信任**任何证书**就可以了。这当然算是一个误导了,也是不安全的,因为这样做会有被**中间人攻击**[**译注1**]的风险。

我们真正想做的是创建一个自定义的TrustManager,这个TrustManager可以信任我们的自签名证书,并且能被我们的自定义SSLContext使用.因此我们需要一份服务端证书链的副本,这个副本必须包括CA的自签名证书,至少也是CA签署的中级证书。如果你正常地导出服务器的证书,你会得到一个文件,这个文件包含有三个证书- CA认证的证书,中级证书和服务器的证书。这是OK的,额外的证书不是必须的,但也不会有坏处(可以参见下面的**挑战#3**)。

信任自签名的服务端证书的方式和提供客户端证书的方式是很相近的.我们把证书装载到KeyStore中,使用KeyStore来生成一个TrustManger的数组,然后使用这些TrustManager来创建SSLContext。

在我们的app中,我们把服务端证书放到资源文件下(猜测应该是asset目录下,因为证书对于每一个用户来说都是相同的,并且也不会经常发生改变),但是也可以放在设备的外部存储上。

下面是加载服务端证书链到keystore的代码:

    byte[] der = loadPemCertificate(new ByteArrayInputStream(certificateString.getBytes()));
    ByteArrayInputStream derInputStream = new ByteArrayInputStream(der);
    CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");
    X509Certificate cert = (X509Certificate) certificateFactory.generateCertificate(derInputStream);
    String alias = cert.getSubjectX500Principal().getName();
    KeyStore trustStore = KeyStore.getInstance(KeyStore.getDefaultType());
    trustStore.load(null);
    trustStore.setCertificateEntry(alias, cert);

既然我们有了可信任并带有服务端证书的keystore,就可以用它来初始化SSLContext,
基于我们之前的代码,现在生成SSLContext的代码变成了如下:

    KeyManagerFactory kmf = KeyManagerFactory.getInstance("X509");
    kmf.init(keyStore, clientCertPassword.toCharArray());
    KeyManager[] keyManagers = kmf.getKeyManagers();
    TrustManagerFactory tmf = TrustManagerFactory.getInstance("X509");
    tmf.init(trustStore);<br>TrustManager[] trustManagers = tmf.getTrustManagers();
    SSLContext sslContext = SSLContext.getInstance("TLS");
    sslContext.init(keyManagers, trustManagers, null);

现在,我们可以安全的连接到服务端了,而且只信任我们自己的服务端证书,持有我们的客户端证书。如果一切顺利的话,你已经成功了,可以不用继续读下去了。

### 挑战3:处理顺序不正确的服务端证书链

事实证明,一些apache mod_ssl 安装配置(或者其他的SSL提供者),
无论是bug还是错误配置,都会导致服务端证书顺序出现错误。

为了保持正常工作,服务端证书链中的证书都要以root开头,或者 CA certificate其次是中级证书,如果服务端证书也被包含进来,一定要放到最后一个,链中的每个证书(除了root开头的)都必须在用来签名的证书之前。

在我们的项目实际情况下,一个被错误配置的服务器会生成一个证书链,这个证书链以服务端的证书开头,接下来是根证书,然后是中级证书,这会导致SSL手失败。

这种情况下最简单的做法就是:如果可以的话,把服务端证书链的顺序调整好,
如果不能,乱序的证书链只能在客户端处理了。

在我们的android app中,我们通过实现一个自定义的X509TrustManager来让证书链重新排序。

X509TrustManager的代码太长了,没办法完全展示到这里,但是当我们实现了它之后,就可以在SSLContext初始化的过程中,通过这个自定义实现来替换’trustStores’的数组。

    KeyManagerFactory kmf = KeyManagerFactory.getInstance("X509");
    kmf.init(keyStore, clientCertPassword.toCharArray());
    KeyManager[] keyManagers = kmf.getKeyManagers();
    TrustManager[] trustManagers = { new CustomTrustManager(trustStore)};
    SSLContext sslContext = SSLContext.getInstance("TLS");
    sslContext.init(keyManagers, trustManagers,null);

只要加载我们自己的证书到keystore和 Truststore中,并且对服务端证书链进行重新排序的方式,我们就能够创建一个通过客户端证书连接到SSL服务器的SSLContext,即使证书链的书序是错误的并且是自签名的。

----
#### 译注:

[**1**]**中间人攻击(Man-in-the-middle attack)**
是指攻击者与通讯的两端分别创建独立的联系，并交换其所收到的数据，使通讯的两端认为他们正在通过一个私密的连接与对方直接对话，但事实上整个会话都被攻击者完全控制。在中间人攻击中，攻击者可以拦截通讯双方的通话并插入新的内容。在许多情况下这是很简单的。

[**2**]**自签名证书(self-signed certificate)**
非CA颁发的证书，通过自签名的方式得到的证书。例如在Web上,浏览器会显示一个对话框，询问您是否希望信任一个自签名证书。这个是不用花钱的。

----
由于公司有这方面的需求,查了很多资料,找到了一篇很好的文章,所以试着翻译了一下,这是一篇译文,**并非原创!!!并非原创!!!并非原创!!!**也没有100%照着原文翻译的,增加了一些标注解释和自己的理解,翻译的很烂,求喷,谢谢。
原文作者 : [Rich Freedman](hhttps://github.com/rfreedman)
文章出处 : [HTTPS with Client Certificates on Android](http://chariotsolutions.com/blog/post/https-with-client-certificates-on/)
示例程序 : [android-ssl](https://github.com/rfreedman/android-ssl)

----

其他可以参考的资料:
http://developer.android.com/intl/zh-cn/training/articles/security-ssl.html
https://www.zhihu.com/question/31872273
http://www.open-open.com/lib/view/open1413071600531.html#6734290-tsina-1-33379-8c2b59db2c0455f25f549fc37e66faa9
http://blog.csdn.net/logo616/article/details/12983395