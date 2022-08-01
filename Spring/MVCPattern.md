# MVC 패턴

[목록으로 돌아가기](/README.md)

## 개요

**MVC 패턴**이란 소프트웨어 디자인 패턴의 일종으로, 하나의 애플리케이션을 `Model`, `View`, `Controller`로 구분하여 제작하는 것이다.

모델은 애플리케이션의 정보(주고 받는 데이터)를 나타내며, 뷰는 사용자(클라이언트)가 보게 될 요소들을 나타내고, 컨트롤러는 모델을 조작하거나, 그 모델로 뷰를 변경하는 등 중간관리자의 역할을 한다.

## Spring에서의 MVC 패턴

![Spring MVC](/images/spring-mvc.png)

스프링에서의 MVC 패턴은 대략 이런 구조로 되어있다.

사용자로부터 요청이 들어오면, 그것을 `DispathcerServlet`이라 불리는 `Front Controller`가 먼저 받아서 처리한다.

`Front Controller`의 역할은 사용자의 요청을 가장 먼저 받고, 그것을 분석하여 적절한 세부 컨트롤러에게 작업을 위임하는 것이다.

```Java
@Controller
public class HelloController {

    @GetMapping("hello")
    public String hello() {
        return "hello";
    }
}

@Controller
public class GoodbyeController {

    @GetMapping("goodbye")
    public String goodbye() {
        return "goodbye";
    }
}
```

이렇게 여러 컨트롤러가 있을 때, 스프링 프레임워크 내부의 `DispathcerServlet`은 사용자의 요청을 분석해서, `HelloController`와 `GoodbyeController` 중 어느 쪽에 처리를 맡길지 결정한다.

이후 해당 컨트롤러가 요청을 수행하고 나면 최종적으로 그 결과물을 View에서 응답을 사용자에게 돌려준다. 이 때 응답의 형태는 html 페이지일 수도, 단순한 문자열(혹은 JSON 형태)일 수도 있다.
