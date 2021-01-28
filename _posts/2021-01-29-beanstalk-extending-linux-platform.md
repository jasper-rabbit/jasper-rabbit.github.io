---
title: AWS Elastic Beanstalk에서 nginx 설정
author: Jasper Ra66it
date: 2021-01-29 00:25:00 +0900
categories: [Blogging,AWS]
tags: [AWS, Beanstalk, .ebextensions, nginx, client_max_body_size]
pin: false
---
오랜만에 신규 프로젝트를 만들어 [AWS Beanstalk](https://aws.amazon.com/ko/elasticbeanstalk/)를 이용하여 배포를 하게 되었다. 조금 대용량 데이터를 업로드 해야하는 기능이라 nginx의 설정을 변경해서 업로드파일 용량제한을 늘려줘야 했다. 운영중인 다른 서비스들에서 이미 [.ebextendsions](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions.html)를 이용하여 서버의 다양한 환경설정등을 변경하고 있었기에 특별한 문제는 아니었다. 대략 아래와 같은 내용의 파일을 `.ebextensions` 디렉토리 아래(`.ebextensions/nginx/conf.d/`) 생성하고 배포만 하면 끝! (인줄 알았는데...)

```
files:
  "/etc/nginx/conf.d/01_proxy.conf":
    mode: "000755"
    owner: root
    group: root
    content: |
      client_max_body_size 50M;
```

그런데 아무리해도 반영이 되지 않는것이 었다. 한참을 삽질을 하다가, 문득 마지막으로 Beanstalk를 신규로 생성한지가 꽤 오래전이었다는 생각이 들었다(대략 몇년?). 현재 운영중인 서버들이 최소 1~2년이상은 지났으니... Beanstalk 콘솔에 `Retired`라는 빨간 딱지들이 붙어있기도 하다. (참고로 Beanstalk 플렛폼 업글은 클릭 몇번으로 간단히 되긴하지만, 톰켓 버전 문제가 걸려서 진행을 못하고 있다.)

역시나 원인은 Beanstalk에서 사용하는 Amazon Linux 버전이 변경되었는데(정확히 언제부터인지는 찾아보지 않았다), 바뀐 Amazon Linux 2에서 설정과 관련한 변경사항이 생겼기 때문이었다. [Migrating your Elastic Beanstalk Linux application to Amazon Linux 2](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.migration-al.html) 이 문서를 참고. Beanstalk Linux의 추가적인 설정은 [Extending Elastic Beanstalk Linux platforms](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/platforms-linux-extend.html) 이 문서를 읽어 보도록하자. 

> Proxy configuration files provided in the `.ebextensions/nginx` directory should move to the `.platform/nginx` platform hooks directory. For details, expand the Reverse Proxy Configuration section in Extending Elastic Beanstalk Linux platforms.

말그대로 nginx의 디렉토리 위치가 `.platform` 아래로 변경되었고, nginx의 `client_max_body_size` 설정 변경을 위해서는 `.platform/nginx/conf.d/` 위치에 `proxy.conf` 파일을 생성하여 내용에는 아래와 같이 설정 값만 입력해주면 된다.

```
client_max_body_size 100M;
```
