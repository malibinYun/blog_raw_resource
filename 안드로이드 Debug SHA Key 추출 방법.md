# 안드로이드 Debug SHA Key 추출 방법



### 디버그용 Key 발급 Gradle 편

gradle명령어로 아래처럼 치면 된다. 짱 쉽다.

```
./gradlew signingReport
```

결과가 아래 처럼 나온다.

```
> Task :app:signingReport
Variant: debug
Config: debug
Store: C:\Users\Malibin\.android\debug.keystore
Alias: AndroidDebugKey
MD5: C1:C7:02:6B:15:A1:5D:ED:E3:DC:6A:A6:C7:69:A4:3F
SHA1: 5A:BB:4C:77:5D:D0:D0:A9:05:FB:90:CB:1B:18:49:05:8B:E5:8D:AE
SHA-256: D8:70:E8:25:31:5B:64:17:F0:9C:A2:3E:AB:D7:0B:00:45:B2:A7:B6:06:86:D4:EE:B4:9A:AA:6A:9E:54:CF:4B
Valid until: 2049�� 2�� 15�� ������
```

안드로이드 스튜디오 내에 있는 terminal로 바로 이 명령어를 사용할 수 있다.

#### 안되는디요?

<img src="image/cannot_execute_gradlew.png"/>

안드로이드 스튜디오에서 start commands execution 이라는 친절한 기능을 만들어 두었다.

아래처럼 명령어를 치고 Ctrl + Enter를 누르면 바로 실행된다.

<img src="image/can_execute_gradlew.png"/>



### 디버그용 key 발급 keytool 편

cmd를 켠다.

명령창에 아래 처럼 친다.

```
keytool -list -v -alias androiddebugkey -keystore %USERPROFILE%\.android\debug.keystore
```

아래와 같은 메시지가 나온다면, 그냥 엔터를 치면 된다. 디버그용 인증서는 비밀번호가 설정되어있지 않다.

```
키 저장소 비밀번호 입력 : 
```

여기까지 따라오면 아래와 비슷한 메시지가 나온다.

```
*****************  WARNING WARNING WARNING  *****************
* 키 저장소에 저장된 정보의 무결성이  *
* 확인되지 않았습니다! 무결성을 확인하려면, *
* 키 저장소 비밀번호를 제공해야 합니다.                  *
*****************  WARNING WARNING WARNING  *****************

별칭 이름: androiddebugkey
생성 날짜: 2019. 2. 23
항목 유형: PrivateKeyEntry
인증서 체인 길이: 1
인증서[1]:
소유자: C=US, O=Android, CN=Android Debug
발행자: C=US, O=Android, CN=Android Debug
일련 번호: 1
적합한 시작 날짜: Sat Feb 23 14:16:27 KST 2019 종료 날짜: Mon Feb 15 14:16:27 KST 2049
인증서 지문:
         MD5:  어쩌구 저쩌구 키 값
         SHA1: MD5 보다 좀 더 긴 키 값
         SHA256: SHA1 보다 더 긴 키 값
서명 알고리즘 이름: SHA1withRSA
주체 공용 키 알고리즘: 1024비트 RSA 키
버전: 1

Warning:
JKS 키 저장소는 고유 형식을 사용합니다. "keytool -importkeystore -srckeystore C:\Users\Malibin\.android\debug.keystore -destkeystore C:\Users\Malibin\.android\debug.keystore -deststoretype pkcs12"를 사용하는 산업 표준 형식인 PKCS12로 이전하는 것이 좋습니다.
```

위의 SHA1 또는 SHA256 키 값을 원하는 곳에 붙여넣으면 된다.

Firebase 또는 Naver Map, KakaoMap 등등...



#### KeyTool 이 없다는데요?

```
'keytool'은(는) 내부 또는 외부 명령, 실행할 수 있는 프로그램, 또는 배치 파일이 아닙니다.
```

이런식으로 실행되지 않는다면...

jdk가 깔려있지 않던지 (안드로이드 스튜디오를 사용하는데 이럴 리는 없다고 본다.)

윈도우 환경변수 Path가 설정되어있지 않아서 그렇다. jdk 경로를 해당 환경변수에 설정한 뒤 cmd를 껐다 키고 나서 다시 keytool을 쳐보면 아래와 같이 나온다.

```
키 및 인증서 관리 툴

명령:

 -certreq            인증서 요청을 생성합니다.
 -changealias        항목의 별칭을 변경합니다.
 -delete             항목을 삭제합니다.
 -exportcert         인증서를 익스포트합니다.
 -genkeypair         키 쌍을 생성합니다.
 -genseckey          보안 키를 생성합니다.
 -gencert            인증서 요청에서 인증서를 생성합니다.
 -importcert         인증서 또는 인증서 체인을 임포트합니다.
 -importpass         비밀번호를 임포트합니다.
 -importkeystore     다른 키 저장소에서 하나 또는 모든 항목을 임포트합니다.
 -keypasswd          항목의 키 비밀번호를 변경합니다.
 -list               키 저장소의 항목을 나열합니다.
 -printcert          인증서의 콘텐츠를 인쇄합니다.
 -printcertreq       인증서 요청의 콘텐츠를 인쇄합니다.
 -printcrl           CRL 파일의 콘텐츠를 인쇄합니다.
 -storepasswd        키 저장소의 저장소 비밀번호를 변경합니다.

command_name 사용법에 "keytool -command_name -help" 사용
```



#### %USERPROFILE%이 뭔데요??

window user 경로를 말합니다.

C드라이브 ➜ 사용자(or User) ➜ UserName(본인 컴퓨터에 설정한 이름)

여기 까지의 경로를 적으면 된다. debug.keystore 파일은 유저 폴더까지 왔다면, .android 폴더 아래에 위치한 것을 볼 수있다.

debug.keystore의 경로를 적어주면 된다.





