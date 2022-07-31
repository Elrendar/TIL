# 2022-07-31 WIL

[목록으로 돌아가기](/README.md)

이번주는 거의 스프링 위주로 공부한 것 같다. 여기에 약간의 AWS와 도커 맛보기...

###### *AWS와 도커에 관한 이야기들은 TIL에 따로 항목을 작성해 두었다.*

---

## 요약

지난 주에는 Controller, Service, Repository와 REST API를 사용하는 기초적인 방법에 대해 배웠다면, 이번 주는 스프링에 대해 조금 더 배운 것 같다.

오늘은 몇몇 중요한 개념들에 대해 간단히 정리해 보겠다.

## 스프링, Inversion of Control과 Dependency Injection

IoC란 Inversion of Control의 약자로, 객체의 생명주기 관리를 제 3자(여기서는 스프링 프레임워크)에게 위임하는 프로그래밍 모델이다.

IoC는 스프링에서 처음 만든 개념도 아니고 스프링만의 특징도 아니지만, 스프링이 이를 활용해 다른 프레임워크들과 차별화돼서 제공하는 기능이 바로 Dependency Injection, 즉 의존성 주입이다.

DI는 IoC 프로그래밍 모델을 구현하는 방법 중 하나로, 스프링은 DI라는 구체적인 방법을 통해 IoC를 구현했다고 볼 수 있다.

하나씩 살펴보자.

* ### 의존성(Dependency)이란 무엇인가?

IoC와 DI얘기를 하기 전에, 의존성부터 정리하고 가야 할 것 같다. 결국 이 모든 것들이 객체간의 의존성 때문에 발생하는 문제들을 해결하기 위해 만들어진 것이기 때문이다.

프로그래밍에서 의존한다는 말은, 어떤 객체 간에 레퍼런스 참조가 되어있다는 뜻이다. A가 B를 의존한다는 말은, B 객체에 변화가 있을 때 이를 의존하는 A 객체에도 그 영향이 간다는 의미이다.

```Java
public class A {
    private B b = new B();

    public void foo() {
        b.bar();
    }
}
```

위와 같은 코드가 있다고 가정했을 때, 만약 B 클래스의 bar()가 변경된다면? A 객체의 foo() 역시 변경된 것이나 다름 없다.

그렇다면, 의존성 주입이란 무엇인가?

* ### 의존성 주입(Dependency Injection)

의존성 주입이란, 의존성을 외부에서 주입해주는 걸 의미한다. 위 샘플코드에서 A 객체는 자기가 의존하는 B 객체를 내부에서 직접 생성해서 사용했다. 이것은 주입이 아니다.

```Java
public class A {
    private B b;

    public A(B b) {
      this.b = b;
    }

    public void methodA() {
        b.methodB();
    }
```

아까와 거의 비슷하다. 하지만 이번엔 B 객체를 내부에서 생성하지 않고, 생성자의 인자로 주입 받았다.

* ### 왜 필요한가?

의존성 주입을 사용하는 가장 큰 이유는 객체간의 의존도를 줄이고, 변경과 재사용을 용이하게 하기 위해서이다.

```Java
public inteface I {
    void foo();
}

public class A implements I {
    @Override
    public void foo() {
        System.out.println("foo() of class A");
    }
}

public class B implements I {
    @Override
    public void foo() {
        System.out.println("foo() of class B");
    }
}

public class C {
    private final I i;

    public C() {
      i = new A();
    }

    public void bar() {
      i.foo();
    }
}
```

foo()라는 메서드를 가진 인터페이스 I와, 이를 구현한 클래스 A, B가 있다고 치자.

그리고 이러한 인터페이스를 의존하는 클래스 C가 있다. C는 생성자에서 A의 인스턴스를 생성하고, 인터페이스 i를 통해 A의 foo() 메서드를 호출한다.

그런데 갑자기 C가 바뀌어야 할 일이 생겼다. A객체의 foo() 메서드가 아닌, B 객체의 foo() 메서드를 필요로 하게 된 것이다.

```Java
public class C {
    private final I i;

    public C() {
      i = new B();
    }

    public void bar() {
      i.foo();
    }
}
```

코드를 이렇게 바꿔줘야 할 것이다. `new A()`가 `new B()`로 바뀌었다. 이렇게 보면 별 거 아닌 것처럼 보인다.

하지만 코드 규모가 커질수록 이런 변경조차 쉽지 않을 뿐더러, 어떤 예상치 못한 문제를 발생시킬지 예측하기 어렵다.

여기서 필요한 것이 바로 의존성 주입이다.

```Java
class C {
    private final I i;

    public C(I i) {
      this.i = i;
    }

    public void bar() {
      i.foo();
    }
}

public static void main(String[] args) {
    // 클래스 외부에서 의존 객체의 인스턴스를 생성해서 주입
    C c = new C(new A());
    c.bar();
}
```

클래스 C를 이렇게 바꾸면 C의 클래스 A, B에 대한 의존도가 감소한다. C는 심지어 A와 B의 존재조차 모른다. 그저 그들이 구현한 인터페이스 I만 알고 사용할 뿐이다.

이렇게 만들면 C는 주입 받은 i가 A의 인스턴스인지, B의 인스턴스인지 신경 쓸 필요가 없다. 앞서 말한 변경 역시 주입하는 외부에서만 신경 써주면 된다. C는 건드릴 필요가 없어지는 것이다.

스프링 프레임워크는 제어의 역전을 통해, 이 DI를 더 사용하기 쉽게 구현했다.

* ### 제어의 역전(Inversion of Control)

```Java
public class C {
    private final I i;

    public C() {
      i = new A();
    }

    // bar()는 생략
}
```

이는 앞서 보여준, 개발자가 직접 A 클래스의 인스턴스를 생성하는 방식이다. 이를 외부에서 의존성을 주입하는 방식으로 바꾸는 방법 또한 위에 적어두었다.

하지만 스프링에서는 더 간편하다.

```Java
@Controller
public class C {
    private final I i;

    public C(I i) {
      this.i = i;
    }

    // bar()는 생략
}
```

이는 스프링 어노테이션을 사용한 방식이다. 이거만 보면 달라진 게 없어 보이겠지만, 아주 큰 차이가 존재한다.

먼저, 외부에서 `new A()`와 같은 식으로 A(혹은 B)의 인스턴스를 생성해서 주입해줄 필요가 없다.

심지어 C도 생성할 필요가 없다. `@Controller` 어노테이션이 C를 `Bean`으로 등록해주기 때문이다.

* ### Bean이란 무엇인가?

`Bean`이란 쉽게 말해서 스프링이 관리해주는 객체라고 볼 수 있다. `@Controller`, `@Service`, `@Component` 등의 어노테이션이 붙은 클래스들은 자동으로 `Bean`으로 등록되며, 스프링 IoC컨테이너가 생명주기를 대신 관리해준다.

다시 위의 예시로 돌아가보자.

```Java
@Controller
public class C {
    private final I i;

    public C(I i) {
      this.i = i;
    }

    // bar()는 생략
}
```

만약 I라는 인터페이스가 스프링 컨테이너의 관리를 받는 `Bean`이라면, 개발자가 직접 `new A()`와 같은 방식으로 인스턴스를 생성해서 주입하지 않아도 스프링이 다 알아서 해준다.

이것이 바로 제어의 역전이다.

```Java
@RequiredArgsConstructor
@Controller
public class ㅊ {
    private final I i;

    // bar()는 생략
}
```

참고로 `@RequiredArgsConstructor` 어노테이션을 사용하면 생성자마저 생략 가능하다.
