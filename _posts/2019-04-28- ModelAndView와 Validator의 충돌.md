---
layout: post
title: ModelAndView와 Validator의 충돌
categories: [Spring]
comments: true
image:

  feature: 충돌.png
---

# ModelAndView와 Validator의 충돌

내가하고 있는 프로젝트의 컨트롤단에서 ModelAndView를 사용해서

Model과 리턴할 ViewName을 지정하는 방법을 사용했다.

```java
@RequestMapping("/member")
@Controller
public class MemberController extends CRUDController<MemberDto, Integer,MemberService>{
	
	@Autowired
	MemberService memberService;
	
	@Autowired
	MemberMailSender memberMailSender;
	
	@Autowired
	MemberValidator memberValidator;
	
	@InitBinder
	protected void initBinder(WebDataBinder binder) {
		binder.addValidators(memberValidator);
		
	}
	
	@RequestMapping(value = "/login", method = RequestMethod.POST)
	public ModelAndView memberLogin(@Valid @ModelAttribute MemberDto memberDto,
		BindingResult result,HttpSession session,ModelAndView mv) {
	// 로그인 로직과 view의 이름을 적어주는 코드는 생략
    		}
			}	
}
```

이 코드는 Validator를 추가하기전 까지 잘 작동하던 코드였으나,

Vaildator를 추가하고부터 컨트롤러 테스트 코드가 오류를 계속냈다.

오류가 뭔지 몰라서 Validator에 로그를 찍어 보았다.

```
supports ? :true
validator 통과 : com.www.homedoc.dto.MemberDto
supports ? :false
validator 통과 : org.springframework.web.servlet.ModelAndView
```

ModelAndView 쪽에서 뭔가 문제가 난 것 같다.

오류내용은 다음과 같았다.

```
org.springframework.web.util.NestedServletException: Request processing failed; nested exception is java.lang.IllegalStateException: Invalid target for Validator [com.www.homedoc.dto.MemberValidator@14c01636]: ModelAndView: materialized View is [null]; model is null
```

오류 내용에 의하면 View가 null이고 model이 null이라고 한다.

## 이유

ModelAndView를 저렇게 파라미터로 사용하는 경우 어떻게 객체를 생성하여 어떤 생성자를 호출할까?

ModelAndView 문서를 한번 봐보자.

![image](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/mv/image1.png?raw=true)

파라미터로 사용하는 경우 기본 생성자인 `ModelAndView()`를 사용하게된다.

컨트롤러에서 View와 Model 코드를 작성했음에도 불구하고,

<u>Validator가 먼저 실행되어 View 와 Model이 null이 되게된다.</u>

그래서 Validator에 의해 null 값이 있는 것들을 잡아내서 Exception을 뿜어준다.

## 해결법

ModelAndView를 굳이 사용하고싶다면, Validator에 걸리지 않도록,

컨트롤러 메소드안에 `ModelAndView mv = new ModelAndView();` 를 사용해서

객체를 생성해주거나, 그냥 return 값으로 String을 주어 View이름만 반환해서 사용하도록 하면된다.



소스코드를 줄이려고 저렇게 사용했는데, 오류가 있는지 몰랐다..
