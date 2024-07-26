
![Untitled (19)](https://github.com/user-attachments/assets/9ec237dd-148b-4269-8672-db6b55f46896)

# **피처 플래깅 솔루션 : Light Switch**

기간 : **2024.04.08 ~. 2024.05.20  / 7주**

기관 : SSAFY 10기 2학기 자율 프로젝트 / 6인

Github : https://github.com/LightSwitch-S202/LightSwitch

성과: SSAFY 10기 2학기 자율 프로젝트 우수상

---


## **피처플래그 (Feature Flag)**


소프트웨어 개발에서 **특정 기능이나 코드를 조건적으로 활성화하거나 비활성화할 수 있게 해주는 기술**

피처 플래그, 피처 토글, 기능 토글 등 다양한 이름으로 불리우며, **새로운 코드를 배포하지 않고 런타임에 특정 기능을 키거나 끌 수 있다**.

이를 활용하면 손쉽게 오류에 대처하거나, 롤백, A/B 테스트, 카리나 배포 등이 가능하다.

<br>

## **피처플래깅 솔루션 : Light Switch**

Light Switch는 **오픈소스 피쳐플래깅 솔루션**으로 Docker Image 형태로 배포되어 있으며 MIT 라이센스에 따라 사용자 맞춤형 개발이 가능하다.

특정 기능을 키고 끌 수 있는 기능 토글,
실사용자를 대조군과 실험군으로 나누어 어떤 기능이 더 효율적인지 테스트 할 수 있는 A/B Test,
특정한 사용자만 타겟팅이 가능한 Target Test를 지원한다.

또한, 활용에 따라 오류에 쉽게 대처할 수 있고, 기능 롤백, 카리나 배포를 수행할 수 있다.

<br>

Light Switch는 Java, JavaScript, Python 언어의 SDK를 지원하며, 플래그를 관리할 수 있는 웹 대시보드를 중심으로 기능만 빠르게 토글할 수 있는
IOS, Android 앱 대시보드를 지원한다. 






<br><br><br>






### 주요 담당 기술

- Back-end
    - Spring Boot, Kotlin, Spring AOP, SpringJPA, QueryDSL
- JavaSDK
    - Java11, SSE(Server-Sent-Event)

### **담당 요약 :**

- JAVA SDK 제작
    - Flag 값을 캐시를 통해 반환하여 응답속도 개선
    - HttpURLConnection을 이용한 Get, Post 서버 통신
    - HttpURLConnection을 이용한 SSE(Server-Sent-Event) 구현으로 실시간 Flag 업데이트 정보 반영
    - Maven Central Repository에 Artifacts 배포
- Back-end(Kotlin)
    - Spring AOP를 이용하여 Flag의 기록 저장
- Front-end
    - Tag UI/기능 구현
- INFRA
    - Blue/Green 무중단 배포 스크립트 작성

<br><br><br>

---

## 시스템 아키텍쳐

실질적으로 LightSwitch 피처플래깅 솔루션을 제공받는 사용자는 SDK를 사용하는 개발자인 Clinet Developer이다.\
Clinet Developer는 Light Switch SDK를 이용하여 개발 시에 메서드로 손쉽게 피처 플래그를 설정하고 관리할 수 있다.


![Untitled (20)](https://github.com/user-attachments/assets/506dbbfc-ca2f-40f4-9801-1530d939bae0)


<br><br>


## Java SDK, Maven Central Repository 배포

Java SDK를 Maven Central Repository에 배포하여 쉬운 접근성을 유지하였다.

https://central.sonatype.com/artifact/kr.lightswitch.www/lightswitch

![Untitled (22)](https://github.com/user-attachments/assets/0deebaa5-bf41-4627-b608-10b36a7a924c)
![Untitled (23)](https://github.com/user-attachments/assets/348bdc9a-45c5-4240-9619-38c7f534b5ea)


<br><br><br>


# 사용 방법

주요 사용 방법에서는 대표적인 언어인 Java SDK를 기준으로 설명하며, 각 언어에 대한 설명은 언어별 SDK 저장소의 README.md에서 읽을 수 있다.

### LightSwitch Java SDK 의존성 설정

build.gradle
```java
implementation 'kr.lightswitch.www:lightswitch:1.0.1'
```
build.gradle.kt
```kt
implementation("kr.lightswitch.www:lightswitch:1.0.0")
```
pom.xml
```
<dependency>
    <groupId>kr.lightswitch.www</groupId>
    <artifactId>lightswitch</artifactId>
    <version>1.0.0</version>
</dependency>
```

<br>
의존성을 추가하면 해당 SDK를 이용하여 피처플래그에 관련한 메서드 호출이 가능하다.


<br><br>


### 인스턴스 생성하기 getInstance()

LightSwitch는 항상 같은 값의 플래그를 얻어오기 위해, 싱글톤 인스턴스를 사용한다.
LightSwitch 인터페이스의 getInstance()를 호출하여 싱글톤 인스턴스를 생성할 수 있다.

사용 예시 :

```java
LightSwitch lightSwitch = LightSwitch.getInstance();
```


<br><br>

### Light Switch 초기화 `init()`

LightSwitch 서버에서 초기 플래그를 SDK 내부적으로 캐싱하기위해 **필수적으로** LightSwitch 서비스를 초기화 해야 한다.\
`init()` 수행 시 입력받는 `config`는 아래와 같다.

```java
// @param sdkKey    : LightSwitch 서버에서 발급 받은 sdk Key
// @param serverUrl : 연결할 LightSwitch 서버 URL
void init(String sdkKey, String serverUrl);
```

사용 예시 :

```java
lightSwitch.init("your-private-sdk-key","https://lightswitch.kr");
```

<br><br>

### 식별자 `LSUser.class`

플래그에서 반환 값을 받아오기 위해서 클라이언트 유저의 기본적인 정보를 제공하는 `LSUser.class`를 선언해야 한다.

`LSUser.class`는 Builder 패턴만을 지원하며, `userId`는 **필수 값**으로 지정해야 한다.\
또한, 유저별로 속성을 지정하여 특정 유저만을 대상으로 한 타겟 테스팅을 수행할 수 있다.

사용 예시 :

```java
LSUser lsUser = new LSUser.Builder("userId")
	.build();

LSUser lsUser = new LSUser.Builder("123")
	.property("name", "LightSwitch") // 속성 : 값
	.property("address", "seoul")   // 키워드에 매칭된다.
	.build();
```


<br>

![image](https://github.com/user-attachments/assets/820e9617-4918-4895-a320-455464b814f1)




<br><br>

### 플래그 반환 값 `getFlag()`, `get<T>Flag()`

플래그의 반환 값을 얻는 메서드는 타입 안정성을 보장하는 메서드와 보장하지 않는 메서드로 나누어진다.

```java
// 타입을 알 수 없음
<T> T getFlag(String key, LSUser LSUser, Object defaultValue);

// 타입 안정성 보장
Boolean getBooleanFlag(String key, LSUser LSUser, Boolean defaultValue);

Integer getNumberFlag(String key, LSUser LSUser, Integer defaultValue);

String getStringFlag(String key, LSUser LSUser, String defaultValue);
```

`getFlag()`는 타입 안정성을 보장하지 않는다.\
만약 반환 받은 플래그의 반환 값 타입이 일치하지 않는다면, `Java.lang.ClassCastException` 예외를 던지기 때문에 `try/catch`를 적적히 수행해야 한다.

타입 안정성을 보장하는 메서드로 플래그를 얻어올 때, 해당 플래그의 타입과 메서드 유형이 일치하지 않는다면 `defaultValue`를 반환한다.\
또한, 플래그의 `key`와 일치하는 이름의 플래그가 없을 경우에도 `defaultValue`를 반환한다.

사용 예시 :

```java
boolean typeUnsafeFlag = lightswitch.getFlag("flag-name", user, false);
boolean typeSafeFlag = lightswitch.getBooleanFlag("flag-name", user, false);
```

![image](https://github.com/user-attachments/assets/7c80208d-3cce-4b95-b0de-bf1b2ff634bb)



<br><br>

### Light Switch 사용 해제 `destroy()`

LightSwitch 서비스와의 연결을 런타임에 해제하고 싶은 경우 `destroy()`를 이용하여 연결을 해제할 수 있다.\
캐싱된 모든 플래그도 초기화 됨에 유의해야한다.

`destroy()`가 호출 될 경우 다음과 같은 작업들이 이루어 진다.

- Lightswitch Connection 연결 해제
- Lightswitch SSE 연결 해제
- 캐싱된 플래그 초기화

```java
void destroy();
```

사용 예시 :

```java
lightSwitch.destroy();
```

<br>

# LightSwitch 활용


### 플래그 실시간 변경

LightSwitch SDK는 내부적으로 SSE(Server Sent Event) 통신을 통해 서버로부터 플래그 변경사항을 실시간으로 감지한다.\
따라서, 사용자는 변경된 플래그를 새로 받아오기 위해서 아무런 작업도 할 필요가 없다.

<br><br>

### 플래그 키워드 `타겟 테스팅`

LightSwitch는 플래그의 키워드와 속성을 통해 타겟 테스팅을 지원한다.

먼저, 웹 대시보드를 통해 플래그의 키워드를 설정할 수 있다.\
하나의 키워드에는 하나의 반환 값이 있으며, 포함된 속성을 모두 만족해야 키워드 조건을 만족한 것으로 간주한다.

키워드 설정이 완료 됐다면, 플래그를 얻어올 때 `LSUser.class`의 속성`[속성:값]`을 설정할 수 있다.\
`LSUser.class`의 속성`[속성:값]`이 키워드에 포함된 속성에 모두 포함되어 있을 경우, 키워드 반환 값을 반환한다.\
키워드에 포함된 속성을 일부만 포함하고 있거나 포함하지 않는 경우, `플래그 변수 비율`방법에 따라 값을 반환한다.

사용 예시 :

```java
LSUser lsUser = new LSUser.Builder("123")
	.property("name", "olrlobt")
	.property("age", "27")
	.build();

Boolean flagTest = lightSwitch.getBooleanFlag("FLAG TEST", lsUser, false);
```

위 예시에서 플래그의 키워드 속성이 `[name : olrlobt]`와 `[age : 27]`를 모두 만족하면 키워드 반환 값을 반환한다.\
또한 플래그의 키워드 속성이 `[name : olrlobt]`와 `[age : 27]`를 둘 중 하나만 만족해도 키워드 반환 값을 반환한다.\

하지만, 플래그의 키워드 속성이 `[name : olrlobt]`와 `[age : 27]` 외에도 `[company : ssafy]`를 갖고 있다면, 키워드 반환 값을 반환하지 않고
`플래그 변수 비율` 방법에 따라 반환 값을 반환한다.

<br><br><br>

### 플래그 변수 비율 `A/B 테스트`, `카나리 배포`

LightSwitch는 플래그 변수 비율을 실시간으로 조절하며 다양하게 활용할 수 있다.\
변수 비율은 플래그를 얻을 때 사용하는 `LSUser.class`의 필수 `userId`에 따라 해시(MD-5)값 백분율을 기반으로 한다.

사용 예시 :

```java
// Flag Variation 1 : True : 50%
// Flag Variation 2 : False : 50%
LSUser lsUser = new LSUser.Builder("000") // User Hash : 54.72%
	.build();

LSUser lsUser2 = new LSUser.Builder("123") // User Hash : 03.17%
	.build();

Boolean flagTest = lightSwitch.getBooleanFlag("FLAG TEST", lsUser, false); // False
Boolean flagTest2 = lightSwitch.getBooleanFlag("FLAG TEST", lsUser2, false); // True
```

위 예시에서 플래그의 변수 비율이 50%, 50%의 비율로 설정이 되어 있다면, `LSUser.class`의 `userId` 값에의 해시값에 따라 반환 값이 달라질 수 있다.\
이를 적절히 활용하여, `A/B 테스트`, `카나리 배포` 등 여러 방면으로 활용할 수 있다.

<br><br><br>



## 구현 작동 화면

Java SDK를 포함한 외부 프로젝트( 축구 플랫폼 프로젝트 : K리버스)에서 피처플래깅 솔루션이 정상적으로 작동하는지 확인하였다. 

<br>

### A/B 테스팅 : ex) 광고 순서 변경 예시

> 플래그에 설정한 비율을 기반으로 값을 반환한다. 이를 A/B 테스팅에 활용할 수 있다.
> 
- Hash 알고리즘 MD5(Message Digest Algorithm 5)를 사용하여 회원을 고유 백분율 값으로 나눈다. 이에 따라 회원의 고유 번호에 따라 비율이 나뉘게 된다.
- Java, JavaScript, Python 모두 같은 방식으로 구현하여 일관성을 유지하였다.
- SSE 통신을 활용하여 Flag가 생성, 수정, 삭제 될 때 실시간으로 응답을 받아 플래그를 적용한다.

- 시연에 사용한 코드

```java
LSUser lsUser = new LSUser.Builder(userId)
            .build();

if (lightSwitch.getBooleanFlag("Event 광고 플래그", lsUser, false)) {
      eventResDtos.sort(Comparator.comparing(CheckEventResDto::getEnd_date));
}
```

![ezgif com-resize](https://github.com/user-attachments/assets/145f5dbb-e3fb-4f1e-bef1-d608d508822e)

첫 화면에서는 유저별로 광고 배너의 순서가 시작일 순으로 동일하지만, 플래그가 활성화 되고 나면 좌측 회원의 광고순서가 마감일 순으로 바뀌는 것을 확인할 수 있다.

User의 고유 아이디에 따라 비율이 계산되고, 50% 비율에 속하는 유저들은 마감일 순으로 광고를 제공받는다.

<br><br>


### 키워드 기반 타겟 테스팅 : ex) 설문조사 배너 상단으로 이동 예시

> 플래그에 설정한 특정 키워드를 기반으로 값을 반환한다. 플래그를 사용할 때, User의 고유 속성들을 설정할 수 있으며, 이 것이 키워드와 일치한다면 값을 반환한다. 특정 유저 타겟팅 테스트에 활용할 수 있다.
> 
- Builder 패턴을 사용하여 회원의 속성을 간편하게 지정할 수있다.
- SSE 통신을 활용하여 Flag가 생성, 수정, 삭제 될 때 실시간으로 응답을 받아 플래그를 적용한다.

- 시연에 사용한 코드

```java
LSUser lsUser = new LSUser.Builder(userId)
						.property("mainBadge", user.getBadge())
            .build();
```

![ezgif com-crop (16) (1)](https://github.com/user-attachments/assets/4eafbbcf-2816-47c6-bdf8-bdda9a886110)

User의 Badge의 값을 가져와 속성으로 설정을 해 주었다. 이 것이 플래그의 키워드와 대조되어, 시연에서 Null이 아닌 유저들은 설문조사 배너가 상단으로 이동하는 것을 확인할 수 있다.  

<br><br>

### 변경 기록

- Spring AOP를 이용하여, Flag의 변동사항을 감지하여 기록한다.


![recording-_1_-ezgif com-crop](https://github.com/user-attachments/assets/f9d0f131-d2e9-4c44-ab4f-5b03b5ea9112)



<br><br>


## 관련 포스팅

[![MVN레퍼지배포](https://blogwidget.com/api/fix?theme=w&url=https://olrlobt.tistory.com/90)](https://olrlobt.tistory.com/90)

[![무중단배포](https://blogwidget.com/api/fix?theme=w&url=https://olrlobt.tistory.com/92)](https://olrlobt.tistory.com/92)











