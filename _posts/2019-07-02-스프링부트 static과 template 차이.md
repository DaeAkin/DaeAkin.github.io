---
layout: post
title: SpringBoot static폴더와 template폴더의 차이
categories: [Spring]
comments: true

---
# SpringBoot에서 static폴더와 template폴더의 차이 

SpringBoot를 처음 접하셨다면, webapp 디렉토리가 존재하지 않고, static과 templates 폴더가 존재하는걸 볼 수 있습니다.

```
demo
  |+-src/main
  |    +-java
  |    +-resources
  |         static
  |         templates
  |         application.properties
  |-pom.xml
```

> #### templates

스프링이 계속 버전이 올라가면서 view 엔진이 JSP 대신 thymeleaf로 바뀌었습니다. 

templates폴더는 thymeleaf의 파일들을 두는 곳입니다. 

> #### static

static 폴더는 content들을 두는 곳입니다. 보통 css나 js를 두곤합니다.`/static` 을 이용해서 웹에서 호출할 수도 있습니다.

Spring MVC에 있는 `ResourceHttpRequestHandler` 를 이용하기 때문에 자신만의 `WebMvcConfigurer` 를 만들어

`addResourceHandlers` 를 오버라이딩 해서 동작을 수정할수도 있습니다.

기본적으로 resources 폴더는 `/**` 로 맵핑되어 있지만, `spring.mvc.static-path.pattern` 속성을 이용하면 변경이 가능합니다.

```properties
spring.mvc.static-path-pattern=/resources/**
```

> 스프링레거시에서는 `src/main/webapp 폴더를 이용했습니다.
>
> 그러나 부트에서는 jar로 패키징이 되기때문에 스프링레거시처럼 webapp 폴더를 이용하고 싶다면 
>
> 패키징 방법을 war로 변경해야합니다.
>
> pom.xml에서 변경하면 됩니다.

```xml
<packaging>war</packaging>
```

