# 스프링5 프로그래밍 입문

## Chapter 4 의존 자동 주입
> `@Autowired`를 이용한 의존 자동 주입에 대해 알아보자
**자동 주입**: 의존 대상을 설정 코드에서 직접 주입하지 않고 스프링이 자동으로 의존하는 빈 객체를 주입해주는 기능








### 1.예제 프로젝트 준비

### 2. @Autowired 애노테이션을 이용한 의존 자동 주입
>**자동 주입 방법**: 의존을 주입할 대상에 @Autowired 애노테이션을 붙여서 스프링이 자동으로 빈을 주입하도록 한다.

필드, 메서드 등에 주입할 수 있다. 




#### 필드 주입
```java
import org.springframework.beans.factory.annotation.Autowired;

public class ChangePasswordService {

    @Autowired
    private MemberDao memberDao; 

    public void changePassword(String userId, String newPassword) {
        memberDao.updatePassword(userId, newPassword);
    }
}
``` 

>**설명**: 이렇게 하면 `ChangePasswordService`가 생성될 때, 스프링이 자동으로 `MemberDao` 객체를 `memberDao` 필드에 주입한다. 





#### if `@Autowired` 애노테이션을 설정한 필드에 알맞은 빈 객체가 주입되지 않는다면? 
= memberDao가 주입되지 않으면?

> `ChangePasswordService`의 `memberDao` 필드가 `null`-> 암호 변경 기능을 실행 시 `NullPointerException` 가 나옴


#### if 일치하는 빈이 없다면?

> `@Autowired` 애노테이션을 사용했지만 일치하는 빈이 없는 경우,스프링은 `NoSuchBeanDefinitionException`을 발생시킴. `MemberDao` 타입의 빈이 스프링 컨테이너에 등록되지 않았기 때문이다.



>따라서 암호 변경 기능이 정상 작동함 = `@Autowired` 애노테이션을 붙인 필드에 실제 `MemberDao` 타입의 빈 객체가 잘 들어감












### 3. @Qualifier 애노테이션을 이용한 의존 객체 선택
> 자동 주입 가능한 빈이 두 개 이상이면 자동 주입할 빈을 지정할 수 있는 방법이 필요 -> @Qualifier 애노테이션을 사용하여 여러 빈 중에서 특정 빈을 선택할 수 있다.



```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;

public class ChangePasswordService {

    private final MemberDao memberDao;

    @Autowired
    public ChangePasswordService(@Qualifier("myMemberDao") MemberDao memberDao) {
        this.memberDao = memberDao; 
    }

    public void changePassword(String userId, String newPassword) {
        memberDao.updatePassword(userId, newPassword);
    }
}
```


#### @Qualifier의 두 가지 사용 위치

> **1. 빈 설정 메서드에서 @Qualifier 사용**
 @Bean 애노테이션을 사용하여 빈을 등록할 때, @Qualifier를 통해 빈의 이름을 명시할 수 있다. 
예제:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.beans.factory.annotation.Qualifier;

@Configuration
public class AppCtx {

    @Bean
    @Qualifier("memberDao1")
    public MemberDao memberDao1() {
        return new MemberDao1();
    }

    @Bean
    @Qualifier("memberDao2")
    public MemberDao memberDao2() {
        return new MemberDao2();
    }
}
이 설정에서는 @Qualifier를 사용하여 빈의 이름을 지정하고, 이 이름으로 빈을 구분합니다.
```


> **2. 의존성 주입에서 @Qualifier 사용**
@Autowired와 함께 사용되는 @Qualifier는 의존성 주입 시 특정 빈을 선택하는 데 사용된다. 필드 주입, 메서드 주입, 생성자 주입에서 모두 가능하다.

```java

public class ChangePasswordService {

    @Autowired
    @Qualifier("memberDao1")
    private MemberDao memberDao;

    public void changePassword(String userId, String newPassword) {
        memberDao.updatePassword(userId, newPassword);
    }
}
```

-메서드 주입: 생성자 또는 세터 메서드의 파라미터에 @Qualifier를 사용하여 빈을 주입가능 
*맨 위 예시 참고



> 빈 설정 메서드에서의 사용은 빈을 등록할 때 이름을 지정하는 것이고, 의존성 주입에서의 사용은 이미 등록된 여러 빈 중에서 어떤 것을 주입할지 명시적으로 선택하는 것





#### 3.1 빈 이름과 기본 한정자
> 별도의 @Qualifier 애노테이션을 사용하지 않으면 기본 한정자로는 빈 이름이 사용
따라서 같은 타입의 빈이 여러 개면 @Qualifier 애노테이션을 써서 한정자를 따로 명시해주자











### 4. 상위/하위 타입 관계와 자동 주입
> **상속 관계에서 발생할 수 있는 문제점**:

MemberPrinter를 상속한  MemberSummaryPrinter클래스가 있다고 하자.
```java
@Configuration
public class AppConfig {

    @Bean
    public MemberPrinter memberPrinter1() {
        return new MemberPrinter();
    }

    @Bean
    public MemberPrinter memberPrinter2() {
        return new MemberSummaryPrinter();
    }
}


public class MemberService {

    @Autowired
    private MemberPrinter memberPrinter;

   
}
```

겉으로는 다른 타입이여서 문제 없을 거 같지만 @Autowired를 사용하면, 실제로는 MemberSummaryPrinter클래스는 MemberPrinter에 할당될 수 있어서 스프링이 두 개의 빈 중 어느 것을 주입해야할 지 모호해지는 문제가 발생 


> **두 가지 해결 방법**:

1. @Qualifier 사용하기
```java
public class MemberService {

    @Autowired
    @Qualifier("memberPrinter1")
    private MemberPrinter memberPrinter;

}```


2. 상속 구조 활용하기 

@Configuration
public class AppConfig {

    @Bean
    public MemberPrinter memberPrinter() {
        return new MemberSummaryPrinter();
    }
}
```
MemberSummaryPrinter 타입은 하나만 존재하므로 MemberSummaryPrinter 빈을 자동 주입 받도록 코드를 수정하면 자동 주입 대상이 두 개여서 발생하는 문제를 해결할 수있음.



### 5. @Autowired 애노테이션의 필수 여부
> 5.1 생성자 초기화와 필수 여부 지정 방식 동작 이해

**기본값 (required=true)**: 빈이 필수로 주입되어야 하며, 빈이 없으면 예외 발생.

**빈이 없어도 되는 경우(빈 주입이 선택적인 경우)**
**1.required=false** 쓰기:  빈이 없어도 애플리케이션이 정상 작동하며, 빈이 없으면 필드가 null로 설정됨.
```java
@Autowired(required=false)
private MyDependency myDependency; 
```

**2.@Nullable**: 빈이 없을 경우 필드에 null이 할당됨.
```java
import javax.annotation.Nullable;

@Autowired
@Nullable
private MyDependency myDependency;
```
**3.@Optional**: 빈이 없으면 필드가 null로 설정됨.
```java
import org.springframework.beans.factory.annotation.Optional;

@Autowired
@Optional
private MyDependency myDependency; // 빈이 없어도 null로 설정됨

```


### 6. 자동 주입과 명시적 의존 주입 간의 관계

>자동 주입: @Autowired를 사용하여 스프링이 빈을 자동으로 주입한다.
명시적 주입: 수동으로 빈을 설정하거나 XML 파일을 사용하여 빈을 정의한다. 

만약 명시적 주입과 자동 주입 둘다 써져있으면 자동주입으로 처리된다고 한다.
둘다 쓰면 나중에 오류 찾기 힘들다고 한다. 일부 자동주입 적용이 어려운 코드를 제외하고는 그냥 자동주입을 쓰라고 한다.














## Chapter 5 컴포넌트 스캔
> 컴포넌트 스캔은 `@Component` 애노테이션을 사용하여 스프링이 자동으로 빈을 검색하여 등록하는 기능이다.


### 1. @Component 애노테이션으로 스캔 대상 지정

```java
import org.springframework.stereotype.Component;

@Component("muComponent")
public class MyComponent {
    
}
```
> @Component를 붙인 클래스는 스캔 대상이 된다. @Component 애노테이션에 값을 주면 그 값을 빈 이름으로 사용한다.

### 2. @ComponentScan 애노테이션으로 스캔 설정

```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "spring") 
public class AppConfig {
  
}
```
> @Component 애노테이션이 붙인 클래스를 스캔해서 스프링 빈으로 등록하려면 설정클래스에 @ComponentScan 애노테이션을 적용해야한다.
basePackages 속성으로 지정한 패키지 내의 빈들만 스캔하여 등록한다.


### 3. 예제 실행 


### 4. 스캔 대상에서 제외하거나 포함하기
> excludeFilters - 특정 클래스를 스캔에서 제외함.
 FilterType -
ANNOTATION: 특정 애노테이션이 붙은 클래스를 스캔에서 제외함
ASSIGNABLE_TYPE: 특정 타입을 스캔에서 제외함

예시로 보자
1.애노테이션 필터
```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

@Configuration
@ComponentScan(
    basePackages = "com.example",
    excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = CustomAnnotation.class)
)
public class AppConfig {
}
```
2.타입 필터
```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

@Configuration
@ComponentScan(
    basePackages = "com.example",
    excludeFilters = @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = MyService.class)
)
public class AppConfig {
}
```
3.두 개 이상의 필터 설정
```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

@Configuration
@ComponentScan(
    basePackages = "com.example",
    excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = CustomAnnotation.class),
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = MyService.class)
    }
)
public class AppConfig {
}
```


> 4.1 기본 스캔 대상
 @Component, @Service, @Repository, @Controller 등 스프링의 스테레오타입 애노테이션이 붙은 클래스들



### 5. 컴포넌트 스캔에 따른 충돌 정리
> 5.1 빈 이름 충돌
같은 이름의 빈이 여러 개 등록되면 충돌이 발생할 수 있다. 이 경우 빈 이름을 명확히 하여 문제를 해결한다.
예시: 두 개의 @Component 빈이 같은 이름을 가진 경우, 스프링은 어떤 빈을 사용할지 결정하지 못할 수 있다. 빈 이름을 명시적으로 설정하여 문제를 해결한다.

> 5.2 수동 등록한 빈과 충돌
 XML 설정 파일에서 수동 등록한 빈과 @Component로 등록된 빈이 충돌할 수 있다. 이 경우 빈의 정의를 명확히 하여 충돌을 방지한다.

예시: XML 설정 파일에서 등록한 빈과 @Component로 등록된 빈 이름이 같은 경우 수동 등록한 빈이 우선되어 하나만 존재한다.
