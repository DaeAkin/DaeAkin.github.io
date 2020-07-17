# ‼️ 스프링 부트의 autoconfiguration



## Auto-configuration

Spring Boot의 auto-configuration은 추가한 jar 파일에 따라 자동적으로 설정을 해줍니다. 예를 들어 HSQLDB가 클래스패스에 존재하면, 데이터베이스의 커넥션을 맺는 bean을 수동으로 구성해주지 않았다면, 자동으로 인메모리 DB로 자동 구성 됩니다.

@EnableAutoConfiguration 또는 @SpringBootApplication 주석을 @Configuration 클래스 중 하나에 추가하면 됩니다.

> @SpringBootApplication 또는 @EnableAutoConfiguration 주석을 하나만 추가해야합니다. 일반적으로 기본 @Configuration 클래스에만 하나를 추가하는 것이 좋습니다.



## Auto-configuration으로 점진적으로 변경하기

언제든지 특정 부분을 auto-configuration으로 변경할 수 있습니다. 예를 들어 DataSource 빈을 사용하고 있다면, 디폴트로 사용되는 임베디드 데이터베이스는 더 이상 사용되지 않습니다.

만약 현재 어느 부분에 auto-configuration이 적용되어있는지 알고 싶다면 애플리케이션을 `--debug` 과 함께 실행시키면 됩니다.  이렇게 함으로써 logger의 debug를 활성화 시킬 수 있어, 콘솔로 확인할 수 있습니다.



## 특정 클래스 Auto-configuration  비활성화하기 



















## 참고자료

https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration

