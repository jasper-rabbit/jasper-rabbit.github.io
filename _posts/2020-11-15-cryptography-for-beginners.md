---
title: Cryptography for Beginners 정리
author: Jasper Ra66it
date: 2020-11-15 01:25:00 +0900
categories: [Blogging,Cryptography]
tags: [Cryptography, Cipher, Symmetric Algorithm, Asymmetric Algorithm]
pin: false
---
프로그래밍을 하면서 항상 가려운 부분 중 한 곳이 암호화 관련이다. 전혀 사용 안하는 것도 아니지만, 사용할 때는 기존에 만들어진 유틸리티 클래스의 메서드들을 사용 하거나 구글링이 답이다. 하지만 업무나 관심사 중 암호화 관련한 부분은 (중요도는 무시하고) 대략 5% 정도이지 않을까. 그러니 공부를 한다해도 지나고나면 다 까먹을 수 밖에 없다. 

[dev.to](http://dev.to) 에서 [Cryptography for Beginners](https://dev.to/aakatev/cyptography-for-beginners-1el7?signin=true) 이란 글을 보고 정리를 해보려고 한다. 특히 마음에 들었던 부분은 용어 설명을 먼저 '간단하게' 하면서 시작한다는 것이다. 한번 용어부터 알아보자.

## Glossary
**Cryptography** - 암호화(암호학) 정도로 읽혀지는데, 이걸 풀어서 쓰면 `'안전한 통신을 위한 기술의 관례와 연구'` 라는 딱딱한 문장이 나온다. **crypto** 나 **cryptology** 라고도 한다. 관련 글들을 읽으면 다양한 단어가 사용되기도 하는데, 이렇게 같은 의미로 받아들이면 되겠다.

**Cipher** - 부끄럽지만, 위에서 말한 것 처럼 언제 만들어졌는지 모르는 유틸리티 클래스의 메서드를 사용할 때 눈에 들어오는 클래스들 중 하나인데, 영어 단어 자체부터 익숙하지 않았고, 찾아보려고도 안했던 것 같다. 단어를 영어 사전에서 찾아보면 '(글로쓰인) 암호' 라고 되어있다. 영영사전을 보면 'A cipher is a secret system of writing that you use to send messages. (=code)' 라고 나온다. 암호화 된 결과물을 의미한다고 보면 될까? 하지만 이 문서에서는 이렇게 정의하고 있다. `'암호화 또는 복호화를 실행하기 위한 알리고리즘'` 풀어보자면, 암호화와 복호화를 위한 여러 알고리즘들이 존재하는데 이 중 어떤 알고리즘을 사용하는지를 의미하는 것 같다. 영어 사전의 정의와는 다소 차이가 있는 것 같다.

**Plaintext** - 암호화되지 않은(unencrypted) 메세지 또는 정보. **cleartext** 라고도 한다.

**Ciphertext** - 암호화 된(encrypted) 메세지 또는 정보. 위의 Cipher 의 사전적 의미로 보여진다.

**Encryption** - 메세지나 정보를 인증 된 쪽에서만 읽을 수 있는 형식으로 **인코딩(encoding)**하는 과정(process). 간단하게 암호화 하는 과정이나 단계를 의미한다. 여기서 '인코딩'이란 단어를 한번 보자. encode 라는 단어는 사전에는 '암호로 바꾸다', '(컴퓨터 용어로) 부호화하다' 라고 나온다. 즉, 평문을 암호화 알고리즘에 따라 변환되는 코드(부호)로 바꿔주는 것을 의미한다. 그리고 이러한 단계의 과정을 통틀어 Encryption 이라고 알면 되겠다.

**Decryption** - 이건 Encryption의 반대라고 보면 될 것 같다. ciphertext 에서 plaintext로 변환(converting)하는 과정. 

**Random Number Generator (RNG)** - 그냥 읽어보면 '랜덤 숫자 생성기'인데, 해당 문서에서는 `'어떠한 패턴도 없는 연속된 숫자들을 생성하기 위해 설계된 장치(device)'` 라고 되어있다. 장치라니, 프로그래밍의 알고리즘으로 생성하는 랜덤한 숫자(난수)는 어디까지나 Pseudo Random(의사난수) 이라고한다. 진정한 난수를 만들기 위해서는 특수한 하드웨어를 이용해야 한다고 하는데, 자세한건 구글링 해보도록 하자.

**Key** - 암호화 cipher(위의 정의대로라면 알고리즘 이라고 생각하면) 의 기능적인 결과를 결정하는 매개변수 라고한다. 특정 알고리즘을 사용하더라도 이를 이용하여 암호화되는 결과물은 이 매개변수 값에 따라 결정 된다.

**Hash Function** - 거꾸로 되돌리는게 사실상 불가능하다고 간주되는 단방향(one-way) 암호화 기능. one-way 라는 것에 주목하자. [https://ko.wikipedia.org/wiki/해시_함수](https://ko.wikipedia.org/wiki/%ED%95%B4%EC%8B%9C_%ED%95%A8%EC%88%98) 참고

**Digest** - 이 단어도 cipher 와 함께 잘 와닿지 않는 단어 중 하나였다. 사전적으로는 '소화하다, 요약(문)' 이라고 되어있는데, ~~항상 '리더스 다이제스트' 라는 잡지가 떠오른다.~~ 문서에서는 위의 'Hash Function의 결과값' 이라고 되어있다.

**Symmetric Algorithm** - Symmetric(대칭) 알고리즘. 암호화 및 복호화를 위해 동일한 암호화 키를 사용하는 알고리즘이다. 

**Asymmetric Algorithm** - 비대칭 알고리즘. 대칭 알고리즘과 반대로 암호화 와 복호화 마다 다른 암호화 키를 사용하는 알고리즘이다. public-key(공개키) 알고리즘이라고도 한다.

## Symmetric Ciphers

![https://res.cloudinary.com/practicaldev/image/fetch/s--dO0UTS2X--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/mti4n72t6k03a26cjkkg.png](https://res.cloudinary.com/practicaldev/image/fetch/s--dO0UTS2X--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/mti4n72t6k03a26cjkkg.png)

대칭 암호화 알고리즘에도 두가지 타입이 있다고 한다.

- **Block Cipher** - block 이라고 부르는 고정길이 비트 그룹상에서 작동(operates)하는 타입
- **Stream Cipher** - 비트 스트림에서 작동하는 타입.

### Common Block Ciphers

자주 보는 DES, AES 가 여기서 등장한다. 얘들은 block cipher 타입이구나.

**Data Encryption Standard (DES)** 

현대 암호화에서 DES는 전자 통신(electronic communication)의 보안을 위한 최초의 표준화 알고리즘(cipher) 이라고 한다. 현대의 컴퓨터 처리 능력으로 인해서 DES는 더이상 사용되지 않는다. 2-Key 또는 3-Key 3DES의 변형으로 사용할 수 있다.

**Advanced Encryption Standard (AES)** 

전자 데이터(electronic data)의 암호화를 위해 2001년에 US National Institute of Standards and Technology(NIST, 미국 국립표준기술연구소) 에서 설정한 표준이라고 한다. 1977년부터 사용되던 DES를 대체했다. AES는 키 길이에 따라서 다양한 형식을 가진다. AES-128, AES-192, AES-256으로 줄여서 사용되며 뒤의 숫자는 키의 비트 길이를 나타낸다.

### Common Stream Ciphers

**Rivest Cipher 4 (RC4)** : 이 단어 자체는 처음 보는 것 같다.

RC4 stream cipher는 WEP(Wired Equivalent Privacy) 및 WPA (Wi-Fi Protected Access) 및 TLS (Transport Layer Security)를 포함한 다양한 프로토콜에서 사용되고 있다. 그러나 2015년에 발견 된 일부 취약점으로 인해 사용량이 감소하기 시작했다. RFC 7465는 모든 버전의 TLS에서 RC4 사용을 금지한다.

## Asymmetric Ciphers

![https://res.cloudinary.com/practicaldev/image/fetch/s--ha7iLphD--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/ccworjqwdr8tqfqm1iuv.png](https://res.cloudinary.com/practicaldev/image/fetch/s--ha7iLphD--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/ccworjqwdr8tqfqm1iuv.png)

스트림과 블럭이라는 용어는 일반적으로 비대칭 암호와 함께 사용되지 않는다. 즉, 공개 키 암호화는 데이터 블록도 암호화한다. 예를 들자면 블럭 크기가 키 크기를 기반으로 하는 RSA가 있다.

### Common Asymmetric Ciphers

**Rivest-Shamir-Adleman (RSA)**

가장 오래되고 가장 널리 사용되는 배대칭 알고리즘 중 하나이다. 두개의 매우 큰 소수를 곱한 결과 값에서 원래의 두 소수를 도출하는 것이 거의 불가능하다는 사실을 기반으로 한다. 사실, 768 비트 RSA 키를 해동하는데 약 1500년의 컴퓨팅 시간이 걸린다고 발표된 연구가 있다.

**Elliptic Curve Cryptography (ECC)**

RSA와 마찬가지로 ECC는 비가역성(irreversibility) 원칙에 따라 작동한다. ECC에서는 곡선의 한 점을 상징하는 숫자에 다른 숫자를 곱하여 곡선의 다른 점을 제공한다. 이 퍼즐을 풀기 위해 곡선의 새로운 점을 알아 내야한다.  ECC의 수학은 원래 점을 알고 있더라도 새로운 점을 찾는 것이 사실상 불가능한 방식으로 만들어졌다.

## Symmetric vs. Asymmetric Ciphers

이 둘의 비교는 다른 무엇보다 기본적인 작업(암호화 및 복호화)의 비용에 관현 비교라고 할 수 있다. 대량의 데이터에 대해 비대칭 암호화를 수행하는 것은 엄청나게 큰 비용이 든다. 대칭 키는 저렴한 계산을 제공하지만 키를 공유한다는 문제가 있다.(the problem of a shared secret.) 공개 키는 비싼 계산을 제공하지만 안전하게 통신하는데 필요한 정보를 쉽게 공유할 수 있다.

이러한 알고리즘들의 사용의 효과는 다른 추상화 계층을 생성하는 것에서 부터 나타난다. 다른 말로 연구원들은 이러한 알고리즘을 기본으로 사용하여 시스템을 구축해야만 한다.

예를 들어 다음과 같은 하이브리드 접근방식은 유효하다.

**Key encryption** - 공개 키 암호화를 사용하여 즉석에서 생성된 대칭키를 암호화한다. 비용이 많이 드는 작업이지만 한번만 수행한다.

**Data encryption** - 그리고 위에서 암호화된 대칭키를 사용해서 실제 데이터를 암호화 및 복호화한다. 이는 훨씬 저렴한 작업이다.

## Hash Algorithms

![https://res.cloudinary.com/practicaldev/image/fetch/s--aHyRziwx--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/qr9mgguauoif18d5bzdi.jpg](https://res.cloudinary.com/practicaldev/image/fetch/s--aHyRziwx--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/qr9mgguauoif18d5bzdi.jpg)

이러한 알고리즘은 일반적으로 정보의 무결성을 확인하거나 암호(password)와 같은 일부 유형의 민감한 데이터를 저장하는데 사용된다. 왜 암호(password)일까? 동일한 해시 함수는 항상 동일한 입력에 대해 동일한 해시를 생성하기 때문이다. 따라서 시도한 암호를 해시하고 그 결과를 저장된 해시와 비교하여 인증 시도를 확인할 수 있다.

암호화 해시 함수에 대한 기억할만한 개념은 HMAC (Hash-based Message Authentication Code)이다. 이는 암호화 해시 함수와 비밀 암호화 키를 포함한 MAC(Message Authentication Code)의 타입이다. HMAC는 데이터 무결성과 메세지의 신뢰성을 동시에 확인하는데 사용된다.

### Common Hash Algorithms

**MD5 Message-Digest Algoritm (MD5)**

MD5는 1992년에 출시되었으며 MD2의 후속 제품인 MD4의 호속 제품으로 만들어 졌다. MD5는 MD4만큼 빠르지 않지만 이전 MDx 구현보다 더 안전한 것으로 간주된다. 사실임에 불구하고 MD5 사용은 어떤 용도로든 피해야한다. 이전 연구에서 입증했듯이 MD5는 암호화가 깨지고 더 이상의 사용에 적합하지 않다. 다만,여전히 비암호화 해싱 알고리즘으로는 사용할 수 있다.

**Secure Hash Algorithms (SHA)**

NIST에서 미국 연방 정보 처리 표준(FIPS)으로 게시한 암호화 해시 함수 제품군이며, 다음 알고리즘을 포함한다.

**SHA-1** - 1993년 개발되었으며 SSL/TLS, PGP, SSH, IPsec 및 S/MIME를 포함한 보안 애플리케이션 및 프로토콜에 널리 사용된다. SHA-1은 160 비트 다이제스트를 생성하며 블럭 크기는 512 비트이다. SHA-1이 여전히 널리 사용되고 있지만 2005년의 암호 분석가는 알고리즘에서 보안을 손상시키는 취약점을 발견했다. 이러한 취약점은 서로 다른 입력과 충돌을 신속하게 찾는 알고리즘의 형태로 나타났다. 즉, 두개의 서로 다른 입력이 동일한 다이제스트에 매핑된다.

**SHA-2** - 2001년에 발표되었으며 각각 32비트와 64비트 단어를 사용하는 SHA-256과 SHA-512라는 두개의 해시 함수로 구성된다. SHA-224, SHA-384, SHA-512/224 및 SHA-512/256으로 알려진 해시 함수의 추가적으로 잘린 버전이 있다. SHA-2는 24 또는 256 크기의 다이제스트를 생성하고 1024 비트 또는 512 비트를 포함하는 블럭 크기를 갖는다.

**SHA-3** - 2015년에 출시된 SHA 표준 제품군의 최신 구성원이다. SHA-3은 더 광범위한 암호화 기본 제품군 Keccak의 하위 집합이며 내부적으로 SHA-1 및 SHA-2와 크게 다르다. SHA-3이 더 나은 알고리즘이지만 NIST는 현재 SHA-2를 철회하거나 개정 된 보안 해시 표준에서 제거할 계획이 없다.

Beginner를 위한 글답게 아주 쉽고 간결하게 정리된 문서인 것 같다. 예전에 읽었지만 내용이 전혀 기억나지 않는 [자바와 암호화](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=48279309)라는 책도 시간날때 한번 더 읽어보도록 하자.