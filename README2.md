![image](https://github.com/user-attachments/assets/f3b74ac4-b2bb-4b44-b15f-5f70deb39dee)# Chapter 10: 스프링 MVC 프레임워크 동작 방식

## 1.스프링 MVC 핵심 구성 요소와 각 요소 간의 관계

![image](https://github.com/user-attachments/assets/713eb420-4e18-4135-848f-e885e11418fb)


- **<<spring bean>>**: 스프링으로 등록해야 하는 요소
- **색 칠해져있는 구성 요소**: 개발자가 직접 구현해야 하는 요소

### 동작 과정

1. **요청 수신**
   - 중앙에 있는 `DispatcherServlet`은 모든 연결을 담당
   - 웹 브라우저로부터 요청이 들어오면 `DispatcherServlet`은 그 요청을 처리하기 위한 컨트롤러 객체를 검색
   - 직접 검색하지는 않고 `HandlerMapping`이라는 빈 객체에 검색을 요청 (2번 과정)

2. **컨트롤러 검색**
   - `HandlerMapping`은 클라이언트 요청 경로를 이용해서 처리할 컨트롤러 빈 객체를 찾아 `DispatcherServlet`에 전달
   - 웹 요청 경로가 `/hello`면 등록된 컨트롤러 빈 중에서 `/hello` 요청 경로를 처리할 컨트롤러 리턴 (`@WebServlet("/hello")` 애노테이션이 붙은 것)

3. **요청 처리**
   - `DispatcherServlet`이 컨트롤러 객체를 받았다고 해서 바로 컨트롤러 객체의 메서드를 쓸 수 있는 건 아님
   - `DispatcherServlet`이 `HandlerAdapter`의 빈에게 요청, 처리를 위임
   - `HandlerAdapter`가 컨트롤러의 알맞은 메서드를 호출해서 요청을 처리하고 그 결과를 `ModelAndView`라는 객체로 변환해서 `DispatcherServlet`에 리턴 (3~6번 과정)

4. **뷰 선택**
   - `HandlerAdapter`로부터 `ModelAndView`를 받은 `DispatcherServlet`은 결과를 보여줄 뷰를 찾기 위해 `ViewResolver`를 사용 (7번 과정)

5. **뷰 객체 생성**
   - `ModelAndView`는 컨트롤러가 리턴한 뷰 이름을 담고 있음
   - `ViewResolver`가 이 뷰 이름에 해당하는 뷰 객체를 찾거나 생성해서 리턴
   - 응답을 생성하기 위해 JSP를 사용하는 `ViewResolver`는 매번 새로운 뷰 객체를 생성해서 `DispatcherServlet`에 리턴

6. **응답 생성**
   - `DispatcherServlet`은 리턴 받은 뷰 객체에게 응답 결과 생성을 요청 (8번 과정)
   - JSP를 사용하는 경우 뷰 객체는 JSP를 실행함으로써 웹 브라우저에 전송할 응답 결과를 생성








## 1.1 컨트롤러와 핸들러

### 왜 컨트롤러를 찾아주는 객체는 'ControllerMapping'이 아니고 'HandlerMapping' 일까?
> @Controller애노테이션을 붙인 클래스를 이용해 클라이언트 요청을 처리할 수도 있지만
원하면 다른 클래스 이용해서 처리할 수도 있다 
예시: 스프링이 클라이언트의 요청을 처리하기 위해 제공하는 타입 중 HttpRequestHandler 

-> 컨트롤러라는 특정용어 쓰는 대신 스프링MVC가 지원하는 다양한 요청처리객체를 표현함

### `HandlerAdapter`를 거치는 이유
DispatcherServlet은 핸들러 객체의 실제 타입에 상관없이 실행 결과를 ModelAndView라는 타입으로 받을 수만 있으면 되는데 
이때 핸들러의 실제 구현 타입에 따라 실행 결과를 ModelAndView 로 리턴해주는 객체도 있고 아닌 객체도 있다.
-> 핸들러의 처리결과를 ModelAndView 로 변환해는 객체가 필요하다


### + 
핸들러의 객체 실제 타입마다 그에 맞는 HandlerMapping이랑 HandlerAdapter를 써야함. 
맞는 HandlerMapping이랑 HandlerAdapter를 스프링 빈으로 등록해아함 -> 뒤에 설명 나온다











## 2.DispatcherServlet과 스프링 컨테이너


![image](https://github.com/user-attachments/assets/2d7d4b84-53b2-400e-86f6-71bcd4cb6c2b)


DispatcherServlet가 초기화 될때마다,설정 파일에 따라 새로운 WebApplicationContext를 생성한다. 웹 요청을 처리하기 위해 필요한 빈들을 포함하고 있어야하기 때문에 HandlerMapping, HandlerAdapter,Controller,ViewResolver 빈에 대한 정의가 있어야한다

### 메인 컨테이너와 서브 컨테이너의 차이점

- **생성 시점**:

메인 컨테이너(ApplicationContext)는 애플리케이션 시작 시 한 번만 생성
서브 컨테이너(WebApplicationContext)는 각 DispatcherServlet이 초기화될 때 생성

- **용도와 범위**:

메인 컨테이너는 전체 애플리케이션에서 공통적으로 필요한 빈들을 관리
서브 컨테이너는 특정 DispatcherServlet에 대한 요청 처리를 담당하며, 웹 요청과 관련된 빈들을 관리

- **빈 관리**:

메인 컨테이너에는 데이터베이스, 보안, 트랜잭션 관리 등 애플리케이션 전반에서 필요한 빈들이 포함.
서브 컨테이너에는 요청을 처리하는 데 필요한 웹 계층의 빈들이 포함









## 3.@Controller를 위한 HandlerMapping과 HandlerAdapter
> 핸들러의 객체 실제 타입마다 그에 맞는 HandlerMapping이랑 HandlerAdapter를 써야함
 해당하는 HandlerMapping이랑 HandlerAdapter를 스프링 빈으로 등록해아함 
 직접 작성은 복잡하므로
 @EnableWebMvc 애노테이션 쓰면 매우 다양한 스프링 빈 설정을 추가해준다
 이 때 빈으로 추가해주는 클래스 중에서 @controller타입의 핸들러를 처리하기 위한 클래스인 RequestMappingHandlerMapping과 RequestMappingHandlerAdapter도 존재한다. 


 예시와 요청 처리 흐름 

 ```java
 @Controller
public class HelloController {

    @RequestMapping("/hello")
    public String sayHello(Model model) {
        model.addAttribute("message", "Hello, World!");
        return "helloView"; 
    }
}

```
**요청 매핑**:
RequestMappingHandlerMapping이 URL /hello를 HelloController의 sayHello 메서드에 매핑

**핸들러 호출**:
RequestMappingHandlerAdapter가 sayHello 메서드를 호출
Model 객체에 데이터("message": "Hello, World!")를 추가

**뷰 반환**:
sayHello 메서드는 "helloView"라는 뷰 이름을 반환
RequestMappingHandlerAdapter는 이 뷰 이름과 모델 데이터를 포함하여 ModelAndView 객체를 생성
ViewResolver가 "helloView"를 실제 뷰 파일로 변환하여 사용자에게 응답






## 4.WebMvcConfigurer 인터페이스와 설정

```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/view/");
        resolver.setSuffix(".jsp");
        registry.viewResolver(resolver);
    }
}
```
> 여기서 @EnableWebMvc 애노테이션을 사용하면 @Controller 애노테이션을 붙인 컨트롤러를 위한 설정을 생성한다 
-> @Controller 애노테이션이 붙은 클래스들이 제대로 동작할 수 있게 필요한 설정들을 자동으로 해준다.


> 기본 설정만으로 부족한 부분은 @EnableWebMvc 애노테이션을 사용해 WebMvcConfigurer타입의 빈을 이용해 mvc 설정을 추가로 생성한다. 
예시:
MvcConfig 클래스는 WebMvcConfigurer 인터페이스를 구현하여 추가적인 설정을 제공한다:

1. configureDefaultServletHandling 메서드: 기본 서블릿 처리를 활성화한다. 이로 인해 정적 리소스(예: 이미지, CSS 파일 등)가 올바르게 제공될 수 있다.

2. configureViewResolvers 메서드: 뷰 리졸버를 설정하여 JSP 파일들을 /WEB-INF/view/ 폴더 아래에서 찾고 .jsp 확장자로 된 파일을 뷰로 사용할 수 있게 한다.



> 정리: @EnableWebMvc를 사용하면 스프링이 웹 애플리케이션을 위한 기본 설정을 해주고, WebMvcConfigurer를 이용해서 이 기본 설정을 너의 애플리케이션에 맞게 추가 또는 변경할 수 있다 







## 5.JSP를 위한 ViewResolver


### ViewResolver 설정:

```java
@Bean
public ViewResolver viewResolver() {
    InternalResourceViewResolver vr = new InternalResourceViewResolver();
    vr.setPrefix("/WEB-INF/view/");
    vr.setSuffix(".jsp");
    return vr;
}
```


InternalResourceViewResolver는 뷰 이름에 접두사(prefix)와 접미사(suffix)를 추가하여 경로를 생성.
예: 뷰 이름이 "welcome"이면 "WEB-INF/view/welcome.jsp" 경로를 사용.

이 경로에 있는 JSP 파일을 InternalResourceView 객체로 리턴.



### DispatcherServlet 동작 과정:

1. 컨트롤러 실행:

컨트롤러가 요청을 처리하고 뷰 이름과 모델 데이터를 반환.
```java

@Controller
public class WelcomeController {
    @RequestMapping("/welcome")
    public String welcome(Model model, @RequestParam(value = "user", required = false) String user) {
        model.addAttribute("message", "Welcome, " + user + "!");
        return "welcome";
    }
}
```

2. ModelAndView 처리:
DispatcherServlet은 HandlerAdapter를 통해 ModelAndView 객체를 받음.
ModelAndView 객체에는 뷰 이름과 모델 데이터가 포함됨.

3. ViewResolver 호출:
DispatcherServlet은 ViewResolver에게 뷰 이름에 해당하는 View 객체를 요청.
InternalResourceViewResolver는 "prefix+뷰이름+suffix" 경로의 JSP 파일을 사용하는 InternalResourceView 객체를 리턴.
예: 뷰 이름 "welcome" -> "WEB-INF/view/welcome.jsp" 경로의 JSP 파일.

4. 응답 생성:
DispatcherServlet은 InternalResourceView 객체에 응답 생성을 요청.
InternalResourceView 객체는 JSP 코드를 실행하여 응답 결과를 생성.
모델에 담긴 데이터(예: "message" 속성)는 View 객체에 Map 형식으로 전달되어 JSP에서 사용됨.

5. 결과:
JSP는 전달받은 모델 데이터를 사용하여 알맞은 응답 결과를 생성하고 클라이언트에 전송.
예: "Welcome, user!" 메시지가 포함된 JSP 페이지가 렌더링되어 클라이언트에게 전송.

























## 6.디폴트 핸들러와 HandlerMapping의 우선순위

### 디폴트 핸들러란?

디폴트 핸들러는 어떠한 HandlerMapping에도 매핑되지 않는 요청을 처리하는 핸들러이다.
특정 URL 패턴을 처리하지 않는 나머지 모든 요청을 처리할 수 있다.
따라서 HandlerMapping의 우선순위가 디폴트 핸들러보다 높다.

### DispatcherServlet의 동작 과정:

1. 요청 수신:
웹 브라우저로부터 요청이 들어오면 DispatcherServlet이 이를 받는다.

2. 핸들러 검색:
DispatcherServlet은 요청 URL을 기반으로 등록된 HandlerMapping 빈을 사용하여 적절한 핸들러를 찾는다.

3. 핸들러 실행:
적절한 핸들러가 발견되면, DispatcherServlet은 해당 핸들러를 실행한다.

4. 디폴트 핸들러 실행:
만약 어떤 HandlerMapping에서도 핸들러를 찾지 못한 경우, 디폴트 핸들러가 요청을 처리한다.


-> 이렇게 DispatcherServlet은 우선순위에 따라 핸들러를 검색하고 실행하여, 웹 애플리케이션의 요청을 적절히 처리한다.




















# Chapter 11: MVC 1 : 요청 매핑, 커맨드 객체, 리다이렉트, 폼 태그, 모델

## 1.요청 매핑 어노테이션을 이용한 경로 매핑

> 웹 어플리케이션 개발 하는 것은 다음 코드를 작성하는 것이다.
1. 특정 요청 URL을 처리할 코드 작성
2. 처리 결과를 HTML 등의 형식으로 응답하는 코드 작성


첫 번째 작업은 `@Controller` 애노테이션을 사용한 컨트롤러 클래스를 이용해서 구현한다.

```java
@Controller
public class YewonController {
    @GetMapping("/yewon")
    public String yewon(Model model, @RequestParam(value = "name", required = false) String name) {
        model.addAttribute("greeting", "Hello " + name);
        return "yewon";
    }
}
```

컨트롤러 클래스는 요청 매핑 애노테이션(`@RequestMapping`, `@GetMapping` 등)을 사용하여 메서드가 처리할 요청 경로를 지정한다. 위 코드의 `YewonController` 클래스는 `@GetMapping`을 사용하여 "/yewon" 요청 경로를 `yewon()` 메서드가 처리하도록 설정하고 있다.

### 여러 요청 매핑 메서드

컨트롤러 클래스에서 여러 요청 경로를 처리하는 메서드를 정의할 수 있다. 예를 들어, 하나의 컨트롤러 클래스를 만들고 여러 메서드에서 각 요청 경로를 처리할 수 있다:

```java
@Controller
public class MyController {
    @GetMapping("/page1")
    public String handlePage1(Model model) {
        model.addAttribute("message", "This is Page 1");
        return "page1";
    }

    @GetMapping("/page2")
    public String handlePage2(Model model) {
        model.addAttribute("message", "This is Page 2");
        return "page2";
    }
}
```

위 코드는 각각 "/page1"과 "/page2" 경로를 처리하는 두 개의 메서드를 가지고 있다.

### 중복된 경로를 클래스 수준으로 리팩토링

메서드마다 중복되는 경로가 있는 경우, 클래스에 공통 경로를 지정할 수 있다. 이렇게 하면 코드가 더 간결해진다:

```java
@Controller
@RequestMapping("/pages")
public class PagesController {
    @GetMapping("/page1")
    public String handlePage1(Model model) {
        model.addAttribute("message", "This is Page 1");
        return "page1";
    }

    @GetMapping("/page2")
    public String handlePage2(Model model) {
        model.addAttribute("message", "This is Page 2");
        return "page2";
    }
}
```

위 코드에서 `@RequestMapping("/pages")`를 클래스 수준에 적용하여 "/pages" 경로가 모든 메서드에 공통으로 적용되도록 했다. 따라서, 각 메서드는 "/pages/page1"과 "/pages/page2" 경로를 처리하게 된다. 

이렇게 하면 경로가 중복되지 않는다.












## 2. GET과 POST의 구분: @GetMApping, @GetPost

### 요청 방식에 따른 매핑

스프링 MVC에서는 기본 설정으로 `@RequestMapping` 애노테이션을 사용하면 GET, POST 방식을 구분하지 않고 모든 요청을 처리한다. 특정 HTTP 메소드만 처리하려면 `@PostMapping`이나 `@GetMapping` 같은 애노테이션을 사용하여 제한할 수 있다.

POST 방식 요청만 처리하고 싶다면 `@PostMapping`을 사용:

```java
@Controller
public class YewonController {
    
    @PostMapping("/yewon/submit")
    public String handleSubmit() {
        return "yewon/submit";
    }
}
```

 `handleSubmit` 메서드는 POST 방식의 `/yewon/submit` 요청 경로만 처리하며, 같은 요청 경로의 GET 요청은 처리하지 않는다.


특정 경로의 GET 방식 요청만 처리하고 싶다면 `@GetMapping`을 사용:

```java
@Controller
public class YewonController {
    
    @GetMapping("/yewon/form")
    public String showForm() {
        return "yewon/form";
    }
}
```

 `showForm` 메서드는 GET 방식의 `/yewon/form` 요청 경로만 처리한다.






## 3. 요청 파라미터 접근

HTML 폼을 통해 요청 파라미터를 전달하고 이를 컨트롤러에서 처리하는 방법은 두 가지가 있다.

약관 동의 체크박스를 포함한 폼 + 이를 처리하는 코드로 이해해보자

### HTML 폼 예시
```html
<form action="/yewon/step2" method="post">
    <label>
        <input type="checkbox" name="agree" value="true"> 약관 동의
    </label>
    <input type="submit" value="다음 단계">
</form>
```
위 폼은 사용자가 약관에 동의할 경우 값이 "true"인 `agree` 요청 파라미터를 POST 방식으로 전송.

### 방법 1: `HttpServletRequest`를 사용하는 방법
```java
@PostMapping("/yewon/step2")
public String handleStep2(HttpServletRequest request) {
    String agreeParam = request.getParameter("agree");
    if (agreeParam == null || !agreeParam.equals("true")) {
        return "yewon/step1";
    }
    return "yewon/step2";
}
```
- `HttpServletRequest`를 파라미터로 받아 `getParameter()` 메서드로 요청 파라미터 값을 구한다.
- `agree` 파라미터 값이 "true"가 아니면 다시 약관 동의 폼을 보여줌.

### 방법 2: `@RequestParam` 애노테이션을 사용하는 방법
```java
@PostMapping("/yewon/step2")
public String handleStep2(@RequestParam(value="agree", defaultValue="false") Boolean agree) {
    if (!agree) {
        return "yewon/step1";
    }
    return "yewon/step2";
}
```
- `@RequestParam` 애노테이션을 사용하여 요청 파라미터를 직접 메서드 파라미터로 받음.
- `value` 속성으로 요청 파라미터 이름을 지정하고, `defaultValue` 속성으로 파라미터가 없을 때 사용할 기본값을 설정.
- `agree` 파라미터 값이 `true`가 아니면 다시 약관 동의 폼을 보여줌.

### 요약
1. **`HttpServletRequest` 사용**: 
   - 요청 파라미터를 직접 가져와서 처리.
   - 예시: `request.getParameter("agree")`

2. **`@RequestParam` 사용**: 
   - 애노테이션으로 요청 파라미터를 간편하게 받음.
   - 예시: `@RequestParam(value="agree", defaultValue="false") Boolean agree`

두 방법 모두 요청 파라미터를 처리하고, 약관에 동의하지 않았을 경우 다시 동의 폼을 보여주고, 동의했을 경우 다음 단계로 넘어간다.

## 4. 리다이렉트 처리
  
> handleStep2()` 메소드는 POST 요청만 처리합니다. 따라서, 웹브라우저에서 직접 URL을 입력해 GET 요청을 보내면 HTTP 405 - Method Not Allowed 에러가 발생하게 됨.

잘못된 전송 방식으로 요청이 왔을 때 에러 화면보다 알맞은 경로로 리다이렉트 하는 것이 더 좋을 수 있다.
컨트롤러에서 특정 페이지로 리다이렉트 시키는 방법은 
 `"redirect:경로"` 를 뷰 이름으로 리턴하여 처리하는 것이다.

1. **예시 코드**:
   ```java
   @Controller
   public class RegisterController {

       @GetMapping("/register/step2")
       public String handleStep2Get() {
           return "redirect:/register/step1";
       }
   }
   ```

   - `handleStep2Get()` 메소드는 GET 요청을 처리하고, 클라이언트를 `/register/step1`으로 리다이렉트합니다.

### 요약
- **문제**: POST 전용 경로에 GET 요청 시 405 에러 발생.
- **해결**: `"redirect:/경로"`를 사용하여 적절한 페이지로 리다이렉트.














## 5. 커맨드 객체를 이용해 요청 파리미터 사용하기


### 커맨드 객체를 이용한 요청 파라미터 처리

#### 문제
폼에서 `email`, `name`, `password`, `confirmPassword`와 같은 요청 파라미터를 전송할 때, `HttpServletRequest`를 사용하면 코드가 길어지고 관리가 어려워질 수 있음.

#### 해결 방법
스프링 MVC는 요청 파라미터를 커맨드 객체에 자동으로 바인딩할 수 있는 기능을 제공
코드의 복잡성을 줄이고 유지보수성을 향상시킬 수 있음.

#### 커맨드 객체 사용 예시

1. **커맨드 객체 클래스**

```java
@Getter
@Setter
public class RegisterRequest {
    private String email;
    private String name;
    private String password;
    private String confirmPassword;
}
```

`@Getter`와 `@Setter`는 롬복(Lombok) 라이브러리의 어노테이션으로, 자동으로 게터와 세터 메서드를 생성.

1. **컨트롤러 메소드**

```java
@PostMapping("/register/step3")
public String handleStep3(RegisterRequest registerRequest) {
    // 커맨드 객체의 필드 값에 자동으로 요청 파라미터가 바인딩됨
    String email = registerRequest.getEmail();
    String name = registerRequest.getName();
    String password = registerRequest.getPassword();
    String confirmPassword = registerRequest.getConfirmPassword();

    // 추가적인 로직 처리
    if (!password.equals(confirmPassword)) {
        return "registerForm";
    }

    return "registrationSuccess";
}
```

`RegisterRequest` 객체의 필드에 요청 파라미터 값이 자동으로 바인딩됨. 이를 통해 코드가 간결해지고 요청 파라미터의 처리가 더 쉬워짐








## 6. 뷰 JSP 코드에서 커맨드 객체 사용하기 
## 7. @ModelAttribute 어노테이션으로 커맨드 객체 속성 이름 변경

### 커맨드 객체와 뷰에서의 속성 이름 변경

스프링 MVC는 기본적으로 커맨드 객체의 클래스 이름의 첫 글자를 소문자로 바꾼 이름을 뷰에 전달한다. 만약 다른 이름으로 접근하고 싶다면, `@ModelAttribute` 애노테이션을 사용하여 이름을 변경할 수 있다.

**컨트롤러 메소드**

```java
@PostMapping("/login")
public String handleLogin(@ModelAttribute("loginData") LoginForm loginForm) {
    
    String username = loginForm.getUsername();
    String password = loginForm.getPassword();
    
    .
    .
    .
    return "loginSuccess";
}
```

`@ModelAttribute("loginData")`를 사용하여, 뷰에서는 `loginData`라는 이름으로 `LoginForm` 객체에 접근할 수 있다.

#### 요약

- **기본 이름**: 커맨드 객체의 클래스 이름에서 첫 글자를 소문자로 바꾼 이름이 기본으로 사용됨 (예: `LoginForm` → `loginForm`).
- **이름 변경**: `@ModelAttribute("사용자정의이름")`을 사용하여 뷰에서 사용할 이름을 지정할 수 있음 (예: `@ModelAttribute("loginData")`).

이렇게 하면 뷰에서 커맨드 객체에 더 명확하고 일관된 이름으로 접근할 수있게 된다.






















## 8. 커맨드 객체와 스프링 폼 연동


### 커맨드 객체와 스프링 폼 연동

스프링 MVC는 커맨드 객체와 연동하여 폼을 간편하게 처리할 수 있는 커스텀 태그를 제공한다. 주요 태그는 `form:form`과 `form:input` 이다.

1. **폼 코드 (JSP)**

```jsp
<form:form action="step3" modelAttribute="registerRequest">
    <form:input path="email" />
    <form:input path="name" />
    <input type="submit" value="Submit" />
</form:form>
```

- **`action`**: 폼의 제출 경로를 설정.
- **`modelAttribute`**: 커맨드 객체의 이름을 지정함 (예: `registerRequest`).

2. **컨트롤러 메소드**

```java
@Controller
public class RegisterController {

    @PostMapping("/register/step2")
    public String handleStep2(@RequestParam(value="agree", defaultValue="false") Boolean agree, Model model) {
        if (!agree) {
            return "register/step1";
        }
        model.addAttribute("registerRequest", new RegisterRequest());
        return "register/step2";
    }
}
```

 `model.addAttribute("registerRequest", new RegisterRequest());`를 통해 모델에 커맨드 객체를 추가함. -> 이 객체가 폼에서 사용됨



- `form:form`: 폼의 제출 경로와 커맨드 객체의 이름을 지정.
- `modelAttribute`: 폼에서 사용할 커맨드 객체의 이름을 지정.
- 컨트롤러: 모델에 커맨드 객체를 추가하여 폼에 데이터를 제공.

이렇게 하면 폼에서 커맨드 객체의 필드에 접근하고, 데이터를 쉽게 바인딩할 수 있다

## 9. 주요 에러 발생 상황

### 1. 요청 매핑 애노테이션 관련 에러

- **404 에러 (Not Found)**: 
  - 요청 경로를 처리할 컨트롤러가 존재하지 않거나,
  - WebMvcConfigurer를 통한 설정이 부족하거나,
  - 뷰 이름에 해당하는 JSP 파일이 존재하지 않을 때 발생

- **405 에러 (Method Not Allowed)**:
  - 지원하지 않는 HTTP 메서드(예: POST만 처리하는 경로에 GET 요청)를 사용할 때 발생

### 2. @RequestParam 및 커맨드 객체 관련 에러

- **400 에러 (Bad Request)**:
  - 요청 파라미터를 `@RequestParam`의 타입으로 변환할 수 없을 때 발생
  - 커맨드 객체의 프로퍼티와 요청 파라미터의 타입이 맞지 않아 변환 오류가 발생할 때도 발생
  - 예를 들어, `int` 타입의 프로퍼티에 `"aaaa"`라는 문자열을 전달하면 변환 실패로 인해 400 에러가 발생




## 10.  Model을 통해 컨트롤러에서 뷰 데이터 전달

### Model을 통해 컨트롤러에서 뷰 데이터 전달

컨트롤러는 뷰가 화면을 구성하는 데 필요한 데이터를 제공하기 위해 `Model` 또는 `ModelAndView`를 사용.

#### 1. Model 사용

  - `Model` 파라미터를 메소드에 추가하고, `addAttribute()` 메소드를 사용하여 뷰에 데이터를 전달.
  - `addAttribute()`의 첫 번째 파라미터는 데이터의 속성 이름입니다. 이 이름을 뷰에서 사용하여 데이터에 접근.

```java
@Controller
public class HaewonController {

    @GetMapping("/greeting")
    public String showGreeting(Model model) {
        model.addAttribute("message", "Hello, yewon!");
        return "greetingView";
    }
}
```

`model.addAttribute("message", "Hello, yewon!");`를 통해 `message`라는 이름으로 데이터를 뷰에 전달

#### 2. ModelAndView 사용

- **데이터와 뷰 이름을 동시에 설정**:
  - `ModelAndView` 객체를 사용하여 모델 데이터와 뷰 이름을 한 번에 설정할 수 있음.
  - `addObject()` 메소드로 데이터를 추가하고, `setViewName()` 메소드로 뷰 이름을 설정함

**예시**

```java
@Controller
@RequestMapping("/welcome")
public class HaewonController {

    @GetMapping
    public ModelAndView showWelcome() {
        ModelAndView mav = new ModelAndView();
        mav.addObject("message", "Welcome, yewon!");
        mav.setViewName("welcomeView");
        return mav;
    }
}
```

 `mav.addObject("message", "Welcome, yewon!");`로 데이터를 추가하고, `mav.setViewName("welcomeView");`로 뷰 이름을 설정함





- **Model**: 데이터를 뷰에 전달하는 데 사용하며, `addAttribute()` 메소드를 통해 속성 이름과 값을 설정합니다.
- **ModelAndView**: 데이터를 모델에 추가하고, 뷰 이름을 설정하여 결과를 한 번에 처리합니다. `addObject()`와 `setViewName()` 메소드를 사용합니다.
