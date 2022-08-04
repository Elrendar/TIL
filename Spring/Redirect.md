# Redirect

[목록으로 돌아가기](/README.md)

## 개요

리디렉트라는 단어는 인터넷을 이용하다 보면 종종 접할 수 있는 용어다.

나는 웹 개발을 공부하기 이전에도, 리디렉트가 대충 '어딘가 다른 곳으로 자동으로 페이지를 넘겨준다.'라는 의미로 이해하고 있었다.

하지만 실제 사용해본 결과, 미묘하게 헷갈리는 부분이 많았다.

오늘은 최근에 겪은 몇몇 리디렉트 관련 시행 착오를 정리해보려 한다.

## 사례1

```Java
// 글 작성 API
@PostMapping
    public String createPost(PostRequestDto postRequestDto) {
        // 글을 db에 저장하고
        postService.savePost(postRequestDto);
        // 글 목록으로 이동
        return "redirect:/page/post_list";
    }
```

컨트롤러에 이런 코드를 작성한 적이 있다.

그런데 이상하게 글 작성만 하면 WhiteLabel 에러페이지가 나왔다. 대체 왜일까?

한참을 삽질하다가, 다른 리디렉트 코드를 보고서야 깨달았다.

내 컨트롤러에는 `/page/post_list`로의 매핑이 없었다는 사실을...

```Java
// 글 목록 보기
@GetMapping("/posts")
    public String getPosts(Model model,
                           @RequestParam(value = "page", defaultValue = "0") int page) {
        var postPages = postService.getPosts(page);
        model.addAttribute("postPages", postPages);
        return "/page/post_list";
    }
```

`/page/post_list.html`를 호출하는 api는 `/page/post_list`가 아니라 `/posts`였던 것이다...

결국 메서드 반환값을 `return "redirect:/posts";`으로 수정하자 문제가 사라졌다.

## 결론

아직 스프링을 공부한지 얼마 안 됐으니 당연하다고 볼 수도 있겠지만, 내가 그동안 너무 기본기도 없이 기능을 막 사용하고 있었다는 생각이 들었다.

리디렉트로 구글링을 하다 보니 `redirect:` 뒤에 `/`유무에 따라서 절대경로인지 상대경로인지가 구분된다고 한다.

또 리디렉트로 uri를 호출하는 것과, 타임리프로 html을 뷰로 넘기는 것의 차이도 알게 되었다.

리디렉트는 아예 해당 uri가 매핑된 컨트롤러의 메서드를 호출하지만, 타임리프는 그저 같은 이름의 html을 찾아서 뷰에 넘겨준다.

이 차이를 모르고 막 사용하다가, 새로고침을 누를 때마다 같은 글이 복사되는 버그를 만들어냈었다.

앞으로는 리디렉트와 뷰네임을 리턴하는 것의 차이를 이해하고, 적재적소에 사용해야 버그가 나지 않을 것 같다.
