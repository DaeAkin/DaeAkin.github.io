# JPA의 DTO와 Entity

이번 시간에는 제가 JPA를 사용하면서 Entity와 DTO의 적절히 사용했던 경험에 대해 알려드리려고 합니다.





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

