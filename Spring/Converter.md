# Convertor

[목록으로 돌아가기](/README.md)

## 개요

게시글에 좋아요를 등록하는 기능을 구현하던 중이었다. `@PathVariable`로 `Enum` 타입을 패러미터로 받는 메서드가 있었는데, 막상 API를 호출하면 에러가 발생했다.

```Json
"status": 400

"error": "Bad Request",
```

`org.springframework.web.method.annotation.MethodArgumentTypeMismatchException`:

Failed to convert value of type '`java.lang.String`' to required type '`com.innovation.myblog.models.TargetType`'; nested exception is `org.springframework.core.convert`.

`ConversionFailedException`:
Failed to convert from type [`java.lang.String`] to type [`@org.springframework.web.bind.annotation.PathVariable com.innovation.myblog.models.TargetType`] for value '`post`'; nested exception is java.lang.IllegalArgumentException: No enum constant com.innovation.myblog.models.TargetType.post

## 문제 탐색

사실 원인은 간단했다. 

`Request`로 받은 `@PathVariable`은 위에 표시된 '`post`'였다.

###### ***api주소: 로컬/api/likes/post/1?param=like***

그리고 `@PathVariable`로 받은 Enum 타입, `TargetType`은 이렇게 생겼다.

```java
@Getter
public enum TargetType {
    POST("POST"),
    COMMENT("COMMENT");

    TargetType(String value) {
        this.value = value;
    }

    private final String value;
}
```

그렇다. 대소문자의 문제였다. `post`와 `POST`가 서로 달라서 `ConversionFailedException`이 뜬 것이다.

실제로 `Request`를 보낼 때 `로컬/api/likes/post/1?param=like` 대신 `로컬/api/likes/POST/1?param=like`로 보내면 잘 작동한다.

## 소문자로 보내려면?

사실 처음 이 문제를 맞닥뜨려서 검색을 했을 때는 대소문자의 문제가 아니라, 그냥 `TargetType`이 내가 만든 타입이라 스프링에서 컨버트를 못 해주는 줄 알았다.

(근데 냉정히 생각해 보면, `Entity`나 `DTO` 클래스들도 잘만 해주는데 `Enum`이라고 못 해줄 이유가 없지 않은가?)

그래서 찾은 해결책은, 내가 임의로 구현한 `Converter<S, T>` 를 구현해서 `@Configuration` 클래스에 등록하는 것이었다.

```Java
public class StringToTargetTypeConverter implements Converter<String, TargetType> {
    @Override
    public TargetType convert(String source) {
        return TargetType.valueOf(source.toUpperCase());
    }
}
```

우선 어딘가에 이런 컨버터 클래스를 작성하고, `convert`메서드를 오버라이드 한다. 내용은 간단하다.

`return TargetType.valueOf(source.toUpperCase())`

입력값으로 받은 `String`을 대문자로 바꾼 뒤, `TargetType`으로 바꿔서 반환하는 메서드다.

```Java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

    // 구현한 StringToTargetTypeConverter를 스프링에 사용하겠다고 알리는 역할
    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new StringToTargetTypeConverter());
    }
}
```

그리고 `WebMvcConfigurer`를 상속 받는 `@Configuration` 클래스를 하나 작성한다.

이 밑에 `addFormatters(FormatterRegistry registry)` 메서드를 오버라이드하고, 내가 만든 커스텀 컨버터 클래스를 등록한다.

이제부터는 별 다른 조치가 필요 없이, 알아서 스프링이 post를 POST로 바꿔서 읽는다.

## 결론

스프링에 대해 배울수록, 내가 커스터마이징 가능한 부분이 정말 많은 것 같다.
