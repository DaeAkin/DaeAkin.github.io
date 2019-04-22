---

```
layout: post
title: 자바 메일 보내기
categories: [Spring]
comments: true
image:

  feature: Mail.png
```

---



# 자바 메일 보내기 



## 필수라이브러리 

```xml
<!-- Java Mail API -->
<dependency>
    <groupId>com.sun.mail</groupId>
    <artifactId>javax.mail</artifactId>
    <version>1.6.1</version>
</dependency>
<!-- for Junit test -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>${org.springframework-version}</version>
</dependency>
```



## Mail 인증 설정하기 

```xml
<bean id="mailSender"
		class="org.springframework.mail.javamail.JavaMailSenderImpl">
		<property name="host" value="smtp.gmail.com" />
		<property name="port" value="587" />
		<property name="username" value="사용할 이메일 아이디" />
		<property name="password" value="비빌번호" />
		<property name="javaMailProperties">
			<props>
				<prop key="mail.transport.protocol">smtp</prop>
				<prop key="mail.smtp.auth">true</prop>
				<prop key="mail.smtp.starttls.enable">true</prop>
				<prop key="mail.debug">true</prop>
			</props>
		</property>
	</bean>
```



## GoogleMail에서 설정해주기

만약 Google 메일을 이용하신다면 

밑의 링크에 들어가서 **<u>보안 수준이 낮은 앱의 액세스</u>**를 사용 함으로 변경해주어야 합니다.

[보안 수준이 낮은 앱의 액세스](https://myaccount.google.com/lesssecureapps?pli=1)

<img src="https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/emailvalidationTutorial/image1.png?raw=true" width=100%/>

## 회원가입축하 메일 보내보기

이제 Mail 관련 로직을 작성해보겠습니다.

회원가입을 하면 회원가입축하 메일을 발송해주곤 하는데요,

이런 회원가입축하 메일을 발송해주는 로직을 작성해보겠습니다.

> #### CongratulationMailSender

```java
package com.daeakin.emailvalidationtutorial.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.stereotype.Component;
@Component
public class CongratulationMailSender {
	
	@Autowired
	JavaMailSender mailSender;
	
	public void sendCongratulationMail(String email) {
		SimpleMailMessage message = new SimpleMailMessage();
		
		message.setSubject("회원가입을 진심으로 축하합니다.");
		message.setText("환영합니다!!");
		message.setFrom("DaeAkin");
		message.setTo(email);
		
		mailSender.send(message);
	}

}
```

이제 이 로직을 테스트 하기 위한 테스트코드를 작성하겠습니다.

> #### MailTest

```java
package com.daeakin.emailvalidationtutorial.test;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.daeakin.emailvalidationtutorial.service.CongratulationMailSender;

@RunWith(SpringJUnit4ClassRunner.class)
public class MailTest {
	
	@Autowired
	CongratulationMailSender sender;
	
	@Test
	public void congMailTest() {
        //파라미터로 보내고싶은 메일주소 전달
		sender.sendCongratulationMail("mindonghyeon890@gmail.com");
	}

}
```

테스트용 xml 파일도 같이 작성해주세요.

**MailTest-context.xml**로 작성하시면 JUnit에서 자동으로 이 xml 파일을 사용하게 됩니다.

> #### MailTest-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
	
<!-- @Component 어노테이션 붙은 클래스를 자동으로 빈등록 해준다 -->	
<context:component-scan base-package="com.daeakin.emailvalidationtutorial.*" />
		<!-- Mail 인증 관련 -->
	<bean id="mailSender"
		class="org.springframework.mail.javamail.JavaMailSenderImpl">
		<property name="host" value="smtp.gmail.com" />
		<property name="port" value="587" />
		<property name="username" value="사용할 이메일 아이디" />
		<property name="password" value="비빌번호" />
		<property name="javaMailProperties">
			<props>
				<prop key="mail.transport.protocol">smtp</prop>
				<prop key="mail.smtp.auth">true</prop>
				<prop key="mail.smtp.starttls.enable">true</prop>
				<prop key="mail.debug">true</prop>
			</props>
		</property>
	</bean>
		
</beans>
```



### 결과

> #### 정상적으로 회원가입 축하 메일을 받은 모습

<img src="https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/emailvalidationTutorial/image2.png?raw=true" width=100%/>



## 회원가입 인증번호 보내보기

방금 만든 회원가입 이메일은 <u>HTML</u>이 아니고 <u>text</u> 로만 발송이 가능합니다. 

그럼 HTML을 보내고 싶으면 어떻게 하면 될까요? 

HTML 문서와 함께 인증번호를 보내보도록 하겠습니다.

**CongratulationMailSender** 클래스 안에 `authenticationSend()` 함수를 작성해주세요.

> #### CongratulationMailSender

```java
package com.daeakin.emailvalidationtutorial.service;

import java.util.Random;

import javax.mail.internet.MimeMessage;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.mail.javamail.MimeMessagePreparator;
import org.springframework.stereotype.Component;

@Component
public class CongratulationMailSender {

	@Autowired
	JavaMailSender mailSender;
	
	//랜덤 값 생성을 위한 Random
	Random random = new Random();

	public void sendCongratulationMail(String email) {
		SimpleMailMessage message = new SimpleMailMessage();

		message.setSubject("회원가입을 진심으로 축하합니다.");
		message.setText("환영합니다!!");
		message.setFrom("DaeAkin");
		message.setTo(email);

		mailSender.send(message);
	}

	public void authenticationSend(String email) {
		MimeMessagePreparator mimeMessagePreparator = new MimeMessagePreparator() {

			@Override
			public void prepare(MimeMessage paramMimeMessage) throws Exception {
				MimeMessageHelper message = new MimeMessageHelper(paramMimeMessage, true, "UTF-8");

				message.setTo(email);
				message.setFrom("Daeakin");
				message.setSubject("회원가입 인증 메일입니다.");

				String content = "<!doctype html> " + "<html lang='ko'> " + " <head>" + "<title>이메일 인증</title>"
						+ "<meta charset=\"utf-8\">\n"
						+ "\n"
						+ "				  </head>\n" + "				  <body>\n"
						+ "				    <div class=\"jumbotron jumbotron-fluid\">\n"
						+ "				  <div align=\"center\" class=\"container\">\n"
						+ "				    <h1 class=\"display-3\">이메일 인증</h1><br>\n"
						+ "				    <p class=\"lead\">안녕하세요. <font size=\"5\" 							color=\"red\"><b>OOOO</b></font> 입니다.\n"
						+ "				        <br>\n" + "				        인증번호는 다음과 같습니다.<br><font size=\"7\"> [ "
						+ random.nextInt(100000) +
						" ] </font>\n" + "				      </p>\n"
						+ "				         <p class=\"lead\">\n"
						+ "				    <a class=\"btn btn-success btn-lg\" href=\"\" role=\"button\">홈페이지 바로가기</a>\n"
						+ "				  </p>\n" + "				  </div>\n" + "				</div>\n" + "\n"
						+ "				  </body>\n" + "				</html>";

				//true로 해줘야 HTML을 사용한다고 알림! 
				message.setText(content, true);

			}
				
		};
		mailSender.send(mimeMessagePreparator);
	}

}
```

### 테스트 하기

> #### MailTest

```java
package com.daeakin.emailvalidationtutorial.test;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.daeakin.emailvalidationtutorial.service.CongratulationMailSender;

@RunWith(SpringJUnit4ClassRunner.class)
public class MailTest {
	
	@Autowired
	CongratulationMailSender sender;
	
	@Test
	public void congMailTest() {
		sender.authenticationSend("mindonghyeon890@gmail.com");
	}
}
```

### 결과

<img src="https://github.com/DaeAkin/DaeAkin.github.io/blob/master/img/blog/emailvalidationTutorial/image4.png?raw=true" width=100%/>



## 마치며

랜덤으로 만든 인증번호를 Collection 배열에 넣었다가,

유저들이 인증번호를 입력하면 Collection에서 가져와 비교를 하는 방법이나,

DB에 인증번호를 넣어서 비교를 하는 방법을 사용해서 

인증을 나머지 구현하시면 됩니다.