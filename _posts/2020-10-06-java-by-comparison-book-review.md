---
title: "[책리뷰] 자바 코딩의 기술"
author: "Jasper Ra66it"
date: 2021-01-06 00:46:00 +0900
categories: [Blogging,Java,Book]
tags: [Java, Book, Review]
pin: false
---

> 바보도 컴퓨터가 이해하는 코드는 작성할 수 있다. 훌륭한 프로그래머는 인간이 이해하는 코드를 작성한다. - 마틴 파울러

길벗 출판사에서 진행한 "개발자 리뷰어"를 통해 읽게 된 책이다. 이 책 [자바 코딩의 기술](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=246571219)은 번역서로 원서는 [Java By Comparison](https://www.amazon.com/Java-Comparison-Become-Craftsman-Examples/dp/1680502875) 이고 2018년에 나온 책인 것 같다. 부재가 "현장에서 뽑은 70가지 예제로 배우는 코드 잘 짜는 법" 인데, 원서의 부재로는 "Become a Java Craftsman in 70 Examples" 이다. 저자가 3명이나 되고, 솔직히 눈에 익은 이름은 아니었다.

우선 이 책의 장점은 군더더기 없이 간결한 구성에 있다고 본다. 목차를 보면 각 챕터별 제목만으로 어떤 내용인지 파악 할 수 있고, 필요한 것만 찾아 보기도 좋다. 거기다 각 챕터별로 우선 '문제가 있는 코드 예제'를 보여 주고, 잘 못 된 이유를 설명한 후 '수정 된 코드 예제'를 보여준다. 원서의 제목처럼 **비교**를 통해 핵심사항을 쉽게 전달하고 있다. 그렇다보니 다 읽은 후 "예제가 70개나 되었나?" 생각이 들정도로 쉽고 빠르게 읽혀진 것 같다.

전체적인 난이도는 초급정도 라고 할 수 있을 것 같다. 그리고 물론 자바에 특화된 내용들도 있긴하지만 프로그래밍의 기본적인 코딩 컨벤션에 대한 내용이라고도 할 수 있을 것 같다. 입문자뿐만 아니라 개발 경력이 있지만 스스로 이런 부분에 관심을 둔적이 없었던 사람들이라면 꼭 한번 읽어 봤으면 하는 책이다. 개인적으로 재미있었던 것은 이미 퇴사한 경력 3년차 개발자의 코드를 보고 (기능상에는 문제가 없지만) "왜 이렇게 작성한 걸까?" 하는 의문을 가진 코드가 이 책의 첫 주제로 나왔다는 것이다. (*1.1 쓸모없는 비교 피하기*) 

```java
if (microscope.isInorganic(sample) == true) {
    ...
}
```

물론 나 역시 이 책을 읽고 "다음엔 이렇게 해야겠군" 생각하며 작은 포스트잇으로 표시해 둔 것도 많았다. 예를 들어 *3.5 구현 결정 설명하기* 에서 [ADR(Architecture Decision Records)](https://adr.github.io/)에 대해 처음 알게 되었는데, 시간 날때 여기서 제공하는 도구나 템플릿을 어떻게 활용 할 수 있을지 검토해 보려고한다. 이 뿐만 아니라 테스트 코드에 대한 내용(*6장 올바르게 드러내기*)과 자바8 기반의 스트림이나 람다, 옵셔널에 대한 내용도 담겨있다. 마지막장(*9장 실전 준비*)에서 소개하는 각종 툴들도 깊게 다루진 않지만 참고할만 하다.

> 물론 \<이펙티브 자바\> 와 \<클린 코드\> 처럼 자바의 코드 품질과 가독성, 유지보수성, 클린 코드를 다룬 고급 책을 이미 접했다면 이미 한 발 멀리 나아갔다고 할 수 있습니다. 그래도 이책에서 새로운 내용을 더 찾을 수 있을 것이고 꼭 그럴 거예요.

책의 저자도 머릿말에 말하고 있지만, 이미 이펙티브 자바나 클린코드와 같은 책들을 섭렵하고 있는 분들께는 사실 너무 쉽고 당연한 내용들일 수 있다. 하지만 입사한지 얼마안된 신입 개발자나, 뭔가 부좀함이 느껴지는 주니어 개발자에게 추천해 줄 만한 책이 아닐까 생각해본다.