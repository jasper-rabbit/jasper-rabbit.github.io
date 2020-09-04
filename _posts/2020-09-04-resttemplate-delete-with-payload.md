---
title: RestTemplate에서 body를 가진 delete 요청 문제
author: Jasper Ra66it
date: 2020-09-04 22:36:00 +0900
categories: [Blogging,Spring]
tags: [RestTemplate,Spring]
pin: false
---

이메일 발송을 위해 이용하는 외부 서비스 연동을 하던 중, `DELETE` 메서드를 이용하는 API가 있었다. 그런데 해당 api로 넘기는 파라미터는 request의 body를 이용하라고 하는 것이 었다. DELETE 메서드를 이용 할 때 복잡한 값들을 넘겨본 적이 많지 않았던 것 같은데, 내 기억으로는 DELETE 로는 body 값을 넘길 **수 없다고** 알고 있었다. 거기다 `RestTemplate`를 이용해서 테스트를 해보니 역시나 안되는 것이었다. (400 Bad Request는 아니었고, 해당 서비스에서 자체 정의한 응답 코드긴 했다.) 그래서 다시 확인해 보니 요청시 설정한 body 값이 넘어가지 않고 있었다.

```java
HttpHeaders headers = new HttpHeaders();
// ... headers 설정
HttpEntity<String> entity = new HttpEntity<>(jsonBody, headers);
ResponseEntity<String> responseEntity = 
    restTemplate.exchange(url, HttpMethod.DELETE, entity, String.class);
```

그래서 내 생각대로 RestTemplate 역시 DELETE 로는 body 값을 설정해도 넘기지 않는구나 싶었다. 그런데 HTTP DELETE 스펙[^httpspec]을 다시 읽어보니 **사용 할 수 없다**가 아니라, DELETE 에서는 payload에 대해서는 특별히 정의하지 않았다는 것이다. 

>
A payload within a DELETE request message has no defined semantics; sending a payload body on a DELETE request might cause some existing  implementations to reject the request.

그렇다면 구현하는 서버 사이드에서 DELETE 요청의 바디를 사용할지 여부를 결정 할 것이라면, RestTemplate으로 바디가 안보내지는건 왜일까? 

결론부터 말하자면, 스프링 버전 문제라고 볼 수 있다. 현재 내가 사용하는 버전은 많이 오래된 3.2.x 인데, 확인해보니 4.2.x 버전부터는 DELETE에서 body 값이 전달되었다. 물론 그렇다고 DELETE로 body 값을 보내자고 버전을 올릴 필요는 없다. RestTemplate에서 request를 생성하는 역할은 `ClientHttpRequestFactory`가 담당하고 있다. 그리고 RestTemplate는 기본적으로 이 인터페이스의 구현체인 `SimpleClientHttpRequestFactory`[^simpleClientHttpRequestFactory] 클래스를 이용한다. 이 클래스가 구현하고 있는 `createRequest` 메서드에 포함된 `prepareConnection` 메서드를 보면 문제의 원인을 확인 할 수 있다.

```java
if ("PUT".equals(httpMethod) || "POST".equals(httpMethod) || "PATCH".equals(httpMethod)) {
    connection.setDoOutput(true);
}
else {
    connection.setDoOutput(false);
}
```

>참고로 `URLConnection`의 **doOutput**이 request의 body를 보내기(ouput) 위한 플래그 역할이다.

그럼 DELETE 에서도 body를 보내기 위해서는 저 부분에 "DELETE"를 추가해주면 되는 것이다. 간단히 `SimpleClientHttpRequestFactory`를 확장해서 `prepareConnection` 메서드만 오버라이드 해주자.

```java
public class MyClientHttpRequestFactory extends SimpleClientHttpRequestFactory {
    @Override
    protected void prepareConnection(HttpURLConnection connection, 
            String httpMethod) throws IOException {

        super.prepareConnection(connection, httpMethod);
        if ("DELETE".equals(httpMethod)) {
            connection.setDoOutput(true);
        }
    }
}
```

* 추가로 구글링을 하다보니 톰캣 7버전(정확히 버전에 대한 부분은 확인하지 않았다)에서는 기본적으로 DELETE 의 바디값을 파싱하지 않는다고 한다. 즉, 이거 모르고 DELETE 요청의 바디로 파라미터를 받는 api를 만들면 왜 데이터가 안들어오는 한참을 삽질 했을 듯 싶다. 아래 링크 참조[^footnote]. 

[^httpspec]: [RFC 7231](https://tools.ietf.org/html/rfc7231#section-4.3.5)
[^simpleClientHttpRequestFactory]: [SimpleClientHttpRequestFactory source](https://github.com/spring-projects/spring-framework/blob/4.0.x/spring-web/src/main/java/org/springframework/http/client/SimpleClientHttpRequestFactory.java)
[^footnote]: [나만 몰랐던 Http Delete Method payload body 문제](http://blog.leekyoungil.com/?p=390)