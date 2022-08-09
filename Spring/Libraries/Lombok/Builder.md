# @Builder

[목록으로 돌아가기](/README.md)

## 개요

`Lombok`의 `@Builder` 어노테이션은 `Builder Pattern`을 손쉽게 적용할 수 있게 도와준다.

```Java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED) // 기본 생성자 제한
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @NotNull
    @Column(unique = true)
    private String username;

    @NotNull
    private String password;

    @Builder
    protected User(String username, String password) {
        Assert.hasText(username, "username이 공백이거나 null입니다.");
        Assert.hasText(password, "password가 공백이거나 null입니다.");

        this.username = username;
        this.password = password;
    }
}
```

위와 같이 `Entity` 클래스를 만든 다음,

```Java
User user = User.Builder()
                    .username(username)
                    .password(encodedPassword)
                    .build();
```

이런 식으로 생성한다.

이렇게 하면 얻는 장점이 몇 가지 있다.

* 객체 생성자에 필요한 매개변수와 인자의 관계가 명확하게 보인다.

  ```Java
  User user1 = new User("username", "password");
  User user2 = new User("password", "username");
  ```

  딱 봐도 user2는 인자 순서가 잘못됐다. 하지만 컴파일러는 아무런 경고를 하지 않는다.

  컴파일러 입장에서는 둘 다 String으로 동일한 자료형이기 때문이다.

  ```Java
  User user = User.Builder()
                      .password(password)
                      .username(username)
                      .build();
  ```

  `Builder`를 사용하면 인자의 순서가 바뀌어도 문제가 없다.

* 불안전한 생성을 차단하고, 불변 객체를 생성할 수 있다.

  맨 위의 `User` 클래스를 보면 `@NoArgsConstructor(access = AccessLevel.PROTECTED)`이 붙어있는 걸 볼 수 있다.

  매개 변수가 없는 생성자를 만들고, 접근 제어자를 Protected로 설정한다는 뜻이다.

  ```Java
  User user = new User();
  ```

  이런 식의 생성을 컴파일러 단계에서 차단할 수 있다.

  그리고 필요한 생성자를 직접 정의한 뒤, 그 위에 `@Builder` 어노테이션을 달아주면 된다.

  ```Java
  @Builder
    protected User(String username, String password) {
        Assert.hasText(username, "username이 공백이거나 null입니다.");
        Assert.hasText(password, "password가 공백이거나 null입니다.");

        this.username = username;
        this.password = password;
    }
  ```

  이렇게 만들면 좋은 이유가 2가지 있다.

  * `Builder`없이 객체를 생성하는 것을 완전 차단

    ```Java
    User user = new User();
    User user = new User("username", "password");
    ```

    위 2가지 생성 방식 모두 불가능해진다. 생성자의 접근제어자가 Protected이기 때문이다.

  * 생성자 내부에서 인자 검증 가능

    `Assert`를 사용해 생성 시에 받은 값이 null은 아닌지, 공백은 아닌지, 등 각종 검사를 수행하여 생성된 객체의 안전성을 확보할 수 있다.

## 결론

프로그래밍에 대해 공부할수록, 오류가 발생하더라도 개발자가 예측 가능하고 통제 가능한 오류를 발생시키는 것이 매우 중요하다는 것이 체감된다.

굳이 복잡하게 생성자의 접근권한을 틀어막고, Builder를 만들고, 불변 객체를 만들고 하는 이 모든 행위들이 결국은 미래의 시간을 절약하기 위한 것이다.
