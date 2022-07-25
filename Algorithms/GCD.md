# 최대공약수 Greatest Common Divisor

[목록으로 돌아가기](../README.md)

두 수의 최대공약수를 구하는 방법으로 유클리드 호제법을 사용해봤다.

`Java`

```Java
public int gcd(int a, int b) {
    if (b == 0)
        return a;

    return gcd(b, a % b);
}
```

원리는 간단하다. 주어진 두 수(a, b)의 나머지값(a % b)을 구하고, 그 다음 번엔 두 수 중 작은 수(b)를 앞에서 구한 나머지값(a % b)으로 나누어 다시 새로운 나머지값을 구한다.

이걸 나머지가 0이 나올 때까지 반복하면, 남은 피제수(a')가 최초에 주어진 두 수(a, b)의 최대공약수이다.

---

## 번외: 최소공배수 Least Common Multiple

최대공약수를 구할 수 있으면 최소공배수를 구하는 건 간단하다.

두 수(a, b)를 서로 곱한 뒤, 최대공약수로 나누면 된다.

`Java`

```Java
public int lcm(int a, int b) {
    return a * b / gcd(a, b);
}
```
