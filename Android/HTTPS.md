https://developer.android.com/training/articles/security-ssl.html#Concepts

## HTTP

```
graph TD

HTTP-->TCP
```

## HTTPS
```
graph TD

HTTP-->TLS/SSL
TLS/SSL-->TCP
```

## 加密

1. 对称加密
> 加密数据用的密钥，跟解密数据用的密钥是一样的。维护成本高，保证数据双向安全。

2. 非对称加密
> 加密数据用的密钥，跟解密数据用的密钥是不一样的。通过公钥加密的数据，只能通过私钥解开。通过私钥加密的数据，只能通过公钥解开。维护成本低，只保证数据单向安全。


## HTTPS 工作流程

![HTTPS 工作流程](http://ooun8fyfu.bkt.clouddn.com/17-5-19/46070636-file_1495184993753_133d7.png)

## 数字证书
证书包含以下信息：
1. 申请者公钥
2. 申请者的组织信息和个人信息
3. 签发机构CA的信息
4. 有效时间
5. 证书序列号
6. 用到的 Hash 算法
7. 以上信息的签名

### 数字证书的签名生成

```
graph LR

原文-->|hash|证书摘要
证书摘要-->|CA 私钥加密|签名
```

### 数字证书的校验
1. 检查证书信息（颁发机构、有效期等）
2. 比较证书摘要1与证书摘要2是否一致，如下图
```
graph TD

证书签名-->|公钥解密|证书摘要1
证书原文-->|HASH|证书摘要2
```

## Android 中常见的 HTTPS 异常

### 服务器证书验证错误

> javax.net.ssl.SSLHandshakeException: java.security.cert.CertPathValidatorException: Trust anchor for certification path not found.

1. 颁发服务器证书的 CA 未知
2. 服务器证书不是由 CA 签署的，而是自签署
3. 服务器配置缺少中间 CA

解决方法
1. 指定 HttpsURLConnection 信任特定的 CA 集合

```java
// Load CAs from an InputStream
// (could be from a resource or ByteArrayInputStream or ...)
CertificateFactory cf = CertificateFactory.getInstance("X.509");
// From https://www.washington.edu/itconnect/security/ca/load-der.crt
InputStream caInput = new BufferedInputStream(new FileInputStream("load-der.crt"));
Certificate ca;
try {
  ca = cf.generateCertificate(caInput);
  System.out.println("ca=" + ((X509Certificate) ca).getSubjectDN());
} finally {
  caInput.close();
}

// Create a KeyStore containing our trusted CAs
String keyStoreType = KeyStore.getDefaultType();
KeyStore keyStore = KeyStore.getInstance(keyStoreType);
keyStore.load(null, null);
keyStore.setCertificateEntry("ca", ca);

// Create a TrustManager that trusts the CAs in our KeyStore
String tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
TrustManagerFactory tmf = TrustManagerFactory.getInstance(tmfAlgorithm);
tmf.init(keyStore);

// Create an SSLContext that uses our TrustManager
SSLContext context = SSLContext.getInstance("TLS");
context.init(null, tmf.getTrustManagers(), null);

// Tell the URLConnection to use a SocketFactory from our SSLContext
URL url = new URL("https://certs.cac.washington.edu/CAtest/");
HttpsURLConnection urlConnection =
    (HttpsURLConnection)url.openConnection();
urlConnection.setSSLSocketFactory(context.getSocketFactory());
InputStream in = urlConnection.getInputStream();
copyInputStreamToOutputStream(in, System.out);
```
2. 跳过证书校验
```java
public class AcceptAllTrustManager implements X509TrustManager {

  @Override public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {
    // do nothing，接受任意客户端证书
  }
    
  @Override public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
    // do nothing，接受任意服务端证书
  }
    
  @Override public X509Certificate[] getAcceptedIssuers() {
    return null;
  }
}
```
```java
/**
 * 信任所有的CA证书，不安全，仅供测试阶段使用
 */
public SSLContext makeContextToTrustAll() throws Exception {
  AcceptAllTrustManager tm = new AcceptAllTrustManager();
  SSLContext sslContext = SSLContext.getInstance("TLS");
  sslContext.init(null, new TrustManager[] { tm }, null);

  return sslContext;
}
```

### 域名验证失败

> java.io.IOException: Hostname 'xxx' was not verified

正在通信的服务器没有提供正确的证书，服务器证书中配置的域名和客户端请求的域名不一致

解决方法
1. 重新生成服务器的证书，用真实的域名信息
2. 自定义 HostnameVerifier，在握手期间，如果 URL 的主机名和服务器的标识主机名不匹配，则验证机制可以回调此接口的实现程序来确定是否应该允许此连接。可以通过自定义 HostnameVerifier 实现一个白名单的功能
```java
HostnameVerifier DO_NOT_VERIFY = new HostnameVerifier() {
  @Override  public boolean verify(String hostname, SSLSession session) {
    // 设置接受的域名集合
    if (hostname.equals(...))  {
      return true;
   }
 }
};
HttpsURLConnection.setDefaultHostnameVerifier(DO_NOT_VERIFY);
```

### Android 上 TLS 版本兼容问题

|Name   |Supported (API Levels)|
|:------|:---------------------|
|Default|10+                   |
|SSL    |10–TBD                |
|SSLv3  |10–TBD                |
|TLS    |1+                    |
|TLSv1  |10+                   |
|TLSv1.1|16+                   |
|TLSv1.2|16+                   |

按官方的文档显示，在API 16+ 以上，TLS1.1 和 TLS1.2 是默认开启的。但是实际上在 API 20+ 以上才默认开启，4.4 以下的版本是无法使用 TLS1.1 和 TLS 1.2 的，这也是Android系统的一个bug。

解决方法

```java
SSLSocketFactory noSSLv3Factory;
if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.KITKAT) {
  noSSLv3Factory = new TLSSocketFactory(mSSLContext.getSSLSocket().getSocketFactory());
} else {
  noSSLv3Factory = mSSLContext.getSSLSocket().getSocketFactory();
}
```

```java
public class TLSSocketFactory extends SSLSocketFactory {

  private SSLSocketFactory internalSSLSocketFactory;

  public TLSSocketFactory() throws KeyManagementException, NoSuchAlgorithmException {
    SSLContext context = SSLContext.getInstance("TLS");
    context.init(null, null, null);
    internalSSLSocketFactory = context.getSocketFactory();
  }

  public TLSSocketFactory(SSLSocketFactory delegate) throws KeyManagementException, NoSuchAlgorithmException {
    internalSSLSocketFactory = delegate;
  }
  
  // ......

  @Override public Socket createSocket(InetAddress address, int port, InetAddress localAddress, int localPort)throws IOException {
    return enableTLSOnSocket(internalSSLSocketFactory.createSocket(address, port, localAddress, localPort));
  }
    
  // 开启对 TLS1.1 和 TLS1.2 的支持
  private Socket enableTLSOnSocket(Socket socket) {
    if (socket != null && (socket instanceof SSLSocket)) {
      ((SSLSocket)socket).setEnabledProtocols(new String[] {"TLSv1.1", "TLSv1.2"});
    }
    return socket;
  }
}
```