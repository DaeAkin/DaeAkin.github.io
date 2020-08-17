---
layout: post
title: Spring Annotation의 원리와 Custom Annotation 만들어보기
categories: [Spring]
comments: true 
---

# ‼️ Spring Annotation의 원리와 Custom Annotation 만들어보기

Spring에서 bean을 만드는 방법은 여러가지 있지만, 그 중에서 @Component 어노테이션을 이용하는 방법이 있습니다. 이 어노테이션은 어떻게 만들고, 어떻게 작동을 할까요? 

##### @Component 어노테이션

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {
    String value() default "";
}
```


Component @Target, @Rentetion, @Documneted 총 3개의 어노테이션과 @Indexed 어노테이션을 갖고 있습니다.


J2SE 5.0 부터 어노테이션을 작성할 때 주로 java.lang.annotation 패키지에 있는 4가지 어노테이션을 이용하여 작성합니다.

- @Documented : Java doc에 문서화 여부를 결정합니다.

- @Retention : 어노테이션의 지속 시간을 정합니다.

  - RetentionPolicy.SOURCE : 컴파일 후에 정보들이 사라집니다. 이 어노테이션은 컴파일이 완료된 후에는 의미가 없으므로, 바이트 코드에 기록되지 않습니다. 예시로는 @Override와 @SuppressWarnings 어노테이션이 있습니다.
  - RetentionPolicy.CLASS : default 값 입니다. **컴파일 타임** 때만 .class 파일에 존재하며, **런타임** 때는 없어집니다. 바이트 코드 레벨에서 어떤 작업을 해야할 때 유용합니다. Reflection 사용이 불가능 합니다.

  - RetentionPlicy.RUNTIME : 이 어노테이션은 런타임시에도 .class 파일에 존재 합니다. 커스텀 어노테이션을 만들 때 주로 사용합니다. Reflection 사용 가능이 가능합니다.
  
- @Target : 어노테이션을 작성할 곳 입니다. **default 값은 모든 대상입니다.** 예를 들어 **@Target(ElementType.FIELD)**로 지정해주면, 필드에만 어노테이션을 달 수 있습니다. 만약 필드 말고 다른부분에 어노테이션을 사용한다면 컴파일 때 에러가 나게 됩니다.

  > *ElementType.TYPE (class, interface, enum)*
  >
  > *ElementType.FIELD (instance variable)*
  >
  > *ElementType.METHOD*
  >
  > *ElementType.PARAMETER*
  >
  > *ElementType.CONSTRUCTOR*
  >
  > *ElementType.LOCAL_VARIABLE*
  >
  > *ElementType.ANNOTATION_TYPE (on another annotation)*
  >
  > *ElementType.PACKAGE (remember package-info.java)*

- @Inherited : 자식클래스에 상속할지 결정



![](https://github.com/DaeAkin/java-custom-annotation/blob/master/docs/javacompiler.png?raw=true)





## 🎯 Custom 어노테이션 만들기

커스텀 어노테이션을 만들 때 몇가지 규칙이 있습니다.

1. 어노테이션 타입은 @interface로 정의해야합니다. 모든 어노테이션은 자동적으로 `java.lang.Annotation` **인터페이스를 상속하기 때문에** 다른 클래스나 인터페이스를 상속 받으면 안됩니다.
2. 파라미터 멤버들의 접근자는 public이거나 default여야만 합니다.
3. 파라미터 멤버들은 byte,short,char,int,float,double,boolean,의 기본타입과 String, Enum, Class, 어노테이션만 사용할 수 있습니다.
4. 클래스 메소드와 필드에 관한 어노테이션 정보를 얻고 싶으면, **리플렉션만** 이용해서 얻을 수 있습니다. 다른 방법으로는 어노테이션 객체를 얻을 수 없습니다.

Custom 어노테이션을 총 **3개**를 만들어 보겠습니다.

- FruitColor : 과일에 색을 지정해주는 어노테이션
- FruitName : 과일의 이름을 지정해주는 어노테이션 
- FruitProvider : 과일을 파는 곳을 지정해주는 어노테이션


##### FruitColor 어노테이션 만들기
```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitColor {
    enum Color{BLUE,RED,GREEN}

    Color fruitColor() default Color.GREEN;
}
```

만약 FruitColor 어노테이션의 Color 값을 주지 않으면 default로 GREEN이 사용 됩니다.

##### FruitName 어노테이션 만들기

``` java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitName {
    String value() default "";
}

```

##### FruitProvider 어노테이션 만들기

``` java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitProvider {

    int id() default -1;

    String name() default "";

    String address() default "";
}
```

총 3개의 어노테이션을 만들었습니다.
이제 Apple 이라는 클래스를 만들어 이 어노테이션들을 사용해보겠습니다.


##### Apple.java
```java
public class Apple {

    @FruitName("Apple")
    private String appleName;

    @FruitColor(fruitColor = FruitColor.Color.RED)
    private String appleColor;

    @FruitProvider(id = 1,name = "HomePlus",address = "Seoul")
    private String appleProvider;
    
    ...getter setter
    }
```



##### **FruitInfoUtil.java**

```java
public class FruitInfoUtil {
    public static void getFruitInfo(Class<?> clazz) {

        String strFruitName = " 과일 이름 :";
        String strFruitColor = " 과일 색 :";
        String strFruitProvider = "과일 파는 곳";

        Field[] fields = clazz.getDeclaredFields();

        for (Field field : fields) {
            if (field.isAnnotationPresent(FruitName.class)) {
                FruitName fruitName = field.getAnnotation(FruitName.class);
                strFruitName = strFruitName + fruitName.value();
                System.out.println(strFruitName);
            } else if (field.isAnnotationPresent(FruitColor.class)) {
                FruitColor fruitColor = field.getAnnotation(FruitColor.class);
                strFruitColor = strFruitColor + fruitColor.fruitColor().toString();
                System.out.println(strFruitColor);
            } else if (field.isAnnotationPresent(FruitProvider.class)) {
                FruitProvider fruitProvider = field.getAnnotation(FruitProvider.class);
                strFruitProvider = " 과일 파는 곳의 ID: " + fruitProvider.id() + " 지점 이름 : " + fruitProvider.name() + " 지점 주소: " + fruitProvider.address();
                System.out.println(strFruitProvider);
            }
        }
    }
}
```

FruitInfoUntil 클래스는, Custom 어노테이션이 사용된 클래스들을 모아서 출력합니다.

어노테이션이 어디서 사용되었는지 알아야 하기 때문에, 리플렉션 API인 class.getDeclaredFields()를 사용하여, 어노테이션이 사용된 클래스들을 전부 가져오게 됩니다.

어노테이션을 만들 때 **@Retention(RetentionPolicy.RUNTIME)** 속성을 줬기 때문에 리플렉션을 사용할 수 있습니다. 만약 **@Retention(RetentionPolicy.CLASS)** 속성을 줬다면, 리플렉션을 사용할 수 없습니다.

##### FruitRun.java

```java
public class FruitRun {

     public static void main(String[] args) {
          FruitInfoUtil.getFruitInfo(Apple.class);
      }
}
```



##### 결과

```
 과일 이름 :Apple
 과일 색 :RED
 과일 파는 곳의 ID: 1 지점 이름 : HomePlus 지점 주소: Seoul
```




## 💡 Spring은 어노테이션을 어떻게 사용하고 있을까?

Spring에서 애플리케이션을 실행 시 @Service나, @Component가 붙은 클래스들을 스캔해서 IoC 컨테이너에 등록해주는 과정이 있습니다. 

##### @Service 어노테이션

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}
```

Service 어노테이션은, Target이 TYPE으로 지정되어 있습니다. **Class나 Interface를 타겟**으로 삼는다는 의미 입니다.

또한 같은 패키지인`org.springframework.stereotype` 의 **Component 어노테이션**을 쓰고 있는걸 볼 수 있습니다.

##### @Component 어노테이션

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {
    String value() default "";
}
```

Spring에서는 @Component , @Service , @Controller 등 어노테이션이 사용된 클래스는 **<u>bean</u>**으로 등록하게 됩니다. 등록해주는 스프링의 코드는 다음과 같습니다.

##### ClassPathBeanDefinitionScanner.java

![](https://github.com/DaeAkin/java-custom-annotation/blob/master/docs/%EC%8A%A4%ED%8A%B8%EB%A0%88%EC%98%A4%ED%83%80%EC%9E%85%20%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98%EC%9D%84%20%EB%93%B1%EB%A1%9D%ED%95%B4%EC%A3%BC%EB%8A%94%20%ED%81%B4%EB%9E%98%EC%8A%A4.png?raw=true)

doScan() 메소드가 ClassPath에 있는 패키지의 모든 클래스를 읽어, 어노테이션이 붙은 클래스를 처리해주는 클래스(ex: IoC 컨테이너에 클래스 등록 등.) 입니다. 
앞에서 만들어 봤던 FruitInfoUtil 클래스와 비슷한 역할을 한다고 생각하시면 됩니다.



## 🔚 마치며

스프링을 사용하면서 많은 어노테이션을 사용하지만, 이 어노테이션은 어떻게 만들고, 어떻게 처리할까?에 대한 의구심이 항상 있었습니다. 예를 들어, @Component 어노테이션을 클래스에 붙이기만 하면 자동으로 그 클래스 이름으로 Bean 으로 등록되는데 사실은 알고 있었지만, 이 과정이 어떻게 진행되는지 항상 궁금했었습니다. 이번 포스팅으로 인해, 스프링의 대한 기본 개념이 확실하게 굳어지는 것 같아 포스팅 하길 잘했단 생각이 듭니다.



## 참고자료

[RetentionPoliy 설명 글](https://stackoverflow.com/questions/3107970/how-do-different-retention-policies-affect-my-annotations)

