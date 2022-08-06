# 불변객체에 json파일 매핑하기

[목록으로 돌아가기](/README.md)

## 개요

변경이 없는 변수는 가급적 final 선언을 해서 사용하는 편이다. 그래서 DTO 클래스의 멤버변수도 가급적 final 선언을 해서 사용했다.

근데 그렇게 했더니, 컨트롤러에서 `@ResponseBody` 를 단 매개변수로 받아올 때 에러가 발생했다.

## 문제 탐색

검색해 보니 원인은 간단했다. Jackson이 오브젝트 매핑을 할 때 객체의 빈 생성자를 필요로 하는데, 불변객체에는 빈 생성자가 없으니 매핑을 하지 못하는 것이었다.

2가지 해결책이 있었다.

### 해결책 1

```Java
// User.java
@Getter
public class User {
    @JsonProperty("user_name")
    private final String userName;
    private final int age;

    @JsonCreator // 생성자가 하나면 생략 가능
    public User(@JsonProperty("user_name") String userName,
                @JsonProperty("age") int age) {

                this.userName = userName;
                this.age = age;
    }
}
```

먼저 파싱에 필요한, 즉 json 내부 데이터와 매핑할 필드의 값을 채워주는 생성자를 만들고, 위에 `@JsonCreator`를 달아준다.

각각의 파라미터에는 `@JsonProperty("json의 키값")`을 붙여준다.

이후 매칭되는 json의 키값과 문자형태가 다른 필드에 생성자와 똑같이 `@JsonProperty("json의 키값")` 어노테이션을 달아준다.

위 예제코드에서 필드 `age`는 json에서도 `age`로 같기 때문에 `@JsonProperty`가 필요 없지만, `useName`은 json의 키값 `user_name`과 다르기 때문에 어노테이션에 명시를 해줘야 매핑이 된다.

### 해결책 2

두 번째 해결책은 Lombok을 활용한 방법이다. 이 방법은 처음에 설정이 좀 필요하지만, 해놓으면 `@JsonProperty` 같은 걸 사용할 필요가 없다.

그러려면 3개의 준비 단계를 거쳐야 한다.

#### Step 1

```Gradle
dependencies {
    implementation 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    implementation "com.fasterxml.jackson.datatype:jackson-datatype-jdk8"
    implementation "com.fasterxml.jackson.datatype:jackson-datatype-jsr310"
    implementation "com.fasterxml.jackson.module:jackson-module-jackson-module-parameter-names"
}
```

먼저 `build.gradle`에 종속성을 추가한다.

#### Step 2

프로젝트 루트에 `lombok.config` 파일을 생성한 뒤, 아래와 같이 입력한다.

```Gradle
lombok.addLombokGeneratedAnnotation = true
lombok.anyConstructor.addConstructorProperties = true
```

#### Step 3

마지막으로 클래스를 하나 만든다.

```Java
public class Config {
    @Bean
    ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.findAndRegisterModules(); //Registers all modules on classpath
        return mapper;
    }
}
```

#### 결과

```Java
// User.java
@Getter
@RequiredArgsConstructor
public class User {
    private final String userName;
    private final int age;
}
```

이제는 그냥 이렇게 사용할 수 있다. 이 객체를 컨트롤러에서 그냥 받아오면, Jackson이 알아서 json을 파싱해서 넣어준다.

당연히, json의 키값과 변수명의 대소문자가 일치해야 한다.
