---
layout: post
title: Spring Validator
categories: [Spring]
comments: true
image:
  feature: /img/validation.jpg
---
# 스프링에서 유효성 검사하기 (Spring Validator) 

## 개요

스프링 프레임워크 4.0은 빈 검증기 1.0(JSR-303) 와 1.1(JSR-349)를 지원하며
Spring의 `Validator` 인터페이스에 적용할 수 있습니다.

어플리케이션은 빈 검증기를 전체에 사용할것인지, 아니면 부분적으로만 사용할것인지 선택할 수 있습니다. 
검증 로직의 어노테이션 사용 없이 `DataBinder` 를 이용하여 `Validator` 를 추가적으로 등록할 수 있습니다.

`DataBinder`은 애플리케이션의 도메인모델에 동적으로 바인딩 할 때 유용합니다. 
스프링은 데이터바인딩을 하기 위해 `DataBinder` 을 제공해줍니다.
`Validator` 와 `DataBinder` 는 MVC 프레임워크에서 주로사용되는 **유효성 검사 패키지** 입니다.

스프링 MVC에서 `Validator` 인터페이스는 HTML 폼에 있는 모든필드를 검증하기 위해서 사용됩니다.
controller 클래스안에 `@InitBinder` 어노테이션을 이용해서 설정할 수 있습니다.

`@InitBinder` 어노테이션은 메소드레벨의 어노테이션이며, `WebDataBinder` 를 초기화하기 위해 사용합니다.

`WebDataBinder` 클래스는 웹에서 온 request 파라미터들을 model 오브젝트에 바인드를 해줍니다.

자신이 만든 커스텀 `Validator` 를 컨트롤러에 등록하고 싶다면 `WebDataBinder.addValidators()` 메소드를 통해 
등록이 가능합니다.

인자나, model 오브젝트에 `@Validator` 어노테이션을 작성하면, 핸들러 메소드가 이를 알고 커스텀 validator를 등록해줍니다.

만약 검증을 위반하는 사항이 발견된다면, 자동적으로 에러가 `BindingResult` 에게 가게 됩니다.



## 들어가며

검증기를 사용하는 방법은 두가지가 있습니다.

첫째로 모델 오브젝트에 주석을 사용해서 작성하는 방법과,

두번째로 `Validator` 인터페이스를 구현하는 방법입니다.



## 주석을 이용해서 검증하기.

먼저 2개의 필드를 가진 MemberDto 클래스를 만들겠습니다. 

> ### MemberDto

```java
package com.kei890.validationtutorial.dto;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import org.springframework.stereotype.Component;

@Component
public class MemberDto {
	
	int no;
	
	@NotNull
	@Size(min=1, max=30)
	String id;
	
	@NotNull
	@Size(min=8, max=30 , message="비밀번호 오류!")
	String pw;
	
	public MemberDto() {
	}

	public int getNo() {
		return no;
	}

	public void setNo(int no) {
		this.no = no;
	}

	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}

	public String getPw() {
		return pw;
	}

	public void setPw(String pw) {
		this.pw = pw;
	}

	@Override
	public String toString() {
		return "id : " + getId() + "\n pw :" + getPw();
	}

}

```

id의 길이는 1 부터 30까지 그리고 pw의 길이는 8 부터 30까지 주었고,
오류가 발생시 `비밀번호 오류!` 라는 메세지를 뿌리도록 작성했습니다.

어노테이션에 대한 설명은 다음과 같습니다.

- **@NotNull** - 값이 null인지 체크해줍니다. 그러나 빈배열은 체크를 해주지 못합니다.
- **@Pattern** - 값을 주어진 표현식으로만 제한합니다. ex) 값을 영어만 받고싶은 경우` @Pattern(regexp=”[^0-9]*”)`
- **@Past** - 현재보다 과거날짜만 입력가능합니다. ex) 사용자의 생일을 입력할 때
- **@Min** - 지정해준 정수값과 같거나 커야합니다.
- **@Max** - 지정해준 정수값과 같거나 작아야 합니다.
- **@NotBlank** - 문자열이 null인지 체크하고 공백을 제외한 문자열이 0보다 큰지 체크합니다. 이 어노테이션은 JSR 303이 아닙니다.
- **@Email** - 유효한 이메일 형식인지 확인합니다. 이 어노테이션 또한 JSR 303이 아닙니다.



> ### MemberController

```java
package com.kei890.validationtutorial.controller;

import javax.validation.Valid;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;

import com.kei890.validationtutorial.dto.MemberDto;

@Controller
public class MemberController {
	
    @RequestMapping("/")
    public String moveTestJsp(Model model) {
    	//빈 객체 넘겨줘야함.
    	model.addAttribute("memberDto", new MemberDto());
    	return "insert";
    }
    
    @RequestMapping("/insert")
    public String memberInsert(@Valid @ModelAttribute MemberDto memberDto,
    		BindingResult result) {
     
    	System.out.println("---- MemberController::memberInsert() ----");
    	    	
    	if(result.hasErrors()) {
    		//form에 에러가 있으면
    		return "insert";
    	}
    	//에러가 없으면
    	return "";
    	
    }
    
}

```



> ### insert.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<html>
    <head>
        <title>Sign Up</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css">
        <style type="text/css">
            .errormsg {
                color: red;
            }
        </style>
    </head>
    <body>
        <div class="container">
            <h2 align="center" class="text-primary">Spring MVC Form Validation Example</h2>
            <hr />
            <div> </div>
 
            <form:form action="insert" method="POST" modelAttribute="memberDto">
                 <div class="form-group">
                    <label>ID :</label><form:input path="id" size="30" cssClass="form-control" placeholder="Enter ID" />             
                    <small><form:errors path="id" cssClass="errormsg" /></small>
                 </div>
                 <div class="form-group">
                    <label>Password:</label><form:password path="pw" size="30" cssClass="form-control" placeholder="Enter password" />
                    <small><form:errors path="pw" cssClass="errormsg" /></small>
                 </div>
                 <div class="form-group">
                    <button type="submit" class="btn btn-primary">검증</button>
                 </div>
            </form:form>
        </div>
    </body>
</html>
```



> #### 비밀번호의 길이가 안맞을 경우 

[![img](https://3.bp.blogspot.com/-ziB_5KO_Ifc/XLHYso5wv1I/AAAAAAAABKI/D3bOFoRCxGo-euv6aHKrlGlR2xWGQE2AgCLcBGAs/s1600/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%2B2019-04-13%2B%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%2B9.37.56.png)](https://3.bp.blogspot.com/-ziB_5KO_Ifc/XLHYso5wv1I/AAAAAAAABKI/D3bOFoRCxGo-euv6aHKrlGlR2xWGQE2AgCLcBGAs/s1600/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%2B2019-04-13%2B%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%2B9.37.56.png)



> #### 아이디를 빈값을 넣을경우에도 메세지 설정을 하지 않아도 default 메세지가 출력된다.

[![img](https://1.bp.blogspot.com/-b0cS1VaY-Tc/XLHYse8jFyI/AAAAAAAABKE/417tD1jWY6s12ZPhiDqUQloFe4OvCynzACLcBGAs/s1600/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%2B2019-04-13%2B%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%2B9.38.08.png)](https://1.bp.blogspot.com/-b0cS1VaY-Tc/XLHYse8jFyI/AAAAAAAABKE/417tD1jWY6s12ZPhiDqUQloFe4OvCynzACLcBGAs/s1600/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%2B2019-04-13%2B%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%2B9.38.08.png)

[어노테이션을 사용하는 방법 깃허브 소스](https://github.com/DaeAkin/ValidationTutorial/tree/withoutAnnotation)

## Validator 인터페이스 구현해서 만들기

### 스프링 설정 파일 

root-context.xml에 다음의 빈을 추가해주세요.

```xml
<bean id="messageSource"
    class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
    <property name="basename" value="/WEB-INF/message" />
</bean>
```

`ReloadableResourceBundleMessageSource` 빈은 지정된 basename과 스프링의 `ApplicationContext` 의 리소스 로딩을 이용하여 리소스에 접근이 가능한 `MessageSource` 의 구현클래스 입니다. 

JDK 기반을 둔 `ResourceBundleMessageSource` 와 반대로, 이 클래스는 스프링의 리소스 핸들러부터 `PropertiesPersister` 방법을 통해 메세지를 위한 사용자 정의 데이터 `Properties` 인스턴스를 사용할 수 있습니다.

전형적인 웹 어플리케이션에서 메시지 파일들은 WEB-INF 밑에 두어 사용하는데, 위에와 같이 `/WEB-INF/message` 로 설정하게 되면 `WEB-INF/message.properties` `WEB-INF/message.xml` `WEB-INF/message_en.xml` 등 모두 인식을 할 수 있습니다. 

`basename` 은 리소스 번들의 위치를 작성해야하며, **반드시** 작성해야하는 프로퍼티입니다.

### 메세지 리소스 파일 

이제 유효성 검사에러를 위한 메세지 파일을 하나 만들어 보겠습니다. 

`/WEB-INF/` 밑에 `message.properties` 를 생성하겠습니다.

> ### message.properties

```properties
 ## memberform.id ##
member.id.empty = property : Insert ID!
 ## memberform.pw ##
member.pw.empty = property : Insert PW!
```



> ### MemberDto

```java
package com.kei890.validationtutorial.dto;
import org.springframework.stereotype.Component;

@Component
public class MemberDto {
	
	int no;
	String id;
	String pw;
	
	public MemberDto() {
	}

	public int getNo() {
		return no;
	}

	public void setNo(int no) {
		this.no = no;
	}

	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}

	public String getPw() {
		return pw;
	}

	public void setPw(String pw) {
		this.pw = pw;
	}

	@Override
	public String toString() {
		return "id : " + getId() + "\n pw :" + getPw();
	}

}
```

아까 변수에 적어줬었던 어노테이션만 제거했습니다.



### Validator 인터페이스 구현하기

Validator 인터페이스를 구현하기 위해서 MemberValidator 클래스를 하나 생성해주세요

> ### MemberValidator

```java
package com.kei890.validationtutorial.dto;
import java.util.regex.Pattern;
import org.springframework.stereotype.Component;
import org.springframework.validation.Errors;
import org.springframework.validation.ValidationUtils;
import org.springframework.validation.Validator;

@Component
public class MemberValidator implements Validator {

	@Override
	public boolean supports(Class<?> clazz) {
		return MemberDto.class.equals(clazz);
	}

	@Override
	public void validate(Object obj, Errors error) {
		ValidationUtils.rejectIfEmpty(error, "id", "member.id.empty","아이디를 입력해주세요.");
		ValidationUtils.rejectIfEmpty(error, "pw", "member.pw.empty","비밀번호를 입력해주세요");
       
        MemberDto memberDto = (MemberDto) obj;
		
        	//만약 이메일의 값을 검증하고싶다면 Pattern 클래스를 이용하자!
          Pattern pattern = Pattern.compile("^[A-Z0-9._%+-]+@[A-Z0-9.-]+\\.[A-Z]{2,6}$",
                Pattern.CASE_INSENSITIVE);
          if (!(pattern.matcher(memberDto.getId()).matches())) {
             err.rejectValue("id", "member.email.invalid");
          }
	}
}
```

Validator를 구현해준 클래스는 빈으로 등록해줘야 하기 때문에 자동으로 빈으로 등록되기 위해서 
`@Component` 어노테이션을 붙여줬습니다.

`supports(Class<?> clazz)` 에서 구현해줘야 할 것은, **인자로 넘어온 클래스가 이 검증 클래스를 지원하는가?** 입니다.
보통 저렇게 쓰거나 , `MemberDto.class.isAssignableFrom(clazz);` 를 많이 사용합니다. 

`validate(Object obj, Erros error)`에서 구현해줘야 할 것은 어떤걸 검증할 것인가? 를 작성해주면 됩니다.
보통 ValidationUtils 스태틱 클래스를 이용해서 작성합니다.

`rejectIfEmpty` 는 오버로딩 메소드라서 원하는 메소드를 골라서 사용하면 됩니다. 

예제에서 사용했던 메소드는 인자가 4개짜리인데요, 
첫번째 인자에는 그냥 error를 담아주고, 2번째 인자에는 필드이름을 적어줍니다.
3번째 인자에는 message.properties에 적었던 이름을 적어주면 됩니다.
4번째 인자에는 만약 3번째 인자의 값을 못찾을 경우 디폴트로 나타내줄 메세지를 적어주시면 됩니다.

또한 넘어온 도메인의 값을 검증하고 싶다면 Pattern 클래스를 이용해서 해당 값을 검증을 할 수 있습니다.

[ValidationUtils](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/validation/ValidationUtils.html)

[Pattern](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html)

> ### MemberController

```java
package com.kei890.validationtutorial.controller;

import javax.validation.Valid;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;

import com.kei890.validationtutorial.dto.MemberDto;
import com.kei890.validationtutorial.dto.MemberValidator;

@Controller
public class MemberController {

	@Autowired
	MemberValidator memberValidator;

	@InitBinder
	protected void initBinder(WebDataBinder binder) {
		binder.addValidators(memberValidator);
	}

	
	@RequestMapping("/")
	public String moveTestJsp(Model model) {
		// 빈 객체 넘겨줘야함.
		model.addAttribute("memberDto", new MemberDto());
		return "insert";
	}

	@RequestMapping("/insert")
	public String memberInsert(@Valid @ModelAttribute MemberDto memberDto, BindingResult result) {

		System.out.println("---- MemberController::memberInsert() ----");

		System.out.println(memberDto.toString());

		System.out.println("오류가 있나요? : " + result.hasErrors());

		if (result.hasErrors()) {
			// form에 에러가 있으면
			return "insert";
		}
		// 에러가 없으면
		return "";

	}

}
```

컨트롤러에서 앞에서 구현해준 `memberValidator`를 <u>DI</u> 해주고,
`@initBinder` 어노테이션을 이용해서 validator를 추가해주면 정상적으로 동작을 합니다.

index.jsp는 수정 없이 그대로 사용해도 됩니다.

> ### 정상적으로 동작하는 모습 

[![img](https://4.bp.blogspot.com/-JI01uajSrUc/XLMiQaNU_GI/AAAAAAAABKU/OSonnYS0wRs8br_0cZb-9A4zL0c9jga7gCLcBGAs/s1600/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%2B2019-04-14%2B%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%2B9.04.50.png)](https://4.bp.blogspot.com/-JI01uajSrUc/XLMiQaNU_GI/AAAAAAAABKU/OSonnYS0wRs8br_0cZb-9A4zL0c9jga7gCLcBGAs/s1600/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%2B2019-04-14%2B%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%2B9.04.50.png)

[Validator 깃허브 소스](https://github.com/DaeAkin/ValidationTutorial/tree/validator)

## 마치며

자바스크립트에서 유효성 검사를 하지만 자바스크립트의 유효성 검사만으로는 안됩니다.

자바스크립트의 유효성 검사는 서버에 전송되는 횟수를 막아주는 것 일뿐, 

서버에서도 유효성 검사를 반드시 해주어야 합니다.
