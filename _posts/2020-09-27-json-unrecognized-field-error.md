---
title: Jackson UnrecognizedPropertyException - Unrecognized field 
author: Jasper Ra66it
date: 2020-09-27 00:55:00 +0900
categories: [Blogging,Spring]
tags: [RestTemplate,Jackson,Json,MessageConverter]
pin: false
---

```java
Caused by: com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: Unrecognized field "$xxxxx", not marked as ignorable ...
```

연동하던 외부 서비스 api 의 json 응답값에 새로운 필드가 추가된 모양이다. DTO에 정의된 필드의 값만 바인딩 되도록 해뒀어야 하는데, 또 까먹은 모양이다. (이게 ObjectMapper의 디폴트 값이면 안되는 것인가...)

방법은 두가지가 있는데, 간단하게는 DTO에 어노테이션을 붙이기만 하면 된다.

```java
@JsonIgnoreProperties(ignoreUnknown = true)
public class MyDTO {
    private String field1;
    ....
}
```

다른 한 가지는 사용하는 ObjectMapper에 설정을 하는 것이다. 
```java
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
```

참고로 RestTemplate을 사용하면서 이 설정을 해주고 싶다면, 위와 같이 정의한 ObjectMapper와 함께 MappingJackson2HttpMessageConverter 를 만들어서 RestTemplate에 설정 해주면 된다.
```java
RestTemplate restTemplate = new RestTemplate();    
MappingJackson2HttpMessageConverter messageConverter = new MappingJackson2HttpMessageConverter();
messageConverter.setObjectMapper(objectMapper);
restTemplate.getMessageConverters().add(0, messageConverter);
```

[Jackson Annotations - [Jackson Docs]](https://github.com/FasterXML/jackson-annotations/wiki/Jackson-Annotations#property-inclusion)

[4.1. Configuring Serialization or Deserialization Feature - Intro to the Jackson ObjectMapper \| Baeldung](https://www.baeldung.com/jackson-object-mapper-tutorial#1-configuring-serialization-or-deserialization-feature)