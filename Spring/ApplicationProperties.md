# ApplicationProperties

[목록으로 돌아가기](/README.md)

## 개요

`application.properties`, 혹은 `application.yml` 파일은 스프링 프로젝트의 환경설정 파일이다.

이 파일에는 여러 설정 구문을 추가할 수 있으며, 사용자가 상수로 사용하기 위해 임의의 값을 넣을 수도 있다.

주로 액세스 키 등이 그것인데, 이런 종류의 보안 키는 외부에 노출되선 안 된다.

그래서 `application` 파일 내에 키를 그대로 적어놓았을 경우, github에 푸쉬하거나 할 때 해당 파일을 제외해야 하는 등, 매우 불편한 과정을 필요로 한다.

## 대책

먼저 프로젝트 루트 폴더에 임의의 파일을 생성한다. 파일 이름은 `application-이름.properties` 이렇게 지으면 된다.

`이름` 부분은 원하는대로 변경 가능하다. 그리고 원본 `application` 파일 내에 이런 옵션을 추가한다.

```yaml
spring:
  profiles:
    include: 이름
```

이렇게 해두면, 아래와 같은 방식으로 앞서 작성한 `application-이름.properties` 파일 내의 값을 참조할 수 있게 된다.

```Java
@Value("${jwt.secret-key}")
String secretKey
```

그 후 실제 값이 담긴 `application-이름.properties` 파일을 `.gitignore`파일에 추가한다.

이러면 중요한 값들을 외부에 노출하지 않고 안전하게 보관할 수 있다.
