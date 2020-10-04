---
title: Spring Security Architecture 내용정리
author: Jasper Ra66it
date: 2020-10-04 14:16:00 +0900
categories: [Blogging,Spring,Security]
tags: [Spring Security]
pin: false
---

오래전 [spring.io](https://spring.io)에 올라왔던 [Spring Security Architecture](https://spring.io/guides/topicals/spring-security-architecture) 의 내용을 정리해 보았다. 읽을 때는 금방 할듯 했는데, 막상 우리말로 적어 보려니 상당히 여럽네. 몇몇 문장은 직역/의역으로 넘기기도 했고.

Spring Security를 프로젝트에 사용해 본 건 몇번 안된다. 제대로 학습 후 사용하기 보다는 급한대로 구글링하면서 적용할 수 밖에 없었다. 또한 Spring Security는 초기 설정하고 구조를 잡고나면 관련해서 거의 볼 일이 없다보니, 다시 사용할 일이 생기면 또 바닥부터 시작할 듯하다.

## Authentication and Access Control

애플리케이션 보안이라고 하면 크게 두가지 나눌 수 있다. 

- 인증 : authentication (who are you?)
- 인가 또는 권한부여 : authorization (what are you allowed to do?)

인증이란 내가(유저) 누구인지를 확인하는 것으로 쉽게 아이디와 비밀번호를 떠올릴 수 있다. 인가(또는 권한부여)는 인증을 완료한 유저에게 권한을 부여할 수 있고, 그 권한에 따라 서비스의 기능(또는 자원 resource)에 접근여부를 관리하는 것이라 볼 수 있겠다. authorization과 관련해서 access control이라는 용어가 등장하기도 하는데, 권한에 따라 자원(기능)에 접근(access) 여부를 제어(control)하는 것이라는 점에서 자연스러운 것으로 보인다.

### Authentication

Spring Security에서 authentication에 관한 핵심 인터페이스는 `AuthenticationManager`이며 단 하나의 메서드만 가지고 있다.

```java
public interface AuthenticationManager {
    Authentication authenticate(Authentication authentication) 
        throws AuthenticationException;
}
```

이 인터페이스는 `authenticate()` 메서드에서 아래 3가지 일 중 한가지를 해야한다고 한다.

1. 입력이 유효한 주체(principal)를 나타내는지 확인 할 수 있는 경우 `Authentication`을 리턴. 
2. 입력이 유효하지 않은 주체(principal)를 나타내는 것으로 생각되면 `AuthenticationException` 예외를 던진다.
3. 결정 할 수 없으면 null을 리턴.

여기서 principal 이라는 단어가 등장하는데, 구글번역과 파파고 모두 '주체'라고 번역을 했다. 문장의 문맥상 어떤 의미인지는 알겠는데 막상 사전을 찾아보면 좀 모호하다. 그런데 `Authentication` 인터페이스를 보면 `Principal`이라는 인터페이스를 상속받고 있다. 이 인터페이스의 주석을 보면 Spring Security에서의 principal의 의미를 명확히 파악 할 수 있다.

> This interface represents the abstract notion of a principal, which can be used to represent any entity, **such as an individual, a corporation, and a login id**.

그리고 authenticate() 메서드에서 던지는 `AuthenticationException`는 런타임 예외로 코드 상에서 잡아서 처리되기 보다는, 웹 UI에서는 인증 실패를 나타내는 페이지를 렌더링하고, 백앤드 HTTP 서비스는 401(Unauthorized)응답을 보내는 것 처럼 어플리케이션 단에서 목적에 따라 일반적인 방식으로 처리하라고 한다.

`AuthenticationManager`의 구현체로는 `ProviderManager`가 일반적으로 사용되고 이놈은 `AuthenticationProvider` 인스턴스들에 위임한다 (문장이 좀 깔끔하지 않는데 뒤에 위임과 관련해서 설명이 나온다). `AuthenticationProvider`는 `AuthenticationManager`와 약간 비슷하지만 주어진 `Authentication` 타입을 지원할 경우 호출자가 조회할 수 있는 추가적인 메서드를 가지고 있다고 한다.

```java
public interface AuthenticationProvider {
    Authentication authenticate(Authentication authentication) throws AuthenticationException;

    boolean supports(Class<?> authentication);
}
```

여담이지만 스프링을 사용하다 보면 인터페이스나 클래스의 이름들이 많이 헷갈린다. 한번 봐서는 머리에 잘 정리가 되지 않는다고 해야하나... AuthenticationManager, ProviderManager, AuthenticationProvider ㅡ.ㅡ;

쨌든 `AuthenticationManager`의 구현체인 `ProviderManager` 클래스를 보면 `AuthenticationProvider`타입의 리스트를 가지고 있다. 그리고 `authenticate()` 메서드에서 이 리스트를 돌면서 AuthenticationProvider 인스턴스의 authenticate()를 호출하므로써 위임을 해주고 있는 것을 볼 수 있다.

```java
@Override
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    Class<? extends Authentication> toTest = authentication.getClass();
    Authentication result = null;
    ...
    for (AuthenticationProvider provider : getProviders()) {
        if (!provider.supports(toTest)) {
            continue;
        }
        ...
        try {
            result = provider.authenticate(authentication);
            if (result != null) {
                copyDetails(authentication, result);
                break;
            }
        }
        ...
    }
    ...
}
```

`AuthenticationProvider` 의 support() 메서드에 Class<?> 인자는 authenticate() 메서드로 전달 되어질 녀석을 지원하는지 여부만 확인하기 위한 실제 Class<? extends Authentication> 이다. 바로 위에서 살펴봤듯이 `ProviderManager`는 `AuthenticationProviders` 체인에 위임하므로써 여러 다른 인증 메커니즘을 지원할 수 있다. 

`ProviderManager`에는 모든 Provider들이 null을 리턴할 경우 사용할 `ProviderManager`(parent라고 부르자)를 지정할 수 있다(없을 수 있음). 만약 이 `ProviderManager`도 사용할 수 없는 경우  `AuthenticationException`이 발생한다.

보호하고자 하는 리소스(protected resources)의 논리적 그룹이 있고(에를 들어 **/api/\*\*** 의 경로 패턴과 일치하는 모든 웹 리소스) 이들 각 그룹마다 자체 전용 `AuthenticationManager`가 있을 수 있다. 이들 각각은 `ProviderManager`이고 parent(위 문단 참조)를 공유한다. 그러면 parent는 모든 Provider에 대한 대체(fallback) 역할을 하는 일종의 'global' 리소스가 된다.

![An `AuthenticationManager` hierachy using `ProviderManager`](/assets/img/authentication.png "An AuthenticationManager hierachy using ProviderManager")

### Customizing Authentication Managers

항상 그렇듯이 Spring Security 역시 설정을 쉽게 해주는 핼퍼를 제공한다. 가장 일반적으로 사용되는 핼퍼는 in-memory, JDBC, LDAP user details 설정이나 custom `UserDetailsService`를 추가하는데 유용한 `AuthenticationManagerBuilder`이다. 아래는 global(parent) `AuthenticationManager`를 설정하는 예제다.

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {
    ... // web stuff here

    @Autowired
    public void initialize(AuthenticationManagerBuilder builder, DataSource dataSource) {
        builder.jdbcAuthentication().dataSource(dataSource).withUser("dave").password("secret").roles("USER");
    }
}
```

위 설정에서의 AuthenticationManagerBuilder는 `@Bean`의 메서드에서 `@Authworied` 되어 있음을 유의해야한다. 이것이 global(parent) `AuthenticationManager`를 빌드하게 만다는 것이다. 이와 반대로 아래와 같이 설정 한다면:

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {
    @Autowired
    DataSource dataSource;
    ... // web stuff here

    @Override
    public void configure(AuthenticationManagerBuilder builder) {
        builder.jdbcAuthentication().dataSource(dataSource).withUser("dave").password("secret").roles("USER");
    }
}
```

configurer의 메서드를 `@Override` 한 `AuthenticationManagerBuilder`는 global 의 자식인 "local" `AuthenticationManager`로 빌드하는데만 사용된다. Spring Boot 애플리케이션에서 다른 빈으로 global 을 `@Authwoired` 할 수 있지만, 명시적으로 직접 노출하지 않는 한 "local"을 이용해서 그렇게 할 수는 없다.

Spring Boot는 default global `AuthenticationManager`를 제공하는데, 하나의 유저만을 가지고 있다.

> The default AuthenticationManager has a single user (‘user’ username and random password, printed at INFO level when the application starts up)

### Autorization or Access Control

인증이 성공하면 권한부여를 할 차례인데, 여기서의 핵심 전략은 `AccessDecisionManager`이다. 프레임워크에서 3가지 구현체를 제공하는데 3가지 모두 `AccessDecisionVoter` 체인에 위임한다. (이건 `ProviderManager`에서의 위임방식과 유사하다) 

정리하면 인증이 완료된 이후 특정 리소스에 접근할 때 권한에 따라 접근을 허용할 것인지 여부를 결정하는 역할을 한다고 볼 수 있겠다. 그리고 제공되는 구현체 3가지를 보자.

- AffirmativeBased : AssessDecisionVoter 중 하나라도 허용하는 경우 접근 권한을 부여
- ConsensusBased : 다수결에 따라
- UnanimousBased : 만장일치에 따라

대략 그림이 좀 그려지는 것 같은데... 기본적인 권한부여 전략을 가지고 그 전략을 구체적으로 결정하는데 사용할 여러 Voter들을 가질 수 있다. (Voter(유권자), vote(투표) 개인적으로는 확~ 와닫지는 않네;) 
그런데 여러 voter의 역할 또는 여러개를 가져야하는 이유가 당장 떠오르지 않는다. 

`AccessDecisionVoter`는 principal을 나타내는 `Authentication`과 접근하고자 하는 자원인 `Object` , 그리고 이 Object와 관련된 설정 속성인 `ConfigAttributes`에 대해서 고려한다. (즉, 이 인터페이스의 메서드 인자로서 이런 정보들을 이용한다는 의미)

```java
public interface AccessDecisionVoter<S> {
    boolean supports(ConfigAttribute attribute);
	boolean supports(Class<?> clazz);
	int vote(Authentication authentication, S object, Collection<ConfigAttribute> attributes);
}
```

`Object`는 제네릭 타입으로 유저가 액세스하려는 모든 것을 의미한다고 보자. 웹 리소스 또는 자바 클래스의 메서드가 가장 일반적인 케이스이다. (이와 관련해서 이 블로그 [Spring Security로 Security 서비스 구축하기 2](https://thecodinglog.github.io/spring/security/2018/08/15/spring-security-service-2.html) 내용을 보고 조금 이해가 되었다.) `ConfigAttributes`는 액세스에 필요한 권한 수준을 결정하는 secure `Object`의 일부 메타 데이터이다. `ConfigAttributes`는 인터페이스며 `String`을 반환하는 메서드 하나만 가진다. 이 문자열은 리소스 소유자가 의도하는 방식으로 인코딩하여 누가 액세스 할 수 있는지에 대한 규칙(rule)을 표현한다. 일반적인 `ConfigAttribute`는 사용자 role의 이름을`ROLE_ADMIN` 또는 `ROLE_AUDIT`)이고 종종 특수한 포멧을(ROLE_ 접두사 같은) 갖거나 평가해야하는 expression을 나타낸다.
 
대부분은 기본 `AccessDecisionManager`인 `AffirmativeBased`를 그냥 사용한다. 이와 관련한 모든 사용자 정의(customization)은 voters에 새로운 voter를 추가하거나 기존 voter의 작동방식을 수정하는 등으로 이뤄진다.

`ConfigAttributes`으로 Spring Expression Language(SpEL) 표현식을 사용하는게 일반적이다. 예를 들어 `isFullyAuthenticated() && hasRole('FOO')`. 이는 표현식을 처리하고 이에 대한 context를 작성할 수 있는 `AccessDecisionVoter`에 의해 지원된다. 처리 할 수 있는 표현식의 범위를 확장하려면 `SecurityExpressionRoot`의 구현이 필요하고 때로는 `SecurityExpressionHandler`도 필요하다.

## Web Security

UI와 HTTP 백엔드를 위한 웹 계층의 Spring Security는 서블릿 `Filters`를 기반으로 한다. 따라서 `Filter`의 역할을 먼저 살펴 보자. 아래 그림은 단일 HTTP 요청에 대한 핸들러의 일반적인 계층을 보여준다.

![Filters](/assets/img/filters.png "Filters")

클라이언트가 앱에 요청을 보내고 컨테이너는 요청 URI의 경로를 기반으로 어떤 필터와 어떤 서블릿을 적용할지 결정한다. 하나의 서블릿이 단일 요청을 처리하지만 필터는 체인을 형성하므로 순서가 지정되며 실제로 요청 자체를 처리하려는 경우 필터가 나머지 체인을 거부 할 수 있다. 필터는 다운스트림 필터와 서블릿을 사용해서 요청과 응답을 수정할 수도 있다. 필터 체인의 순서는 매우 중요하며 Spring Boot는 두 가지 메커니즘을 통해 이를 관리한다. 하나는 `Filter` 타입의 `@Beans`에 `@Order`를 붙이거나 `Orderd`를 구현하는 것이고, 다른 하나는 API의 일부로 순서를 가지는 `FilterRegistrationBean`의 일부가 되는 것이다. 일부 이미 만들어진 필터는 서로 상대적인 순서를 나타내는데 도움이 되도록 상수를 정의한다. (예로 Spring Session의 `SessionRepositoryFilter`는 `Integer.MIN_VALUE + 50`의 값을 가지는 `DEFAULT_ORDER`를 가지며, 이는 체인의 앞단에 있고자 하지만 그 앞에 다른 필터를 배제하지는 않는다.)

Spring Security는 필터 체인에 단일 `Filter`로 설치되고 타입은 `FilterChainProxy`이다. Spring Boot 앱에서 security 필터는 `ApplicationContext`의 `@Bean`이고 모든 요청에 적용되도록 설치된다. `SecurityProperties.DEFAULT_FILTER_ORDER`에 의해 정의된 위치에 설치되며 `FilterRegistrationBean.REQUEST_WRAPPER_FILTER_MAX_ORDER` 값을 이용해서 초기화 되고 있다.(참고로 `REQUEST_WRAPPER_FILTER_MAX_ORDER`는 2.1.0 버전부터 Deprecated 되었고 `OrderedFilter.REQUEST_WRAPPER_FILTER_MAX_ORDER`로 변경 되었다.) 

```java
public static final int DEFAULT_FILTER_ORDER = OrderedFilter.REQUEST_WRAPPER_FILTER_MAX_ORDER - 100;
```

컨테이너의 관점에서 Spring Security는 단일 필터지만 내부적으로는 각각 특별한 역할(role)을 하는 추가적인 필터들이 있다. 아래 그림을 참조:

![Spring Security is a single physical Filter but delegates processing to a chain of internal filters](/assets/img/security-filters.png) 

사실은 security 필터에서 간접적인 계층이 하나 더 있다. 일반적으로 `DelegatingFilterProxy`로 설치되며 Spring @Bean 일 필요는 없다. 이 Proxy는 항상 @Bean인 `FilterChainProxy`에 위임하며 일반적으로 `springSecurityFilterChain`라는 고정된 이름을 사용한다. 필터들의 체인으로 내부적으로 정리된 모든 보안 로직을 포함하는 `FilterChainProxy`다. 모든 필터는 동일한 API를 가지고 있으며(모두 서블릿 규약의 `Filter` 인터페이스를 구현하고 있음) 나머지 체인에 대해 거부 할 기회가 있다.
(좀 더 자세한 내용은 [Spring Security Reference](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#servlet-delegatingfilterproxy)를 한번 읽어보자. 그런데 내가 읽은건 좀 지난 버전이고 이 링크는 최신 버전인데... 내용이 좀 달라보이긴한다.)

Spring Security 필터는 필터 체인 목록을 포함하고 일치하는 첫 번째 체인에 요청을 보낸다. 아래 그림은 요청 경로 일치에 따라 발생하는 처리를 보여준다. 이건 일반적인 방법이긴 하지만 유일한 방법은 아니다. 

![The Spring Security FilterChainProxy dispatches requests to the first chain that matches](/assets/img/security-filters-dispatch.png)

커스텀 security 설정이 없는 바닐라 Spring Boot 애플리케이션은 여러개의(n) 필터 체인이 있다. 첫 번째 (n-1) 체인은 `/css/**` 와 `/images/**` 같은 정적 리소스 패턴과 오류 뷰 `/error`를 무시한다. 마지막 체인은 모든 경로 `/**`와 일치하며, 권한 부여, 예외 처리, 세션처리, 헤더 쓰기 등에 대한 로직을 포함하여 더 활성화 된다. 이 체인에는 기본적으로 총 11개의 필터가 있지만 일반적으로 사용자가 어떤 필터를 언제 사용하는지 고민 할 필요가 없다.

### Creating and Customizing Filter Chains

Spring Boot 앱의 기본 fallback 필터 체인은 순서가 SecurityProperties.BASIC_AUTH_ORDER로 사전정의 되어있다.`security.basic.enabled = false`를 설정하여 완전히 끌 수도 있고, 더 낮은 순서로 다른 규칙을 정의 할 수도 있다. 이를 위해 `WebSecurityConfigurerAdapter` (또는 `WebSecurityConfigurer`) 타입의 `@Bean`을 추가하고 `@Order`를 클래스에 붙여라. 

```java
@Configuration
@Order(SecurityProperties.BASIC_AUTH_ORDER - 10)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.antMatcher("/foo/**")
        ...;
    }
}
```

이 빈은 Spring Security가 새로운 필터 체인을 추가하고 순서를 fallback 이전으로 지정하고 있다.

많은 애플리케이션은 다른 리소스와 비교 할 때 한 리소스 집합에 대해 완전히 다른 액세스 규칙을 가지고 있다. 예를 들어 UI 및 backing API를 호스팅하는 애플리케이션은 UI부분에 대해서는 로그인 페이지로 리다이렉션되는 쿠키 기반 인증을, API 부분에서는 인증되지 않은 요청에 대한 401 응답을 사용하는 토큰 기반 인증을 지원 할 수 있다. 각 자원 세트에는 고유한 순서와 자체 요청 matcher가 있는 자체 `WebSecurityConfigurerAdapter`가 있다. 일치하는 규칙이 겹치면 가장 빠른 순서의 필터 체인이 우선한다.

### Request Matching for Dispatch and Authorization

보안 필터 체인(또는 `WebSecurityConfigurerAdapter`)에는 HTTP 요청에 적용할 것 인가를 결정하는데 사용되는 request matcher가 있다. 특정 필터 체인을 적용하기로 결정하면 다른 필터 체인이 적용되지 않는다. 그러나 필터 체인 내에서 `HttpSecurity` 설정에 추가로 matcher를 설정하여 권한 부여를 보다 세밀하게 제어 할 수 있다.

```java
@Configuration
@Order(SecurityProperties.BASIC_AUTH_ORDER - 10)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.antMatcher("/foo/**")
      .authorizeRequests()
        .antMatchers("/foo/bar").hasRole("BAR")
        .antMatchers("/foo/spam").hasRole("SPAM")
        .anyRequest().isAuthenticated();
  }
}
```
Spring Security를 설정 할 때 저지르는 가장 흔한 실수 중 하나는 이러한 matcher가 다른 프로세스들에 적용된다는 사실을 잊는 것이다. 하나는 전체 필터 체인에 대한 request matcher이며 다른 하나는 적용 할 액세스 규칙을 선택하는 것이다.

### Combining Application Security Rules with Actuator Rules

Spring Boot Actuator를 사용할 때 이 역시 기본적으로 보안에 안전하다. Spring Security가 적용된 애플리케이션에 Actuator를 추가하면 actuator endpoint에만 적용되는 필터 체인이 추가된다. actuator endpoint에만 일치하는 request matcher로 정의 되며, 디폴트 `SecurityProperties` fallback 필터보다 5만큼 낮은 `ManagementServerProperties.BASIC_AUTH_ORDER`의 순위를 가진다. 따라서 ballback 전에 참조된다.

만약 actuator endpoint에도 나의 security rule을 적용하려면 actuator 순위보다 빠른 순위로 필터 체인을 추가하면 된다. 그게 아니고 actuator endpoint에 대해서는 디폴트 security 세팅을 더 선호한다면 내가 추가할 필터 체인의 순위를 actuator와 fallback 사이의 순위로 추가하면 된다. (예: ManagementServerProperties.BASIC_AUTH_ORDER + 1)

```java
@Configuration
@Order(ManagementServerProperties.BASIC_AUTH_ORDER + 1)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.antMatcher("/foo/**")
        ...;
    }
}
```

## Method Security

Spring Security는 자바 메서드 실행에 대해서도 접근 rule의 적용을 지원한다. 접근 rule을 정의 하는 방식은 동일하지만 코드상에서의 위치가 다르다. 메서드 보안을 활성화 시키기 위해서는:

```java
@SpringBootApplication
@EnableGlobalMethodSecurity(securedEnabled = true)
public class SampleSecureApplication {
}
```

그리고 메서드 레벨에:

```java
@Service
public class MyService {
    @Secured("ROLE_USER")
    public String secure() {
        return "Hello Security";
    }
}
```

위 클래스는 보안 메서드를 가지고 있는 서비스다. 위 타입의 `@Bean`을 만들면 프록시가 적용된다. 따라서 이 메서드가 호출되면 실재 메서드 호출 전에 security 인터셉터를 통하게 된다. 만약 접근이 거부되면 `AccessDeniedException`가 발생한다.

메서드에 보안관련 제약을 강제하기 위한 어노테이션이 더 있다. 특히 `@PreAuthorize`, `@PostAuthorize`를 사용하여 메서드 매개 변수에 대한 참조를 포함하는 표현식을 작성할 수 있다.

## Working with Threads

Spring Security는 기본적으로 스레드 바운드다. 기본적인 빌딩 블럭은 `Authentication`을 가지는 `SecurityContext`다 (유저가 로그인하면 명시적으로 `authenticated` 된 `Authentication`이 된다). `SecurityContextHoler`의 static 메서드를 통해 언제든지 `SecurityContext`에 접근하고 조작 할 수 있다. 이는 단순히 ThreadLocal을 조작하는 것 이다.

```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
assert(authentication.isAuthenticated);
```

이런 코드는 유저 애플리케이션 코드에 일반적인 것은 아니지만, 사용자 정의 인증 필터를 작성하기 위해 유용하다. 

웹 endpoint에서 현재 인증된 사용자에 대한 접근이 필요한 경우 @RequestMapping에서 메서드 매개 변수를 사용할 수 있다.

```java
@RequestMapping("/foo")
public String foo(@AuthenticationPrincipal User user) {
    ... // do stuff with user
}
```

위 코드는 `SecurityContext`에서 현재 `Authentication`를 가져오고 거기서 getPrincipal()메서드를 호출하여 메서드 파라미터를 생성하게 된다. `Authentication`에서 `Principal`의 타입은 인증 유효성을 검사하는데 사용되는 `AuthenticationManager`에 따라 달라진다. 그러니 사용자 데이터에 대한 type-safe 한 참조를 얻는데 유용한 방식이다.

Spring Security가 사용 중인 경우 `HttpServletRequest`의 `Principal`은 `Authentication` 타입이므로 직접 사용할 수도 있다.

```java
@RequestMapping("/foo")
public String foo(Principal principal) {
    Authentication authentication = (Authentication) principal;
    User = (User) authentication.getPrincipal();
    ... // do stuff with user
}
```

이는 Spring Security가 사용되지 않는 상태에서 코드를 작성할 때 유용하다. (`Authentication` 클래스를 로딩하는데 더 방어적이어야 한다.)

### Processing Secure Methods Asynchronously

SecurityContex는 스레드 바운드이므로 `@Async`가 붙은 메서드와 같이 비동기로 백그라운드에서 처리되는 secure 메서드를 호출해야 한다면 해당 컨텍스트도 전파되어야 한다. 이는 백그라운드에서 실행되는 작업 (`Runnable`, `Callable` 등)과 `SecurityContext`를 래핑하는 것으로 요약할 수 있다. Spring Security는 `Runnable` 및 `Callable`에 대한 wrapper와 같은 몇 가지 도우미(helper)를 제공한다. `SecurityContext`를 `@Async` 메서드를 전파하려면 `AsyncConfigurer`를 설정하고 `Executor`가 옳은지 확인해야한다.

```java
@Configuration
public class ApplicationConfiguration extends AsyncConfigurerSupport {
    @Override
    public Executor getAsyncExecutor() {
        return new DelegatingSecurityContextExecutorService(Executors.newFixedThreadPool(5));
    }
}
```