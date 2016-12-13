RxJava의 compose 또는 RxLifecycle에 관하여
========================

![badge](https://img.shields.io/badge/manasobi-RxAndroid-yellowgreen.svg?style=flat-square)

###### Qiita [kazy Edited at 2016-01-03](http://qiita.com/kazy/items/8733c1313e7d60908298#tldr)

## TLDR 
- `compose`는 메소드 체인중에서 `Observable`이 억세스가능한 최고의 오퍼레이터.
- `lift`와의 차이점은 `Observable`의 값에 접근 가능한지 아닌지임.
- `flatMap`과의 차이점은 `Observable`의 값에 접근 가능한지 아닌지와 더불어 실행되는 타이밍.
- RxLifecycle은 takeUntil에서 라이프사이클 이벤트를 바인딩 함.

`compose`라고는 하지만, 최근에는 [trello/RxLifecycle](https://github.com/trello/RxLifecycle)에도 사용되고 있기때문에, 익숙한 사람들도 많을 것이다. `compose`는 [lift](http://qiita.com/kazy/items/69d56c171bd4a393385c)와 마찬가지로 RxJava의 메소드체인을 구현하고있는 요소 중의 하나이다. 그럼 이 두가지의 오퍼레이터 사이에는 어떤 차이점이 있는것일까?

## lift와 compose의 차이
> 당신이 만든 오퍼레이터가 `Observable`로부터 방출되는 개개의 요소에 작용하는 것이라면 `lift`를 사용해야만 한다. 하지만, `Observable` 그 자체에 적용하는 경우에는 `compose`를 사용해야만 한다.

`lift`와 `compose`는 닮은것 같지만, 실제 가능한 기능은 다르다. 표로 정리하면 아래와 같다.

|Operator|직접 Observable를 다루는가?|Observable 안의 값에 접근가능한가?|
|---|---|---|
|compose|가능|불가능|
|lift|불가능|가능|

`compose`는 직접 값에 접근이 불가능하기때문에, `map`과 같은 처리는 수행하지 못한다. 하지만, `compose`는 Observable를 직접 다루는 메리트가 있다. `RxLifecycle`은 이 특성을 이용해서 라이프사이클 이벤트를 Observable에 대하여 bind하고있다.

## compose의 구성
그럼 실제로 `compose`가 어떻게 동작하고 있는지를 확인해보자. 

```java
public <R> Observable<R> compose(
    Transformer<? super T, ? extends R> transformer) {
    return ((Transformer<T, R>) transformer).call(this);
}
```

`Transformer<? super T, ? extends R>`를 인수로 하여, Transformer에 따른 변환을 자기자신에게 적용하고 있다. 그럼 Transformer의 인터페이스를 살펴보자.

```java
public interface Transformer<T, R> 
    extends Func1<Observable<T>, Observable<R>> {
    // cover for generics insanity
}
```

Transformer는 `Observable<T>`를 받아들이고, `Observable<R>`을 반환하는 `Func1`이다. 즉, `Transformer`는 Observable을 수신하고 가공해서, 반환하는 함수이므로, `compose`는 `Observable이 자기자신에 함수를 적용하기 위한 오퍼레이터`라는 것을 알 수 있다.

## compose와 flatMap의 차이
RxJava의 처리에서 자주 사용되는 flatMap이라는 것이 있다. 이것도 `Observable`을 반환하는 오퍼레이터인데, 어떠한 차이가 있는지 살펴보자.

|Operator|호출되는 타이밍|반환 값|직접 Observable을 조작|Observable의 내용 값에 접근 가능|
|---|---|---|---|---|
|compose|subscribe된 타이밍(한번 만)|Observable|가능|불가능|
|flatMap|onNext가 호출될때 마다(복수 가능)|Observable|불가능|가능|






