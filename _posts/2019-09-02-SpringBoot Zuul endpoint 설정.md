---
layout: post
title: SpringBoot Zuul endpoint 설정
categories: [MicroService]
comments: true

---
스프링부트 버전 2부터는 버전 1.X에서 쓰던 endPoint에서 변경이 되었습니다.

제가 공부하는 마이크로서비스 책은 스프링부트 1.X 버전대라 endPoint가 약간 다릅니다.

예를들어 Spring Zuul을 구성하게 되면 route 된 서비스들을 보기위해서는

스프링부트 1.X 버전은 HTTP를 통해서 /actuator/routes를 입력하면 route 목록들이 나오지만

스프링부트 2버전부터는 이 endpoint을 노출시켜줘야 합니다.

enabled 설정과 exposed 설정 두개를 해줘야 합니다.

지금은 어플리케이션의 Spring Security의 존재와 설정에 상관 없이

`/health`와 `/info`의 endpoints만 노출되어 있습니다.

다음을 이용해서 endpoints를 노출시킬 수 있습니다.

```
management.endpoints.web.exposure.include=*
```

다음을 이용해 명시적으로 `/shutdown`endpoint를 활성화 할 수 있습니다.

```
management.endpoint.shutdown.enabled=true
```

모든endpoints를 노출 시키지만 `env`endpoint는 제외시킵니다.

```
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=env
```

yml 방식

```
management:
  endpoints:
    web:
      exposure:
        include: "*"
```



![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/zuul/image1.png?raw=true)

이렇게 route 경로를 보지 못했지만. 모든 endpoints를 노출시키는 설정을 적용시키면

(저의 8762포트는 zuul을 사용하는 포트입니다.)



![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/zuul/image2.png?raw=true)

이렇게 routes들이 나옵니다. (저는 한개만 등록했습니다.)

출저 : [래퍼런스 문서](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#endpoints)