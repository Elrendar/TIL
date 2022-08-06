# Jackson

[목록으로 돌아가기](/README.md)

## 개요

Jackson은 자바 객체와 json 간의 직렬화, 역직렬화를 도와주는 자바 라이브러리이다.

json은 클라이언트와 서버 간의 통신에서 자주 쓰이는 포맷이다.

하지만 라이브러리의 도움 없이 하드코드의 형태로 json을 작성하는 것은 사실상 불가능하다. 너무 번거롭기 때문이다.

그래서인지 몰라도 Jackson 라이브러리는 스프링부트 스타터 웹 안에 기본 포함되어 있다.

## 사용법

기초적인 사용법은 그냥 `@RestController`에서 자바 객체를 반환하는 것이다.

```Java
@PostMapping("/{type}/{typeId}")
    public Object like(@PathVariable TargetType type,
                       @PathVariable Long typeId,
                       @RequestParam String param) {
        return likeService.like(type, typeId, param);
    }
```

이렇게 반환된 Object는 Jackson에서 알아서 json으로 변환해서 내보낸다.

```Java
@Getter
public class PostDto {
    private String title;
    @JsonIgnore
    private String author;
}
```

이런 DTO 객체를 반환한다고 치면

```Json
{
  "title": "foo"
}
```

이런 json이 반환될 것이다. author는 `@JsonIgnore` 어노테이션을 사용했기 때문에 변환에서 생략됐다.

---

주의할 점은 Jackson의 기본 매핑은 객체의 프로퍼티, 즉 `Getter` 기준으로 이루어진다는 것이다.

멤버 변수 기반으로 하고 싶다면 추가로 설정을 해줘야만 한다.

```Java
public class PostDto {
    @JsonProperty("title")
    private String title;
    @JsonProperty("author")
    private String author;
}
```

반환값:

```Json
{
  "title": "foo",
  "author": "bar"
}
```

---

## 추가적인 사용법들

* ### 출력 순서 지정하기 (`@JsonPropertyOrder`)

```Java
@Getter
public class PostDto {
    private String title;
    private String author;
    private String content;
    private String comments;
    private int likeCount;
}
```

이 객체를 반환했을 때

```Json
{
  "title": "foo",
  "author": "bar",
  "content": "Zzz",
  "comments": "No Jam",
  "likeCount": 3
}
```

이런 형태의 값을 기대하겠지만, 실제로는 그렇지 않을 확률이 높다.

```Json
{
  "likeCount": 3,
  "content": "Zzz",
  "title": "foo",
  "comments": "No Jam",
  "author": "bar"
}
```

이렇게 뒤죽박죽이 되서 나오는 것이다. 완전 무작위는 아니고 내부적으로 뭔가 규칙이 정해져 있겠지만, 이걸 사람이 직관적으로 파악하기는 어렵다.

그래서 개발자가 직접 변환할 데이터의 순서를 지정해줄 수도 있다.

```Java
@Getter
@JsonPropertyOrder({"title", "author", "content", "comments", "likeCount"})
public class PostDto {
    private String title;
    private String author;
    private String content;
    private String comments;
    private int likeCount;
}
```

이렇게 `@JsonPropertyOrder` 어노테이션 뒤에 원하는 순서를 기입하면, 개발자가 원하는 순서대로 출력되서 나온다.

## 결론

그동안 컨트롤러를 작성하면서 자동변환 기능을 많이 사용해왔지만, 정작 그것이 어떤 라이브러리에 의해 수행되는지, 어떤 규칙이 있는지는 잘 모르고 썼었다.

그러다 이번에 데이터의 출력 순서를 바꿔보고자 방법을 찾던 중, 좀 더 디테일한 정보를 얻게 되었다.

앞으로도 Jackson 라이브러리의 사용법에 대해 종종 추가할 예정이다.
