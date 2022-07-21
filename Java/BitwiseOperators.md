# 자바 비트 연산자

알고리즘 문제를 풀던 도중, 2진수를 사용하는 문제가 나왔다.

둘 다 0이면 " "(빈칸), 하나라도 1이면 "#"을 채워넣는 식이었는데, C++에서 사용했던 비트 연산을 사용하면 쉽겠다는 생각이 들었다.

## 10진수와 2진수 String간의 변환

```Java
// 10진수 숫자 10
int decimal = 10;

// 2진수 문자열 "1010"
String binary = Integer.toBinaryString(decimal);

// 다시 10진수 숫자 10
// parseInt의 2번째 인자에는 진수(radix)가 들어가는데, 기본값은 10이다.
decimal = Integer.parseInt(binary, 10);
```

## 비트 연산자 사용

비트 연산자에는 4종류가 있는데, 내가 사용한 것은 `OR |`연산자다.

```Java
// binA = 1010
int decA = 10;
// binB = 10100
int decB = 20;

// OR 연산은 두 2진수의 각 자리수끼리 비교해서, 하나라도 1이면 결과값도 1이 나온다.
//  01010
// +10100
// ------
//  11110

// binary = "11110"
String binary = Integer.toBinaryString(decA | decB);

// AND는 둘 다 1이어야 1, XOR는 서로 같으면 0, 다르면 1이다.
// NOT은 단순히 모든 자릿수를 반전시킨다. 1은 0으로, 0은 1로.
```
