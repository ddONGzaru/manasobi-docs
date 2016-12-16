RxJava의 lift에 관하여
=============

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square) ![badge](https://img.shields.io/badge/manasobi-RxAndroid-yellowgreen.svg?style=flat-square)

## lift의 정의
RxJava는 메소드 체인으로 처리를 연결해 가는 것이 일반적인데, 이 메소드 체인을 실현하고 있는 것이 `lift`이다. 일반 개발에서 `lift`를 접할 기회는 적다고 생각하지만, `map`, `filter`등의 오퍼레이터의 내부는 `lift`와 `Operator<R, T>`의 조합으로 만들어져있다. `lift`를 이해하면 스스로 커스텀 오퍼레이터를 만드는것도 가능하다. [Implementing Your Own Operators]()

## lift 구조
`map`을 예로 들어 `lift`가 무엇을 하고 있는지를 확인해 본다. `map`은 값을 입력받아, 가공하고 반환하는 오퍼레이터이다.

```java
.map(text -> {
   return Integer.valueOf(text);
})
```

그럼 map의 구현을 들여다보자. `map`은 [T를 수신해서 R을 반환하는 `Func1`을 인수로서 받아들이고, `Observable<R>`을 반환]하는 메소드이다.

```java
public final <R> Observable<R> map(Func1<? super T, ? extends R> func) {
   return lift(new OperatorMap<T, R>(func));
}
```

//TO-DO



