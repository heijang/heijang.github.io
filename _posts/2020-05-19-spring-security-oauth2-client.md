---
title:  "Spring Security OAuth2 Client"
excerpt: "Spring Security OAuth2 Client 따라하기"

categories:
  - SpringSecurity
tags:
  - Spring
  - Security
  - OAuth2
  - Client
  - Java
last_modified_at: 2020-05-19T16:52:00-00:00
toc: true
toc_min: 1
toc_max: 2
toc_sticky: true
toc_label: "페이지 목차"
author_profile: false
---
[root]: http://localhost:8080/

요즘 많은 웹 사이트들의 로그인 화면에서 아래와 같은 기능을 제공하고 있다.
- Login with Google
- Login with Facebook
- 네이버아이디로 로그인

Spring Security OAuth2 Client는 대표적인 인증시스템(구글, 페이스북 등) 또는 별도의 인증시스템과의 OAuth 인증 연동을 할 수 있도록 기능을 제공한다.

OAuth 인증과 Spring Security OAuth2 Client에 대해서 알아보자.

# OAuth 인증

## OAuth 인증 연동 이유

### 시스템 입장
- 사용자 정보를 관리하는 것은 부담되는 일이다.<br>
  (개인정보 동의, 비밀번호 암호화 저장 등 해야 할 일이 많다.)
- 사용자는 우리의 시스템을 신뢰하지 않는다.<br>
  (회원가입 자체를 꺼리는 고객도 다수 존재한다.)

### 사용자 입장
- 내 계정 정보를 신뢰할 수 없는 시스템에 입력 하는 것은 불안하다.<br>
  (모든 아이디, 비밀번호를 동일하게 쓰고 있는데 이상한 짓 하는거 아니야?)
- 매번 로그인 하는 것은 귀찮은 일이다.<br>
  (하지만 구글이나 페이스북은 항상 로그인 되어 있다.)

별도의 회원가입 없이 믿을 수 있는 구글이나 페이스북을 통해 로그인 하고 시스템을 이용할 수 있도록 한다.

# 환경 설정

## 프로젝트 생성
   
   Spring Initializr 또는 IDE 를 통해 Spring Boot 프로젝트를 생성한다.

## build.gradle 설정

   Spring Initializr 생성 시 선택 하거나 build.gradle 에 직접 설정 할 수 있다.
   - spring-boot-starter-web
   - spring-boot-starter-oauth2-client

```groovy
dependencies {
    implementation: 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'
    testImplementation('org.springframework.boot:spring-boot-starter-test') {
        exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
}
```

## application.yml 설정
   
구글 인증 클라이언트 정보를 설정한다.<br>
구글 인증 설정 전이므로 임의의 값을 우선 입력한다.

application.properties 파일을 application.yml 로 변경하면 yaml 형식으로 사용 가능하다.

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: myid
            client-secret: mysecretkey
```

springframework logging 레벨을 debug로 설정하면 Spring Security 동작을 확인 할 수 있다.

```yaml
logging:
  level:
    org.springframework: debug
```

## 프로젝트 실행

이제 프로젝트를 실행해보자.<br>
아래 gradle 실행 또는 IDE 의 Spring Boot Run 을 실행하면 된다.

```
$ ./gradlew bootrun
```

실행 하면 기본 8080 포트로 웹 프로젝트가 기동 되고 [루트 페이지][root]로 접속 하면 아래와 같이 승인 오류가 발생한다.

![승인오류](https://i.imgur.com/Nr4XK83.png)

클라이언트를 찾을 수 없다고 나온다. 그럼 이제 구글 클라이언트를 만들어 보자.

# 구글 클라이언트 설정

구글 클라이언트를 만들기 위해서는 구글 개발자 계정이 필요하다.<br>
등록 된 서비스 만이 구글 인증 서비스를 사용할 수 있어야 하므로 클라이언트 정보를 등록 해야 한다.

[구글 개발자 콘솔](https://console.developers.google.com/) 에서 등록 가능하다.

## 구글 클라이언트 등록
- 새 프로젝트 생성
- OAuth 동의 화면
- 사용자 인증 정보
  - [OAuth 2.0 클라이언트 ID] 등록
  - 승인된 자바스크립트 출처 URI 등록 : http://localhost:8080
  - 승인된 리디렉션 URI 등록 : http://localhost:8080/login/oauth2/code/google

## 클라이언트 정보 설정
생성한 클라이언트 ID(client-id)와 클라이언트 보안 비밀(client-secret)을 application.yml 에 입력한다.

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: 1079243940002-jei4nrugdn42evmc2ftol6j1nl1vhrrl.apps.googleusercontent.com
            client-secret: B8f70jk8rT_dsk7ze11PO6sB
```

## 프로젝트 실행
프로젝트를 재 기동 한 후 다시 접속하면 [루트 페이지][root]가 정상 호출 된다.

![정상에러페이지](https://i.imgur.com/jqV7XAs.png)

구글 로그인 상태에 따라 구글 로그인 페이지 또는 승인 페이지가 나온다.

# 자동 설정 소스 설명

Spring Boot 자동 설정 된 Security 설정 중 일부를 확인해보자.

## OAuth2ClientProperties
> [Github 링크](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/oauth2/client/OAuth2ClientProperties.java)

인증 등록(Registration)이 제공자(Provider) 별로 관리 되는 구조이다.<br>
앞서 application.yml 에서 google 명칭으로 registration 을 설정했으므로 google Provider 가 사용된다.<br>
필요한 경우 임의의 Provider를 등록 가능할 것이다.

```java
package org.springframework.boot.autoconfigure.security.oauth2.client;

@ConfigurationProperties(prefix = "spring.security.oauth2.client")
public class OAuth2ClientProperties {

  /**
	 * OAuth client registrations.
	 */
	private final Map<String, Registration> registration = new HashMap<>();

  ... 생략 ...

	public static class Registration {
		private String provider;
		private String clientId;
		private String clientSecret;
		private String clientAuthenticationMethod;
		private String authorizationGrantType;
		private String redirectUri;
		private Set<String> scope;
		private String clientName;

    ... 생략 ...

  }
}
```


## CommonOAuth2Provider

> [Github 링크](https://github.com/spring-projects/spring-security/blob/master/config/src/main/java/org/springframework/security/config/oauth2/client/CommonOAuth2Provider.java)


GOOGLE, GITHUB, FACEBOOK, OKTA 인증 서버에 대해서 기본 설정 되어 있고 클라이언트 정보만 입력하면 인증 연동이 가능하다.<br>
그 외 인증서버와 연동하기 위해서는 별도 설정이 필요하다.<br>
DEFAULT_REDIRECT_URL 에 등록 된 /login/oauth2/code/{registrationId} 경로로 인증 code 정보가 리다이렉트 된다.

```java
package org.springframework.security.config.oauth2.client;

public enum CommonOAuth2Provider {

	GOOGLE {

		@Override
		public Builder getBuilder(String registrationId) {
			ClientRegistration.Builder builder = getBuilder(registrationId,
					ClientAuthenticationMethod.BASIC, DEFAULT_REDIRECT_URL);
			builder.scope("openid", "profile", "email");
			builder.authorizationUri("https://accounts.google.com/o/oauth2/v2/auth");
			builder.tokenUri("https://www.googleapis.com/oauth2/v4/token");
			builder.jwkSetUri("https://www.googleapis.com/oauth2/v3/certs");
			builder.userInfoUri("https://www.googleapis.com/oauth2/v3/userinfo");
			builder.userNameAttributeName(IdTokenClaimNames.SUB);
			builder.clientName("Google");
			return builder;
		}
	},

	GITHUB {...},

	FACEBOOK {...},

	OKTA {...};

	private static final String DEFAULT_REDIRECT_URL = "{baseUrl}/{action}/oauth2/code/{registrationId}";

  ...
}
```

> 참고자료 : [Spring Security Reference Guide](https://docs.spring.io/spring-security/site/docs/5.3.2.RELEASE/reference/html5/#oauth2)