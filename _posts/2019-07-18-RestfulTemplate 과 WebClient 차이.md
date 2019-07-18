---
layout: post
title: RestfulTemplate 과 WebClient 차이
categories: [Spring]
comments: true

---
# RestfulTemplate vs WebClient

웹 어플리케이션에서 다른 서비스에게 HTTP로 호출하는 것은 공통 요구사항이 되었습니다.

스프링5 이전까지는 클라이언트에서 HTTP 접근을 위한 RestTemplate이 있었습니다. 

Spring MVC 프로젝트의 일부인 RestTemplate은 HTTP 서버와의 통신을 가능하게하고, RESTful 규칙을 이용합니다.

Spring Boot 앱에서는 HTTP 동작을 수행하기 위한 다른 대안으로 <u>Apache HttpClient</u> 라이브러리가 있습니다.

이런 대안들은 Java 서블릿 API 기반으로 하였으며, blocking 방식입니다.

스프링 프레임워크가 5버전이 되면서 이제는 HTTP 클라이언트 라이브러리를 통해 더 높은 수준의 

공통 API를 제공하는 새로운 <u>reactive WebClient</u>를 사용할 수 있습니다.

## RestfulTemplate과 WebClient 공통점

둘다 HTTP를 다룰때 사용 됩니다.

## RestfulTemplate과 WebClient 차이점 

|              | RestTemplate | WebClient |
| ------------ | ------------ | --------- |
| Non-blocking | 불가능       | 가능      |
| 비동기화     | 불가능       | 가능      |



> #### Non-blocking 이란 ?
>
> 시스템을 호출한 직후에 프로그램으로 제어가 다시 돌아와서 시스템 호출의 종료를 기다리지 않고 다음 처리로 넘어갈 수 있습니다.
>
> 장점으로는 호출한 시스템의 동작을 기다리지 않고 동시에 다른작업을 진행할 수 있어 작업의 속도가 빨라진다는 장점이 있습니다.



> #### Note
>
> 스프링 5.0부터 RestTemplte 보다 좋은 WebClient가 나왔기 때문에 RestTemplate은 deprecated 될 수 있습니다.

## 연습해보기

Restful 방식을 이용하여 통신해보겠습니다.



> #### Test 모듈

```java
package com.donghyoen.testmodule.resources;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

//tihs module port number is 8011.
@RestController
public class TestResource {

    @RequestMapping("/")
    public String testModule() throws InterruptedException {

        //delay
        Thread.sleep(3000);

        return "this is TestModule";
    }
}
```

이 Test 모듈은 포트번호를 8011을 사용하고, 호출이 되면 **"this is TestModule"** 을 반환합니다.

그리고 Non-blocking 방식을 증명하기 위해 메소드가 호출되면 3초동안 멈추게 했습니다.



> #### Resttemplate을 이용한 모듈

```java
package com.donghyoen.resttemplate.resources;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class RestTemplateResources {

    @RequestMapping("/")
    public String restTemplateTest() {
        RestTemplate restTemplate = new RestTemplate();
        //request and get responses
        String str = restTemplate.getForObject("http://localhost:8011/",String.class);

        //execute after 3 seconds later.
        System.out.println("1...");
        System.out.println("2...");
        System.out.println("3...");

        System.out.println(str);
        return str;
    }
}
```

> #### 결과

![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/restfultemplate/RestfulTemplate.gif?raw=true)

RestTemplate은 testModule의 함수의 호출이 끝날 때 까지 기다리는 모습을 볼 수 있습니다.



> #### WebClient를 이용한 모듈

```java
package com.donghyoen.WebClient.resources;

import ch.qos.logback.core.net.SyslogOutputStream;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

@RestController
public class WebClientResource {

    @RequestMapping
    public String webClientTest() {
        WebClient.Builder webClientBuilders = WebClient.builder();

        System.out.println("init retrieve");
        Mono<String> mono =  webClientBuilders.build()
                .get()
                .uri("http://localhost:8011/")
                .retrieve()
                .bodyToMono(String.class);


        System.out.println("1...");
        System.out.println("2...");
        System.out.println("3...");

        String str = mono.block();

        System.out.println(str);

        return str;
    }
}
```

> #### 결과

WebClient는 testModule이 끝나지 않아도 자신의 코드를 계속 실행하는 모습을 볼 수 있습니다.

![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/restfultemplate/WebClient.gif?raw=true)





[github 소스](https://github.com/DaeAkin/Spring-RestfulTemplate-WebClient-tutorial)

깊은 이해를 하기위해서는 RxJava를 보시는걸 추천드립니다.
