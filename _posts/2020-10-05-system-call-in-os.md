---
title: System call 이란?
author: Jasper Ra66it
date: 2020-10-05 23:13:00 +0900
categories: [Blogging,OS,java]
tags: [System call, OS, NIO, java]
pin: false
---

최근 자바 NIO 관련 책을 보고 있는데 자연스럽게 System Call 이 가끔 언급된다. 물론 OS와 프로그램 사이에서 low-level의 기능이 필요할 때 사용하는 인터페이스라는 정도로 대충 이해 할 수 있겠다.  

앞 문장을 좀 정리 해보면, '프로그램' 이라기 보다는 사용자의 프로그램이 실행되고 있는 '프로세스 Process' 라고 하는게 좀 더 정확한 것 같다. 그리고 low-level의 기능이라 함은 OS의 커널이 제공하는 '서비스 Service' 라고 하면 되겠다. 지금 보고 있는 자바 책에서도 '커널로부터 제공받는 서비스' 라는 표현이 반복해서 나온다. 그리고 System call은 미리 정의된 API를 통해서 이뤄지고 결과적으로 커널로의 유일한 진입점인 것이다.

이런 System call이 할 수 있는 일은:

- 파일을 읽거나 쓰고
- 프로세스를 생성 및 관리
- 네트워크를 통한 패킷 송수신
- 프린터 같은 하드웨어 디바이스를 사용

그에 따라 System call은 몇가지 타입으로 나눠진다고 한다.

- Process Control : end, abort, create, terminate, allocate and free memory
- File Management : create, open, close, delete, read file
- Device Management
- Information Maintenance
- Communications

아래 System call 표는 [Introduction of System Call - geeksforgeeks](https://www.geeksforgeeks.org/introduction-of-system-call/) 사이트에서 가져왔다.

![](/assets/img/examples-system-calls.png "An AuthenticationManager hierachy using ProviderManager")

좀 더 자세한 사항은 [System Call in OS: Types and Examples - guru99](https://www.guru99.com/system-call-operating-system.html) 여기를 참고하자.