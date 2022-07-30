# cannot resolve symbol 'symbol name'

[목록으로 돌아가기](/README.md)

## 에러를 마주친 상황

과제 작성 도중이었다. 이번 주에 공부한 것들 중 중요 키워드를 선정해 질문과 답변을 작성해는 과제였다.

주제는 '추상 클래스를 사용하는 이유'였고, 나는 익숙한 예시를 사용해 예제 코드를 작성하던 중이었다.

GameObject 추상 클래스를 상속 받은 Player와 Monstet 클래스가 각각 피해를 입는 메서드 `takeDamage()`를 호출하는 내용이다.

```Java
// Player와 Monster 모두 hp라는 필드와, takeDamage라는 메서드가 필요하다.
// 이를 추상클래스로 선언한 뒤
abstract class GameObject {
    protected int hp;
    protected GameObject(int hp) {
        this.hp = hp;
    }
    protected boolean takeDamage(int amount) {
        this.hp -= amount;
        return hp > 0;
    }
}
// Player와 Monster에서 상속받는다
class Player extends GameObject {
    protected Player(int hp) {
        super(hp);
    }
}

class Monster extends GameObject {
    protected Monster(int hp) {
        super(hp);
    }
}

// 이후 인스턴스를 생성해서 호출한다.
public class Battle {
  Player player = new Player(100);
  Monster monster = new Monster(200);
  player.takeDamage(50);
  monster.takeDamage(80);
}
```

아주 단순한 코드이다.

그런데 이상하게 IntelliJ가 takeDamage 메서드 호출에서 에러를 냈다.

`심볼 takeDamage를 해결할 수 없습니다.`

영어로는 cannot resolve symbol인 것 같다.

처음엔 `Alt + Enter` 단축키를 사용해봤지만 IntelliJ는 아무런 해결책을 제시하지 않았다.

구글링을 해봤는데, 이 문제는 여러 가지 원인이 있을 수 있다는, 다소 애매한 답변을 발견했다.

결국 이런 저런 방법을 다 시도해 보았다.

## 시도해본 방법들

* 접근 제한의 문제인가 싶어서 접근 제어자를 전부 public으로 바꿔보았다. 이 문제가 아니었다.

* 네이밍이나 반환값의 문제인가 싶어서, 필드명과 메서드명을 바꿔보고, 반환값도 void로 바꿔보았지만 변한 게 없었다.

## 문제 해결

```Java
// 이후 인스턴스를 생성해서 호출한다.
public class Battle {
  public static void main(String[] args) {
    Player player = new Player(100);
      Monster monster = new Monster(200);
      player.takeDamage(50);
      monster.takeDamage(80);
  }
}
```

아주 단순한 문제였다. 위 코드와의 차이점이 무엇일까?

바로 main메서드의 유무이다. 좀 더 정확히 말하자면, 기존의 코드는 메서드 외부, 즉 **클래스 내부에서 직접 메서드 호출**을 시도하고 있었다!

C++에서는 전역으로 변수 선언, 함수 선언, 함수 호출 모두 가능하기 때문에, 이런 문제를 겪어본 적이 없었다.

참 바보 같은 실수였지만, 덕분에 다시는 잊지 않을 것 같다.

자바에는 무조건 클래스와 메서드가 있어야 한다는 사실을...
