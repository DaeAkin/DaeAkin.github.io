## 시작하기 전

이 글은 JUnit Test 및 Mock에 대한 이해도가 있으신 분들에게만 추천 드립니다.

만약 없으시다면 Junit 및 Mock에 대한 글을 먼저 읽어보시는 걸 추천 드립니다.

## MockMvc

스프링의 컨트롤러를 테스트하기 위해서는 해당 URL 호출해서 파라미터 값을 같이 전송해주고,
컨트롤러에서 쓰는 라이브러리 주입 등, 준비할 과정이 많습니다.

그렇게 해서 나온 것이 MockMvc는 서버사이드인 Spring MVC 테스트를 도와주는 프레임워크입니다.

Spring에서 MVC패턴으로 작성한 프로그램의 컨트롤러를 시작해서 테스트를 해주는 프레임워크입니다.

## 필수라이브러리

```xml
<dependency>
    <groupId>com.jayway.jsonpath</groupId>
    <artifactId>json-path</artifactId>
    <version>2.2.0</version>
</dependency>
```

`.andExpect(Json)` 에서 Json값을 가지고 사용하기 떄문에 필요합니다. 

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-all</artifactId>
    <version>1.9.5</version>
    <scope>test</scope>
</dependency>
```

**MockMvc를** 사용하기 위해서 필요합니다.

## 선언하기

이 프레임워크를 사용하기 위해서는 **Mockmvc** 타입의 변수가 하나 필요합니다.

다음과 같이 선언해주세요.

```java
private MockMvc mockMvc;
```

## 초기화하기

이제 이변수를 **<u>초기화</u>**를 해줘야 사용할 수 있습니다.

스태틱 클래스인 **MockMvcBuilders** 를 이용하여 초기화를 해줄 수 있습니다.

**MockMvcBuilder** 클래스에는 두가지 메소드를 이용해 초기화를 해줄 수 있습니다.

- **static StandaloneMockMvcBuilder standaloneSetup(java.lang.Object... controllers)** : 
  @Controller 어노테이션이 붙은 클래스를 등록하고 Spring MVC 인프라를 설정하여 MockMvc를 만듭니다. 

  ```java
  mockMvc = MockMvcBuilders
          .standaloneSetup(memberController)
          .build();
  ```

- **static DefaultMockMvcBuilder webAppContextSetup(WebApplicationContext context) :** WebApplicationContext를 가지고 MockMvc를 초기화 합니다.

- ```java
   WebApplicationContext wac = ...;
  
   MockMvc mockMvc = webAppContextSetup(wac).build();
  ```

```

```

이 중에 `standaloneSetup(java.lang.Object... controllers)` 를 좀더 살펴보겠습니다. 

이 메소드는 리턴값이 **<u>StandaloneMockMvcBuilder</u>** 이며 빌더패턴을 이용하고 있습니다.
또한 이 클래스를 이용하여 컨트롤러에 <u>인터셉터</u>를 추가하여 테스트하거나, <u>필터추가</u>, <u>어드바이스</u> 추가등
조금 더 세밀한 컨트롤러테스트를 가능하게 해줍니다.

그 중에서 몇개의 메소드만 살펴보겠습니다.

### StandaloneSetup 메소드

- **StandaloneMockMvcBuilder addInterceptor(HandlerInterceptor... interceptors)** : 인터셉터를 추가하는 메소드입니다.
- **StandaloneMockMvcBuilder setViewResolvers(ViewResolver... resolvers)** : 어떤 ViewResolver를 사용할건지 셋팅합니다.

여기서 중요한건 **StandaloneMockMvcBuilder** 클래스는 **<u>AbstractMockMvcBuilder를</u>** extends 했기 때문에 
**AbstractMockMvcBuilder** 클래스에 있는 메소드를 사용할 수 있는데, 여기에 <u>FIlter</u>를 추가하는 메소드가 있습니다.

```java
@Before
public void setUp() {
    mockMvc = MockMvcBuilders
            .standaloneSetup(memberController)
             .addFilters(new CORSFilter())
            .build();
}
```

[StandaloneMockMvcBuilder](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/setup/StandaloneMockMvcBuilder.html)

[AbstractMockMvcBuilder](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/setup/AbstractMockMvcBuilder.html)



## MockMvc 이용하기

### MockMvc의 메소드

- **DispatcherServlet getDispatcherServlet()** : MockMvc와 함께 초기화된 **DispatcherServlet**을 리턴합니다. 핸들링 요청이 런타임 시 DispatcherServlet에게 위임하기 때문에 커스텀 핸들링 요청을 사용할 때는 Dispatcher를 함께 주입해줘야 합니다.
- **ResultActions perform(RequestBuilder requestBuilder)** : 요청을 수행하고 액션들을 체인으로 연결해서 예상 값이나 결과값을 리턴받습니다.



perfomr 메소드는 인자로 **<u>RequestBuilder 인터페이스</u>**를 구현한 것들을 넣어줘야합니다.
`RequestBuilders`의 스태틱 팩토리 메소드인 `MockMvcRequestBilders`를 사용하면 됩니다.

### MockMvcRequestBuilders의 메소드

자주 사용되는 메소드 몇개만 살펴보겠습니다.

- **MockHttpServletRequestBuilder get(java.lang.String urlTemplate)** : Get 방식의 request를 만들어 요청을 해줍니다.
- **MockHttpServletRequestBuilder post(java.lang.String urlTemplate)** : Post 방식의 request를 만들어 요청을 해줍니다.

```java
@Test
public void memberLogin() throws Exception {
        // /member/login 에게 POST 방식으로 요청합니다.
        mockMvc.perform(post("/member/login"))		
} 
```

get() 과 post()는 **<u>MockHttpServletRequestBuilder</u>** 를 리턴하기 때문에 
**MockHttpServletRequestBuilder** 의 함수들을 체인형식으로 이어 붙일 수 있습니다. 

위에 소스에서 post 방식으로 해당 url로 요청하는 동시에 <u>paramter</u>도 같이 넘겨 주겠습니다.

바로 아래에 있는 메소드를 사용하면 됩니다.



- **MockHttpServletRequestBuilder param(java.lang,String name, java.lang.String... values)** : MockHttpServletRequest에 request paramter를 추가합니다.

  ```java
  @Test
  public void memberLogin() throws Exception {
          // /member/login 에게 POST 방식으로 요청합니다.
          mockMvc.perform(post("/member/login")		
                      .param("id", "testid")
  					.param("pw", "1234"))
  } 
  ```

  

- **MockHttpServletRequestBuilder session(MockHttpSession session)** : 재사용이 가능한 HTTP session을 설정합니다.

- **MockHttpServletRequestBuilder accept(java.lang.String... mediaTypes**) : 'Accept' 헤더를 설정해줍니다.

이 함수 외에도 많은 메소드들이 많아 API 문서를 보고 사용하면 됩니다.

[MockHttpServletRequestBuilder](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/request/MockHttpServletRequestBuilder.html)



`perform()` 함수를 다시 보겠습니다. `ResultActions perform(RequestBuilder requestBuilder)`
`perform()` 함수는 리턴 값이 **<u>ResultActions</u>** `Interface` 입니다. 

### ResultActions의 메소드

**ResultActions** 인터페이스는 3개의 메소드로 구성되어 있습니다.

- **ResultActions andDo(ResultHandler handler)** : 어떤 동작을 수행할때 사용합니다.
- **ResultActions andExpect(ResultMatcher matcher)** : 결과값을 예상합니다.
- **MvcResult andReturn()** : 요청을 실행한 결과를 MvcReesult 타입으로 리턴합니다.



**ResultActions**의 함수 `andDo()` 와 `andExpect()` 는 ResultActions을 리턴하므로 체인형식으로 이어 붙일 수 있습니다.

```java
@Test
public void memberLogin() throws Exception {
	mockMvc.perform(post("/member/login")
					.param("id", "testid")
					.param("pw", "1234"))
			.andDo(null)
			.andExpect(null);
}		
```

지금은 보여드리기 위해서 파라미터 값으로 null 값을 줬습니다. 

`andDo()` 함수는 파라미터를 **<u>ResultHandler</u>** `Interface` 를 구현한 클래스를 받습니다.

**MockMvcResultHandlers** 클래스를 사용하면 됩니다.

- **ResultHandler log()** 

- **ResultHandler print()**

  ```java
  mockMvc.perform(post("/member/login")
  				.param("id", "testid")
  				.param("pw", "1234"))
  				.andDo(print())
  ```

  ##### Output

  ```
  MockHttpServletRequest:
        HTTP Method = POST
        Request URI = /member/login
         Parameters = {id=[testid], pw=[1234]}
            Headers = {}
  
  Handler:
               Type = com.www.homedoc.controller.MemberController
             Method = public java.util.Map<java.lang.String, java.lang.Object> com.www.homedoc.controller.MemberController.memberLogin(java.util.Map<java.lang.String, java.lang.Object>,javax.servlet.http.HttpSession)
  
  Async:
      Async started = false
       Async result = null
  
  Resolved Exception:
               Type = null
  
  ModelAndView:
          View name = null
               View = null
              Model = null
  
  FlashMap:
         Attributes = null
  
  MockHttpServletResponse:
             Status = 200
      Error message = null
            Headers = {Content-Type=[application/json;charset=UTF-8]}
       Content type = application/json;charset=UTF-8
               Body = {"code":"OK"}
      Forwarded URL = null
     Redirected URL = null
            Cookies = []
  ```

  

- **ResultHandler print(java.io.OutputStream stream)**

- **ResultHandler print(java.io.Writer writer)**

`andExpect()` 함수는 파라미터를 **<u>ResultMatcher</u>**  `Interface` 를 구현한 클래스를 받습니다. 

**MockMvcResultMatchers** 클래스를 사용하면 됩니다.
메소드들이 많은데 그중 몇개만 살펴 보겠습니다. 

- **StatusResultMatchers status()** : 응답 상태를 검증해줍니다 (예를들어 응답코드가 정상인지(200))

  ```java
  mockMvc.perform(post("/member/login")
  					.param("id", "testid")
  					.param("pw", "1234"))
  					.andDo(print())
  					//리다이렉트 응답코드는 302
  					.andExpect(status().is(302))
  ```

  

- **ModelResultMatchers model()** : 모델에 접근해서 모델객체에 대한 검증을 합니다. 

- **ResultMatcher redirectedUrl(java.lang.String expectedUrl)** : 해당 url로 정상적으로 리다이렉트 되었는지 검증합니다.

- **\<T> ResultMatcher jsonPath(java.lang.String expression, Matcher\<T> matcher)** : JsonPath 표현식을 이용하여 response body에 접근하여 검증하고 찾은 JSON 값을 검증하는 방법으로  Hamcrest matcher를 이용합니다.

  ```java
  .andExpect(jsonPath("$.code", is("OK")))
  ```

  `$.Json의 키` `is("Json의 값 ")` 으로 사용하시면 됩니다.

이제 `andReturn()` 함수를 이용해서 MvcResult 타입으로 리턴해주겠습니다.

```java
MvcResult result =
            mockMvc.perform(post("/member/login")
                .param("id", "testid")
                .param("pw", "1234"))
                .andDo(print())
                //리다이렉트 응답코드는 302
                .andExpect(status().is(302))
                .andReturn();
```

### MvcResult 메소드

MvcResult의 메소드를 몇개만 다뤄보겠습니다.

- **HandlerInterceptor[] getInterceptors()** : 핸들러에 있는 인터셉터들을 리턴합니다. 

- **ModelAndView getModelAndView()** : 핸들러에 의해 준비된 ModelAndView를 리턴합니다.

  ```java
  ModelAndView mav = result.getModelAndView();
  ```

- **MockHttpServletRequest getRequest()** : perform함수를 실행하고 난 request 객체를 리턴합니다. 

  ```java
  MockHttpServletRequest request = result.getRequest();
  ```

- **MockHttpServletResponse getResponse()** : 응답 결과를 리턴합니다.

MvcResult의 함수를 호출해서 원하는 객체를 받은 후 그 객체를 이용해서 효과적인 검증을 할 수 있습니다.

