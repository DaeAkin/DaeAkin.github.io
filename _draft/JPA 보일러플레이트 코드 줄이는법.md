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
public void joinUser(@RequestBody @Valid UserJoinRequest userJoinRequest) {
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
이 작업은 어떤 식으로 진행하는 것이 좋을까요?



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

