# 안드로이드 SSL 문제 해결기

WebRTC를 이용한 스트리밍 앱을 만들면서 웹소켓 SSL 접속 장애가 발생해 해당 문제를 해결한 우당탕탕 작업기입니다.

WebRTC + Kurento Media Server를 이용해서 [누군가가 만든 예제 앱](https://github.com/nhancv/nc-kurento-android)을 구동시켜 실행해 보려 하는데.......
아래 에러가 저를 괴롭혔습니다.
다신 보지말자 Untrusted chain아

```
E/CONSCRYPT: ------------------Untrusted chain: ----------------------
E/CONSCRYPT: == Chain0 == 
     Version:   3
E/CONSCRYPT:  AuthorityKeyIdentifier:   41830168014142eb317b75856cbae500940e61faf9d8b14c2c6
E/CONSCRYPT:  SubjectKeyIdentifier:   4160414e0b641fa196a0c461245398ae5879f04904b4bdf
E/CONSCRYPT:  Serial Number:   4d9d279d094fb50490a25f6565254db69ec
E/CONSCRYPT:  SubjectDN:   CN=goto.downsups.kro.kr
E/CONSCRYPT:  IssuerDN:   CN=R3, O=Let's Encrypt, C=US
E/CONSCRYPT:  Get not before:   Mon Jan 04 18:19:32 GMT+09:00 2021
E/CONSCRYPT:  Get not after:   Sun Apr 04 18:19:32 GMT+09:00 2021
E/CONSCRYPT:  Sig ALG name:   SHA256withRSA
E/CONSCRYPT:  Signature:   3f7c9c698837756dea5deda7114...생략...9ac1c0052bd2300974723d754
E/CONSCRYPT:  Public key:
     30 82 01 22 30 0d 06 09 2a 86 48 86 f7 0d 01 01 01 05 00 03
     82 01 0f 00 30 82 01 0a 02 82 01 01 00 d7 6e cd 0e f0 05 5f
	...생략...
E/CONSCRYPT: == Chain1 == 
     Version:   3
E/CONSCRYPT:  AuthorityKeyIdentifier:   41830168014c4a7b1a47b2c71fadbe14b9075ffc41560858910
E/CONSCRYPT:  SubjectKeyIdentifier:   4160414142eb317b75856cbae500940e61faf9d8b14c2c6
E/CONSCRYPT:  Serial Number:   400175048314a4c8218c84a90c16cddf
E/CONSCRYPT:  SubjectDN:   CN=R3, O=Let's Encrypt, C=US
E/CONSCRYPT:  IssuerDN:   CN=DST Root CA X3, O=Digital Signature Trust Co.
E/CONSCRYPT:  Get not before:   Thu Oct 08 04:21:40 GMT+09:00 2020
E/CONSCRYPT:  Get not after:   Thu Sep 30 04:21:40 GMT+09:00 2021
E/CONSCRYPT:  Sig ALG name:   SHA256withRSA
E/CONSCRYPT:  Signature:   -26b31f360a7b77c8ce2444ec1d4c0...생략...c53d15e842
E/CONSCRYPT:  Public key:

     30 82 01 22 30 0d 06 09 2a 86 48 86 f7 0d 01 01 01 05 00 03
     82 01 0f 00 30 82 01 0a 02 82 01 01 00 bb 02 15 28 cc f6 a0
	...생략...
W/System.err: javax.net.ssl.SSLHandshakeException: java.security.cert.CertPathValidatorException: Trust anchor for certification path not found.
```

![뭐야 이게 무서워](image\뭐야 이게 무서워.jpg)

....뭔가 안됩니다. 그래. 첨부터 잘 될리가 없지.



---



마지막에 뜬 아래의 놈을 검색하니

```
java.security.cert.CertPathValidatorException: Trust anchor for certification path not found.
```

여러가지 게시글들이 있지만, [구글 문서](https://developer.android.com/training/articles/security-ssl)를 많이 참조했습니다.



처음에는 서버의 인증서가 자체 서명 되어있기도 했고, 인증서의 만료기간이 15~16년도로 설정되어있어서 그런지, 어떤 방법을 해도 먹히지 않는 기현상이 있었습니다... 아마 인증서 자체가 이상해서 그랬지 않았나 생각합니다.

여러가지 방법들을 가지고 실험하던 중간에 서버개발자분이 SSL 인증서를 서버에 달아놔서 이 때부터 처음부터 다시 해결을 해보았습니다.



### 0. SSL을 달아놨는데도 인증서 문제가 생긴다 ??

처음에는 정신이 없어서 이 생각을 못했다가 나중에 생각하니, 매우 이상하긴 했습니다. SSL 달아놨는데 왜...?

필자가 s8+ 폰을 쓰는데, 3~4년정도 되어가는 폰이라 인증서가 폰에 없어서 이런 문제가 생겼다고 나중에 와서 생각해봤습니다.



### 1. 데모 앱 개발자가 만들어둔 라이브러리 사용

구글 문서의 알 수 없는 인증기관 해결에 대한 코드가 분리되서 거의 그대로 사용되고 있길래 사용해봤지만, 먹히지 않았습니다. 코드가 거의 똑같았는데 왜 안먹히는지 정말 알 수 없어서 포기했습니다. [라이브러리](https://github.com/nhancv/nc-android-webrtcpeer/blob/master/webrtcpeer/src/main/java/com/nhancv/webrtcpeer/rtc_comm/ws/DefaultSocketService.java)는 어차피 쓰지 않고 개발해야하기 때문에!

```java
public void setTrustedCertificate(InputStream inputFile) {
        try {
            CertificateFactory cf = CertificateFactory.getInstance("X.509");
            InputStream caInput = new BufferedInputStream(inputFile);
            Certificate ca = cf.generateCertificate(caInput);
            String keyStoreType = KeyStore.getDefaultType();
            this.keyStore = KeyStore.getInstance(keyStoreType);
            this.keyStore.load((InputStream)null, (char[])null);
            this.keyStore.setCertificateEntry("ca", ca);
 		...
}
```



### 2. SSL 인증서 무시

많은 스택 오버플로우에서 SSL을 그냥 무시하는 코드를 추천해줍니다. 이런 추천글에는 항상 "이건 보안상 위험한거니 감안하고 써야한다" 라는 말을 항상 덧붙입니다.

이 코드를 사용해서 시도했을 때 SSL 문제가 해결되었으나, (Java Websocket을 이용했습니다.) 이런 방법은 근본적인 문제 해결 방법이 아니라고 판단해 다른 방법을 더 시도해봤습니다.

```java
SSLContext sslContext = SSLContext.getInstance("SSL");
sslContext.init(null, new TrustManager[] {
  new X509TrustManager() {
    public void checkClientTrusted(X509Certificate[] chain, String authType) {}
    public void checkServerTrusted(X509Certificate[] chain, String authType) {}
    public X509Certificate[] getAcceptedIssuers() { return new X509Certificate[]{}; }
  }
}, null);
this.client.setSocket(sslContext.getSocketFactory().createSocket());
```



### 3. 구글 레퍼런스

#### 1. 자체 서명된 인증서 문제

서버에 인증서 문제가 있었을 때는 이 케이스였지만, 지금은 해당 케이스가 아니었습니다. 

이때는 Issuer에 kurento media server 관련된 정보가 적혀있었습니다.

```
...
E/CONSCRYPT:  SubjectDN:   CN=goto.downsups.kro.kr
E/CONSCRYPT:  IssuerDN:   CN=R3, O=Let's Encrypt, C=US
...
```

서버 개발자가 무료 인증서를 발급받았다고 말했기도 했고, 처음 봤던 에러 메시지 중에 Issuer가 R3라는 로그도 찍혀있었기 때문에 해당 문제는 아니었습니다.



#### 2. 누락된 중간 인증 기관 문제

구글에서 제공해주는 케이스에서는 cmd에 명령어를 쳐서 아래처럼 Certificate chain이 1개만 나온다면 해당 문제로 해결 할 수 있지만,

```
$ openssl s_client -connect egov.uscis.gov:443
    ---
Certificate chain
     0 s:/C=US/ST=District Of Columbia/L=Washington/O=U.S. Department of Homeland Security/OU=United States Citizenship and Immigration Services/OU=Terms of use at www.verisign.com/rpa (c)05/CN=egov.uscis.gov
       i:/C=US/O=VeriSign, Inc./OU=VeriSign Trust Network/OU=Terms of use at https://www.verisign.com/rpa (c)10/CN=VeriSign Class 3 International Server CA - G3
    ---
```

해당 명령어를 쳤을 때는 아래 처럼  Certificate chain이 2개 다 나오는 것을 확인할 수 있었기 때문에 이 문제도 아니라고 판단했습니다.

```
$ openssl s_client -connect goto.downsups.kro.kr:443
CONNECTED(00000188)
depth=1 C = US, O = Let's Encrypt, CN = R3
verify error:num=20:unable to get local issuer certificate
---
Certificate chain
 0 s:CN = goto.downsups.kro.kr
   i:C = US, O = Let's Encrypt, CN = R3
 1 s:C = US, O = Let's Encrypt, CN = R3
   i:O = Digital Signature Trust Co., CN = DST Root CA X3
---
```



#### 3. 알 수 없는 인증 기관 문제

서버 개발자 분이 발급 받은 인증서는 R3에서 발급 받았기 때문에, 알 수 없는 인증 기관 문제가 아니라고 판단했지만, 필자가 사용하는 안드로이드 기기가 3~4년정도 된 S8+ 이기 때문에 CA가 없을 수도 있다고 판단해서 코드를 구성했습니다.

>이 경우에는 시스템에서 신뢰할 수 없는 CA를 보유했기 때문에 `SSLHandshakeException`이 발생합니다. 그 원인은 새 CA로부터 가져온 인증서가 Android에서 아직 신뢰할 수 없거나 앱이 CA가 없는 구형 버전에서 실행 중이기 때문일 수 있습니다. 대개의 경우 CA는 알 수가 없는데 그 이유는 공개 CA가 아니라 자체 사용을 위해 정부, 회사 또는 교육 기관 같은 조직에서 발급하는 비공개 CA이기 때문입니다.
>
>\- 안드로이드 공식 문서

우선 서버에서 인증서를 다운 받고, 안드로이드 raw 폴더에 .cer 파일을 넣은 후, 아래 코드를 실행했습니다.

텍스트 파일로 열면, 이런식으로 생겨먹었습니다

```
  -----BEGIN CERTIFICATE-----
  MIIFLzCCBBegAwIBAgISBNnSedCU+1BJCiX2VlJU22nsMA0GCSqGSIb3DQEBCwUA
  MDIxCzAJBgNVBAYTAlVTMRYwFAYDVQQKEw1MZXQncyBFbmNyeXB0MQswCQYDVQQD
  EwJSMzAeFw0yMTAxMDQwOTE5MzJaFw0yMTA0MDQwOTE5MzJaMB8xHTAbBgNVBAMT
	...엄청 길어서 중략...
  ivhRoydpnSUGYAsE2wFPPegmBWSA7olFlXvpiXyP0gtYUlg8oQTwj9CCgrIeGH5Q
  3gZi6iCI7OUBQSoxMepgDIWp20dj3/Sw1ZcscXdU6VcKg4mMCsvNION6/kqho6T3
  +YBImsHABSvSMAl0cj11AVc2fpZo8ordgkmEPpKj/6+uVyQ=
  -----END CERTIFICATE-----
```

\<인증서 다운받는 법 바로가기>


```java
CertificateFactory cf = CertificateFactory.getInstance("X.509");
InputStream caInput = new BufferedInputStream(
        application.getResources().openRawResource(R.raw.kurento_certification));
Certificate ca;
try {
    ca = cf.generateCertificate(caInput);
} finally {
    caInput.close();
}

String keyStoreType = KeyStore.getDefaultType();
KeyStore keyStore = KeyStore.getInstance(keyStoreType);
keyStore.load(null, null);
keyStore.setCertificateEntry("ca", ca);

String tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
TrustManagerFactory tmf = TrustManagerFactory.getInstance(tmfAlgorithm);
tmf.init(keyStore);

SSLContext context = SSLContext.getInstance("TLS");
context.init(null, tmf.getTrustManagers(), null);
this.client.setSocket(context.getSocketFactory().createSocket());
```





결과는 실패!!!!!

사실 위의 모든 방법이 실패했었습니다...

![e_mon_dog_soriya](image\e_mon_dog_soriya.jpg)



### 이 모든 것들은 사내에서 테스트를 했었다.

갑자기 회사에서 경험한 이야기를 풉니다.

회사 처음 와서 안드로이드 스튜디오를 깔고 Gradle Build를 돌릴 때, 라이브러리 파일들을 다운 받지 못하는 현상이 발생했었습니다. **이것도 이유는 SSL 문제.** 사내에서 보안문제 때문에 외부망에 잘 접속할 수 없는 문제가 발생합니다.

이 때 maven과 google jenter 다운로드 링크를 http://로 강제 지정해서 해결했습니다.



#### 그렇다면...... 회사 와이파이를 쓰고 있으니 생기는 내부망 이슈가 아닐까?!?!

라는 생각이 바로 들어서, 바로 퇴근하자마자 집에가서 회사에서 했던 위의 모든 테스트를 다시 한 번 굴려봤습니다.

![됐다짤](image\됐다짤.png)

아이고... 드디어 됐습니다... 드디어 소켓연결이 되어서 데모 스트리밍에 성공했습니다 😂😂😂😂😂

SSL 무시하는 코드도 사내망에서는 안됐다가 집에와서는 동작을 하네요. 그래도 SSL 무시 방법은 패스하고, 인증서를 기기 내에 집어넣고 굴리는 방법을 써서 결국 성공했습니다.

여러분들도 SSL 문제가 생기면, 제 해결기가 도움 되기를 바랍니다.

