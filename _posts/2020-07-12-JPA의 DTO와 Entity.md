---
layout: post
title: JPA의 DTO와 Entity
categories: [JPA]
comments: true 
---



# ❗️JPA의 DTO와 Entity

이번 시간에는 제가 JPA를 사용하면서 Entity와 DTO의 적절히 사용했던 경험에 대해 알려드리려고 합니다.

보통 회원가입 기능을 만든다고하면, 회원에 관련된 데이터베이스 컬럼과, 회원가입 폼에서 사용하는 컬럼은 대부분 다를 것 입니다. 

회원가입의 Entity가 있다고 가정해 보겠습니다. 회원의 관련된 Entity인 `User 클래스`는 다음과 같습니다.

```java
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Column(nullable = false)
    private String email;

    private String nickname;

    @JsonIgnore
    private String password;

    @Column(columnDefinition="bit(1) default 1")
    private boolean enabled;

    @Column(columnDefinition="bit(1) default 1")
    private boolean acceptOptionalBenefitAlerts;

    private String fcmToken;

    private boolean privateAccount;
}
```

그러나 회원가입을 진행할 때 클라이언트로부터 받는 데이터는 `email`과 `nickname` 그리고 `password`만 필요합니다. 이 3개의 데이터를 어떻게 받아 올 수 있을까요?

저는 주로 각 기능마다 필요로 하는 필드들을 모은 **DTO 클래스**를 만들어 받습니다. 

예를 들어 회원가입 로직이 들어가있는 API의 형태는 다음과 같습니다.

```java
@PostMapping("/join")
public UserJoinResponse joinUser(@RequestBody @Valid UserJoinRequest userJoinRequest) {
	return userService.joinUser(userJoinRequest);
}
```

UserJoinRequest의 구성은 다음과 같습니다.

```java
public class UserJoinRequest {
    @NotBlank(message = "이메일을 다시 확인해주세요.")
    String email;

    @NotBlank(message = "비밀번호를 다시 확인해주세요.")
    String password;

    @NotBlank(message = "닉네임을 다시 확인해주세요.")
    String nickname;
}
```

그러나 Service 단에서, DAO로 데이터를 저장시키기 위해서 이 DTO를 Entity로 변환해주는 작업이 필요합니다.

```java
public UserJoinResponse joinUser(UserJoinRequest userJoinRequest) {
  User user=userRepository.save(user); //??
}
```

이 작업은 어떤 식으로 진행하는 것이 좋을까요?

UserJoinRequest를 User Entity로 변환해주는 `책임`은 바로 UserJoinRequest에게 있습니다. 그렇기 때문에 UserJoinRequest가 User로 변환을 해주는 로직이 필요합니다.

```java
public class UserJoinRequest {
    @NotBlank(message = "이메일을 다시 확인해주세요.")
    String email;

    @NotBlank(message = "비밀번호를 다시 확인해주세요.")
    String password;

    @NotBlank(message = "닉네임을 다시 확인해주세요.")
    String nickname;

    public User toEntity() {
        return User.builder()
                .email(email)
                .nickname(nickname)
                .password(password)
                .build();
    }
}
```

toEntity() 함수에서 User 객체를 만들때는 [Builder 패턴](https://daeakin.github.io//articles/2020-02/%EB%B9%8C%EB%8D%94-%ED%8C%A8%ED%84%B4)을 이용했습니다.

서비스 단에서는 다음과 같이 사용합니다.

```java
public UserJoinResponse joinUser(UserJoinRequest userJoinRequest) {
  User saveUser = userJoinRequest.toEntity();
  User user = userRepository.save(user);
}
```

정상적으로 User가 데이터베이스에 저장되었다면, 클라이언트에게 보내줄 데이터를 만들어 보겠습니다.

저는 예제로 클라이언트에게 유저의 email을 클라이언트에게 보내보겠습니다. 
현재 서비스단의 `joinUser()` 함수의 리턴 값은 UserJoinResponse 이기 때문에 **User를 UserJoinResponse로 변경해주는 작업**이 필요합니다.

이 또한 이 변경의 `책임` 은 누구한테 있는지 고민한다면 `User` 에게 있을 것입니다. 그러므로 User 클래스 안에 UserJoinResponse 객체를 만들어 주는 함수가 있어야 합니다.

회원가입이 성공적으로 완료되면 클라이언트에게 email 값을 주는 UserJoinResponse 입니다.

```java
public class UserJoinResponse {
    String email;
}
```

User의 함수는 다음과 같습니다.

```java
public class User {
	...필드 생략
	
	public UserJoinResponse toUserJoinResponse() {
		 return User.UserJoinResponse()
                .email(email)
                .build();
	}
}
```

지금까지 제가 JPA를 사용하면서 주로 사용했던 내용을 작성했습니다.

저는 위와 같은 방법을 주로 사용하지만, 이렇게 클래스마다 Entity <-> DTO 변환 클래스를 작성하지 않아도, 자동으로 변환해주는 라이브러리가 있어서 소개해 드리려고 합니다.

## 😯 ModelMapper

ModelMapper 라이브러리를 사용하게 되면 간편하게 Entity <-> DTO 변환이 가능해 집니다.

```java
public UserJoinResponse joinUser(UserJoinRequest userJoinRequest) {
  ModelMapper mm = new ModelMapper(); 
  User user = new User();
  mm.map(userJoinRequest, user);
  User user =	userRepository.save(user);
}
```

이런 방법도 있지만, 개인적으로 처음 설명해드렸던 방법을 추천합니다.

## 🧐 Entity vs DTO

이렇게 특정 필드만 받을 수 있게 DTO 클래스를 만들었습니다. 그러나 프로젝트가 커지게 되면 DTO 클래스는 계속 증가합니다.

그냥 Entity 클래스를 받아서 필요한 값만 추출해서 사용하면 될것인데, 왜 굳이 이렇게 DTO 클래스를 새로 만들어서 사용 할까요?

DTO 클래스를 사용하는 이유는 다음과 같습니다.

서비스 단은 데이터베이스와 독립적이어야 합니다. 데이터베이스의 변경사항이 있으면, Entity를 사용하는 서비스 단에도 변경이 일어날 수 있습니다. 

또한 Entity 클래스를 DTO로 재사용 하는 일은 코드가 더러워질 수 있습니다. 클래스가 DTO로써 사용 될 때 사용하는 메소드와 Entity로써 사용 될 때 사용하는 메소드가 공존하게 됩니다. 이런 점에서 `관심사` 가 깔끔하게 분리되지 않고, 클래스의 결합력만 높이게 됩니다. 

## 참고문서

https://stackoverflow.com/questions/5216633/jpa-entities-and-vs-dtos

https://auth0.com/blog/automatically-mapping-dto-to-entity-on-spring-boot-apis/

