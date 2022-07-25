# Lombok

[목록으로 돌아가기](../../README.md)

## 개요

Lombok은 자바 라이브러리의 하나로, getter, setter, toString 등의 반복되는 메서드를 자동으로 생성해주는 코드 다이어트 라이브러리이다.

## 사용 예시

```Java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Person extends Timestamped {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    @Column(nullable = false)
    private String name;
    @Column(nullable = false)
    private int age;
    @Column(nullable = false)
    private String job;

    public Person(String name, int age, String job) {
        this.name = name;
        this.age = age;
        this.job = job;
    }
}
```

위 코드에서 `Getter`, `Setter`, 그리고 `NoArgsConstructor` 이 3개의 어노테이션이 바로 Lombok에서 제공하는 것들이다.

`Getter`와 `Setter`는 이름 그대로 클래스의 각 필드에 대해 `Getter`와 `Setter` 함수를 생성해준다.

`NoArgsConstructor`는 인자가 없는 생성자를 자동으로 생성해준다.

만약 Lombok이 없었다면 아래와 같은 코드를 작성해야 했을 것이다.

```Java
@Entity
public class Person extends Timestamped {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    @Column(nullable = false)
    private String name;
    @Column(nullable = false)
    private int age;
    @Column(nullable = false)
    private String job;

    public Person() {}

    public Person(String name, int age, String job) {
        this.name = name;
        this.age = age;
        this.job = job;
    }

    public Long getId() { return id; }
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getJob() { return job; }

    //... (setter는 생략)
}
```

이 경우 추후에 필드를 추가하거나, 삭제할 때마다 `Getter`와 `Setter`를 추가/삭제 해야 하는데, Lombok을 사용하면 모두 자동으로 처리해준다.
