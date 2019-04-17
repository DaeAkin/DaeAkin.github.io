---
layout: post
title: Interceptor
categories: [Spring]
comments: true
---

# Spring Interceptor



## 소개

이번 포스팅은 Spring MVC *HandlerInterceptor* 를 이해하고 올바르게 사용하는 법을 알아 볼 것이다.



## Spring MVC Handler

인터셉터를 이해하기 위해서는 *HandlerMapping* 을 잘 알아야한다.  *HandlerMapping* 는 URL에 관련된 메소드인데,
*DispatchServlet* 이 요청을 처리할 때 호출 할 수 있는 메소드이다.
그리고 *DispatcherServlet* 는 실제 메소드를 호출하기 위해서 *HandlerAdapter* 를 사용한다.
​	이제 전반적인 context 설정파일을 이해해보자. 요청을 처리하기 전이나,후 처리를 완료하기전 (view단이 렌더링 될때) 어떠한 동작을 수행하기 위해 *HandlerInterceptor* 를 이용하게 된다.
인터셉터는 중복되는 관심 또는 반복되는 핸들러 코드 (예를 들어 로깅작업)을 처리하기위해 사용된다.



## Maven Dependency

인터셉터를 사용하기 위해서는 pom.xml에 라이브러리를 추가해줘야한다

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.0.6.RELEASE</version>
</dependency>
```



## Spring Handler Interceptor

프레임워크에서 *HandlerMapping* 과 같이쓰는 인터셉터들은 반드시 *HandlerInterceptor* 인터페이스를 구현해야한다.

*HandlerInterceptor* 는 3가지 메인 메소드가 있다 :

- prehandle() : 실제 핸들러가 실행되기전에 호출된다, 그러나 view는 아직 생성되지 않음.
- postHandle() : 핸들러가 실행되고 나서 호출된다.
- afterCompletion() : 요청을 끝내고 , view가 생성될때 호출된다.

> *HandlerInterceptor* 와 *HandlerInterceptorAdapter의* 주요 차이점은 전자는 위에 언급한 3가지 메소드를 override를 해야하고, 반면에 후자는 필요한 메소드만 구현하면 된다는 차이점이 있다.



아래는 *preHandle()*을 간단하게 구현한 모습이다

```java
@Override
public boolean preHandle(
  HttpServletRequest request,
  HttpServletResponse response, 
  Object handler) throws Exception {
   // 코드입력
    return true;
}
```

return 값은 만약 요청이 핸들러에 의해 계속 진행댄다면 **true** 값을 주고 그게 아니라면 **false를** 준다.



다음은 *postHandle()*을 구현한 것이다.

```java
@Override
public void postHandle(
  HttpServletRequest request, 
  HttpServletResponse response,
  Object handler, 
  ModelAndView modelAndView) throws Exception {
    // 코드 입력
}
```

이 메소드는 요청이 *HandlerAdapter*에 의해 처리된 후 즉시 호출된다. 그러나, view가 생성되기 전이다.



마지막으로 우리가 구현해야하는 메소드는 *afterCompletion()*이다.

```java
@Override
public void afterCompletion(
  HttpServletRequest request, 
  HttpServletResponse response,
  Object handler, Exception ex) {
    // 코드 입력
}
```

view가 성공적으로 생성되면, 요청과 관련된 추가적인 작업을 이용할 수 있다.

(다시정리)마지막으로 숙지해야할 사항은 *HandlerInterceptor*는 *@Controller* 어노테이션이 붙은 어떠한 클래스에 인터셉터를 등록할 수 있는 책임을 가진 *DefaultAnnotationHandlerMapping* 빈에 등록되어있다. 거기에다가 웹 어플리케이션에 인터셉터의 수를 특정할 수 있다.





## 커스텀 Logger 인터셉터

웹 어플리케이션에 로깅 작업을 하는 예제이다. 첫번째로 *HandlerInterceptorAdapter*를 상속해야한다.

```java
public class LoggerInterceptor extends HandlerInterceptorAdapter {
    ...
}
```

인터셉터 안에 logging 을 활성화 시켜주자.

```java
private static Logger log = LoggerFactory.getLogger(LoggerInterceptor.class);
```

이 작업은 Log4J는 로그를 표시할뿐만 아니라, 현재 어떤 클래스가 특정한 출력에 정보를 기록하고 있는지 나타낸다.
다음으로, 인터셉터 구현에 집중해보자.



- Method *preHandle()*

  이 메소드는 요청을 처리하기전에 호출된다. **return true**로 설정하면 프레임워크에게 추가요청을 허용해준다. (아니면 다음 인터셉터에게) 
  만약 **false** 설정하면 요청이 처리됬고, 더 이상 추가 요청을 하지 않는다라고 가정하게된다.

  또한 요청에 관한 파라미터들(어디서 요청이 들어왔는지) 의 정보를 알아낼 수 있다.

  ```java
  @Override
  public boolean preHandle(
    HttpServletRequest request,
    HttpServletResponse response, 
    Object handler) throws Exception {
       
      log.info("[preHandle][" + request + "]" + "[" + request.getMethod()
        + "]" + request.getRequestURI() + getParameters(request));
       
      return true;
  }
  ```

  볼 수 있듯이, 요청에 대한 기초적인 정보들을 로깅한다.
  만약 비밀번호 처럼 민감한 사항은 로그 정보에 안뜨게 할수 있다.
  가장 간단한 방법은 데이터들을 * 로 대체하는 것이다.

  ```java
  private String getParameters(HttpServletRequest request) {
      StringBuffer posted = new StringBuffer();
      Enumeration<?> e = request.getParameterNames();
      if (e != null) {
          posted.append("?");
      }
      while (e.hasMoreElements()) {
          if (posted.length() > 1) {
              posted.append("&");
          }
          String curr = (String) e.nextElement();
          posted.append(curr + "=");
          if (curr.contains("password") 
            || curr.contains("pass")
            || curr.contains("pwd")) {
              posted.append("*****");
          } else {
              posted.append(request.getParameter(curr));
          }
      }
      String ip = request.getHeader("X-FORWARDED-FOR");
      String ipAddr = (ip == null) ? getRemoteAddr(request) : ip;
      if (ipAddr!=null && !ipAddr.equals("")) {
          posted.append("&_psip=" + ipAddr); 
      }
      return posted.toString();
  }
  ```



  마지막으로 HTTP 요청한 사람의 IP주소를 가져올 수도 있다.

  ```java
  private String getRemoteAddr(HttpServletRequest request) {
      String ipFromHeader = request.getHeader("X-FORWARDED-FOR");
      if (ipFromHeader != null && ipFromHeader.length() > 0) {
          log.debug("ip from proxy - X-FORWARDED-FOR : " + ipFromHeader);
          return ipFromHeader;
      }
      return request.getRemoteAddr();
  }
  ```



- **Method *PostHandle()***

  이 메소드는 *DispatcherServlet*이 아직 view를 렌더를 하지 않았지만, *HandlerAdapter*가 핸들러를 호출할때 실행된다.

  ModelAndView 나 클라이언트의 요청을 처리하기 위해 걸리는 핸들러 시간을 확인하기 위해 추가적인 속성을 사용할 수 있다.

  *DispatchServlet*이 렌더링 되기 전에 request를 그냥 log로 출력해 볼 것이다.

  ```java
  @Override
  public void postHandle(
    HttpServletRequest request, 
    HttpServletResponse response,
    Object handler, 
    ModelAndView modelAndView) throws Exception {
       
      log.info("[postHandle][" + request + "]");
  }
  ```

- **Method *afterCompletion()***

  ​	만약에 어떠한 exception이 생기면 요청이 끝나고 뷰가 렌더링 될 때, request , response 데이타에 대한 exception 정보를 얻을 수 있다.

  ```java
  @Override
  public void afterCompletion(
    HttpServletRequest request, HttpServletResponse response,Object handler, Exception ex) 
    throws Exception {
      if (ex != null){
          ex.printStackTrace();
      }
      log.info("[afterCompletion][" + request + "][exception: " + ex + "]");
  }
  ```



- **설정**

  인터셉터를 스프링 설정에 적용하려면 ,  *WebMvcConfigurer*를 구현한 *WebConfig* 클래스 안에 있는 *addInterceptors()* 메소드를 오버라이드 해야만 한다. 

  ```java
  @Override
  public void addInterceptors(InterceptorRegistry registry) {
      registry.addInterceptor(new LoggerInterceptor());
  }
  ```



  아니면 XML 스프링 설정 파일을 수정해도 된다.

  ```xml
  <mvc:interceptors>
      <bean id="loggerInterceptor" class="org.baeldung.web.interceptor.LoggerInterceptor"/>
  </mvc:interceptors>
  ```

  설정을 활성화 하면 인터셉터는 활성화 되고, 애플리케이션의 모든 요청들은 적절히 로깅 될것이다.

  만약 여러개의 인터셉터들을 설정한다면 *PreHandle()* 메소드는 설정 순서에따라 실행돌 것이다. 반면에
  *postHandle()* 과 *afterCompletion()* 메소드는 역순으로 호출된다.



  web.xml

  ```xml
    <servlet>
      <servlet-name>appServlet</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
      </init-param>
      <load-on-startup>1</load-on-startup>
    </servlet>
  ```

  web.xml보면 컨텍스트설정파일위치를 *DispatcherServlet* 가지고 있다. 
  *DispatcherServlet이* 가지고 있는 설정파일위치에 bean을 작성해줘야 정상적으로 작동이된다.



  [![img](https://1.bp.blogspot.com/-LtOZGN6GCf4/W_aHMNFxBAI/AAAAAAAABEk/pLw8X0ZAC8AYQMHIadsLMMXjmBIviR_CACLcBGAs/s640/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%2B2018-11-22%2B%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%2B7.34.31.png)](https://1.bp.blogspot.com/-LtOZGN6GCf4/W_aHMNFxBAI/AAAAAAAABEk/pLw8X0ZAC8AYQMHIadsLMMXjmBIviR_CACLcBGAs/s1600/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%2B2018-11-22%2B%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%2B7.34.31.png)x	
