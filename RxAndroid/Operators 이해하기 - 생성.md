Operators 이해하기 - 생성
========================

![badge](https://img.shields.io/badge/manasobi-RxAndroid-yellowgreen.svg?style=flat-square)

Rx프로그래밍의 장점은 데이터 흐름대로 코딩이 가능합니다.
체이닝이라는 것을 통해 우리가 원하는 작업대로 데이타를 가공하고 처리 할 수 있습니다.
이를 도와주는 것 중 정해진 일을 처리하는 Observable들이 있습니다. 우리는 이것을 Operator라고 부르는데요.
오늘은 이 Operator의 종류와 하는 일에 대해서 알아보겠습니다.

공식 문서를 보면 아래와 같은 마블 다이어그램을 확인 하실 수 있는데 각 아이템에 대한 설명은 다음과 같습니다.

![alt](http://cfile9.uf.tistory.com/image/247D1D3356CD13A913A85F)

Operator를 카테고리 별로 나눠보면 다음과 같습니다.

생성, 변형, 분류, 조합, 에러처리, 유틸리티, 조건과 상태(boolean), 수학과 집계, backpressure, connectable, 변환
한번에 다둘려고 했으나 양이 너무 많아 파트를 나눠서 포스팅 합니다.

이번 포스팅은 옵져버블의 생성에 관련된 오퍼레이터 입니다.

## 생성 옵저버블
새로운 Observable을 생성하는 Operator 입니다.

- **Create** 수동으로 옵져버 메소드 호출하여 새로운 옵저버블을 생성합니다.

![alt](http://reactivex.io/documentation/operators/images/create.c.png)

```java
Observable.create(new Observable.OnSubscribe<String>() {
    @Override
    public void call(Subscriber<? super String> subscriber) {
        try {
            subscriber.onNext("Hello_Create");
            subscriber.onCompleted();
        } catch (Exception e) {
            subscriber.onError(e);
        }
    }
})
.compose(
    mMainView.ActivityLifecycleProvider()
    .bindToLifecycle()
)
.observeOn(AndroidSchedulers.mainThread())
.subscribe(s -> {
            Log.d(TAG, s);
            mMainView.TextChange(s);
        },
        throwable -> throwable.printStackTrace(),
        () -> {
            Log.d(TAG, "onComplete");
            LogTextView();
        }
);

```

- **Defer** 구독하기 전까지 옵저버블을 생성하지 않습니다. 그리고 각각의 옵져버에게 매번 새로운 옵저버블을 생성합니다.
다른 생성 오퍼레이터와 다른 점이 뭔지 애매했었는데 데이타스트림이 메모리에 할당되는 타이밍이 다른 것이 였습니다.
다른 오퍼레이터들은 오퍼레이터를 선언하는 순간 메모리에 할당 되지만 defer는 subscribe가 호출 될 때에 할당 된다고 합니다.

![alt](http://reactivex.io/documentation/operators/images/defer.c.png)

```java
Observable.defer(() -> {
    return SomethingLongTask(); //return Observable<String>
})
.compose(mMainView.ActivityLifecycleProvider().bindToLifecycle());
.observeOn(AndroidSchedulers.mainThread())
.subscribe(s -> {
            Log.d(TAG, s);
            mMainView.TextChange(s);
        },
        throwable -> throwable.printStackTrace(),
        () -> {
            Log.d(TAG, "onComplete");
            LogTextView();
        }
);
```

### Empty/Never/Throw 
매우 정확하고 제한적인 행동의 Observable을 생성합니다.

- **Empty** 방출하는 아이템이 없고 정상적으로 종료되는 옵저버블을 생성합니다.
![alt](http://reactivex.io/documentation/operators/images/empty.c.png)

- **Never** 방출하는 아이템이 없고 종료되지 않는 옵저버블을 생성합니다.
![alt](http://reactivex.io/documentation/operators/images/never.c.png)

- **Throw** 방출하는 아이템이 없고 에러를 발생하여 종료되는 옵저버블을 생성합니다.
![alt](http://reactivex.io/documentation/operators/images/throw.c.png)

- **From** 배열이나 Iterable의 요소를 순차적으로 방출 시키는 Observable으로 변환합니다.
![alt](http://reactivex.io/documentation/operators/images/from.c.png)

- **Interval** 특정한 시간 간격으로 아이템을 방출하는 Observable을 생성합니다.
일정시간 마다 반복적인 작업이 필요할 때 사용하면 좋을 것 같네요.
![alt](http://reactivex.io/documentation/operators/images/interval.c.png)

- **Just** 오브젝트나 오브젝트셋을 바로 방출하는 Oservable으로 변환 합니다.
만약에 아무것도 하지 않는 옵저버블을 만들기 위해 null 을 넣는다면 null을 방출하는 옵저버블이 만들어 집니다.
아무것도 하지 않은 옵저버블을 원하신다면 empty를 사용하시면 됩니다.
![alt](http://reactivex.io/documentation/operators/images/just.c.png)

- **Range** 정수의 순차적인 범위를 가지고 있는 Observable을 생성합니다.
Interval과 비슷하지만 반복횟수의 제한이 있습니다. m개 만큼의 반복합니다.
![alt](http://reactivex.io/documentation/operators/images/range.c.png)

- **Repeat** 일정 횟수를 반복하는 Observable을 생성합니다.
이 오퍼레이터는 단독으로 사용되지 않고 다른 오퍼레이터 뒤에 붙여서 사용되며 .Repeat(n) 바로 앞 오퍼레이터를 일정횟수 만큼 반복합니다.
![alt](http://reactivex.io/documentation/operators/images/repeat.c.png)

- **Start** 함수의 결과 값을 방출하는 Observable을 생성합니다.
[https://github.com/ReactiveX/RxJavaAsyncUtil](https://github.com/ReactiveX/RxJavaAsyncUtil)을 디펜던시에 추가해야 사용 할 수 있다.
추가한 디펜던시에서 사용할 수 있는 오퍼레이터는 아래의 링크에서확인이 가능 합니다.
[https://github.com/ReactiveX/RxJava/wiki/Async-Operators](https://github.com/ReactiveX/RxJava/wiki/Async-Operators)
![alt](http://reactivex.io/documentation/operators/images/start.c.png)

- **Timer** 일정 시간의 딜레이 이후에 단일 항목을 방출하는 Observable을 생성합니다.
![alt](http://reactivex.io/documentation/operators/images/timer.c.png)













