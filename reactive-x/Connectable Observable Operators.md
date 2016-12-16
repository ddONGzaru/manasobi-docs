Connectable Observable Operators
========================

![badge](https://img.shields.io/badge/manasobi-RxAndroid-yellowgreen.svg?style=flat-square)

[ConnectableObservable](http://reactivex.io/RxJava/javadoc/rx/observables/ConnectableObservable.html)의 서브 클레스와 오퍼레이터에 대해서 설명을 하고자 한다.

| Operators | 설명 |
| --- | --- |
| ConnectableObservable.connect( ) | Connectable Observable에게 아이템 방출을 시작하라고 지시한다. |
| Observable.publish( ) | Observable을 Connectable Observable으로 변형시킨다. |
| Observable.replay( )  | 모든 Observer들에게 방출이 시작된 후에 구독을 했을 경우라도 같은 순서의 방출된 아이템을 볼 수 있도록 보장합니다. |
| ConnectableObservable.refCount( ) | Connectable Observable을 일반적인 Observable처럼 작동하도록 만듭니다. |

Connectable Observable은 구독을 하더라도 아이템 방출을 시작하지 않는다는 점을 제외하면 일반적인 Observable과 비슷합니다. connect()을 호출했을 때에만 방출합니다. 이 방법으로 Subscriber들에게 Observable가 방출을 시작하기전에 Observable구독하도록 기다릴 수 있습니다.

![alt](https://github.com/ReactiveX/RxJava/wiki/images/rx-operators/publishConnect.png)

아래의 예제코드는 같은 Observable을 구독하는 두개의 subscriber를 보여주는 코드입니다. 첫번째 케이스에서는 일반적인 Observable이고 두번째 케이스에서는 Connectable Observable으로 subscriber가 모두 구독한 이후 연결하였습니다. 차이점은 아웃풋을 통해 확인이 가능합니다.

#### Example #1:
```java
Observable firstMillion = 
    Observable.range(1, 1000000)
              .sample(7, java.util.concurrent.TimeUnit.MILLISECONDS);

firstMillion.subscribe(
   { println("Subscriber #1:" + it); },       // onNext
   { println("Error: " + it.getMessage()); }, // onError
   { println("Sequence #1 complete"); }       // onCompleted
);

firstMillion.subscribe(
    { println("Subscriber #2:" + it); },       // onNext
    { println("Error: " + it.getMessage()); }, // onError
    { println("Sequence #2 complete"); }       // onCompleted
);
```
```
Subscriber #1:211128
Subscriber #1:411633
Subscriber #1:629605
Subscriber #1:841903
Sequence #1 complete
Subscriber #2:244776
Subscriber #2:431416
Subscriber #2:621647
Subscriber #2:826996
Sequence #2 complete
```

#### Example #2:
```java
Observable firstMillion = 
    Observable.range(1, 1000000)
            .sample(7, java.util.concurrent.TimeUnit.MILLISECONDS)
            .publish();

firstMillion.subscribe(
   { println("Subscriber #1:" + it); },       // onNext
   { println("Error: " + it.getMessage()); }, // onError
   { println("Sequence #1 complete"); }       // onCompleted
);

firstMillion.subscribe(
   { println("Subscriber #2:" + it); },       // onNext
   { println("Error: " + it.getMessage()); }, // onError
   { println("Sequence #2 complete"); }       // onCompleted
);

firstMillion.connect();
```
```
Subscriber #2:208683
Subscriber #1:208683
Subscriber #2:432509
Subscriber #1:432509
Subscriber #2:644270
Subscriber #1:644270
Subscriber #2:887885
Subscriber #1:887885
Sequence #2 complete
Sequence #1 complete
```