# JPA의 DTO와 Entity

이번 시간에는 제가 JPA를 사용하면서 Entity와 DTO의 적절히 사용했던 경험에 대해 알려드리려고 합니다.



회원가입의 Entity가 있다고 가정해 보겠습니다. 회원의 관련된 Entity인 User 클래스는 다음과 같습니다.

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



그러나 회원가입을 진행할 때 클라이언트로부터 받는 데이터는 email과 nickname 그리고 password만 필요합니다. 이 3개의 데이터를 어떻게 받아 올 수 있을까요?

저는 주로 각 기능마다 필요로 하는 필드들을 모은 DTO 클래스를 만들어 받습니다. 

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
		User user =	userRepository.save(user); //??
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

toEntity() 함수에서 User 객체를 만들때는 builder 패턴을 이용했습니다.

서비스 단에서는 다음과 같이 사용합니다.

```java
public UserJoinResponse joinUser(UserJoinRequest userJoinRequest) {
  	User saveUser = userJoinRequest.toEntity();
		User user =	userRepository.save(user);
}
```

정상적으로 User가 데이터베이스에 저장되었다면, 클라이언트에게 보내줄 데이터를 만들어 보겠습니다.

저는 예제로 클라이언트에게 유저의 email을 클라이언트에게 보내보겠습니다. 
현재 서비스단의 `joinUser()` 함수의 리턴 값은 UserJoinResponse 이기 때문에 User를 UserJoinResponse로 변경해주는 작업이 필요합니다.

이 또한 이 변경의 `책임` 은 누구한테 있는지 고민한다면 `User` 에게 있을 것입니다. 그러므로 User 클래스 안에 UserJoinResponse 객체를 만들어 주는 함수가 있어야 합니다.

회원가입이 성공적으로 완료되면 클라이언트에게 email 값을 주는 UserJoinResponse 입니다.

```java
public class UserJoinResponse {
		
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



## 방법

- modelmapper 쓰는방법

  ```
  MyEntity mye = repository.finById(id);
  ModelMapper mm = new ModelMapper();    
  mm.map(myDTO, mye);
  repository.save(mye):
  ```

- dozer? 

## 참고문서

https://stackoverflow.com/questions/5216633/jpa-entities-and-vs-dtos

https://auth0.com/blog/automatically-mapping-dto-to-entity-on-spring-boot-apis/

