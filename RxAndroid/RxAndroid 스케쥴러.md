RxAndroid 스케쥴러
=============

![badge](https://img.shields.io/badge/manasobi-RxAndroid-yellowgreen.svg?style=flat-square)

RxAndroid에서는 스케쥴러를 통해 어느 쓰레드에서 실행이 될지 결정 할 수 있습니다.<br>
스케쥴러는 subsctibeOn(), observeOn() 에서 각각 지정할 수 있는데 subsctibeOn()은 observable의 작업을 시작하는 쓰레드를 선택 할 수 있습니다.
중복해서 적을 경우 가장 마지막에 적힌 스레드에서 시작합니다.

observeOn()은 이후에 나오는 오퍼레이터, subscribe의 스케쥴러를 변경 할 수 있습니다.

RxJava와 RxAndroid에서 제공하는 스케쥴러는 다음과 같습니다.

| 스케쥴러 | 설명 |
|:---|:---|
| Schedulers.computation() | 이벤트 룹에서 간단한 연산이나 콜백 처리를 위해 사용됩니다. RxComputationThreadPool라는 별도의 스레드 풀에서 돌아갑니다. 최대 cpu갯수 개의 스레드 풀이 순환하면서 실행됩니다.|
| Schedulers.immediate() | 현재 스레드에서 즉시 수행합니다. <br>observeOn()이 여러번 쓰였을 경우 immediate()를 선언한 바로 윗쪽의 스레드를 따라갑니다.|
| Schedulers.from(executor) | 특정 executor를 스케쥴러로 사용합니다. |
| Schedulers.io() | 동기 I/O를 별도로 처리시켜 비동기 효율을 얻기 위한 스케줄러입니다. 자체적인 스레드 풀 CachedThreadPool을 사용합니다. API 호출 등 네트워크를 사용한 호출 시 사용됩니다. |
| Schedulers.newThread() | 새로운 스레드를 만드는 스케쥴러입니다. |
| Schedulers.trampoline() | 큐에 있는 일이 끝나면 이어서 현재 스레드에서 수행하는 스케쥴러. |
| AndroidSchedulers.mainThread() | 안드로이드의 UI 스레드에서 동작합니다. |
| HandlerScheduler.from(handler) | 특정 핸들러 handler에 의존하여 동작합니다. |

```java
Observable<Integer> observable = Observable.create(subscriber -> {
    for (Integer i : new Integer[]{1, 2, 3, 4, 5}) {
        Log.v(TAG, Thread.currentThread().getName() + " : onNext " + i);
        subscriber.onNext(i);
    }
});
 
observable.subscribe(
    integer -> test(integer), 
    e -> Log.v(TAG, Thread.currentThread().getName() + " : onErrorHandler")
);
```
이와 같은 작업이 있다고 했을 때 실행결과는 다음과 같다.

```
tiii.com.rxandroid V/MainPresenter_Imp: main : onNext 1
tiii.com.rxandroid V/MainActivity: main : subscribed 1
tiii.com.rxandroid V/MainPresenter_Imp: main : onNext 2
tiii.com.rxandroid V/MainActivity: main : subscribed 2
tiii.com.rxandroid V/MainPresenter_Imp: main : onNext 3
tiii.com.rxandroid V/MainActivity: main : subscribed 3
tiii.com.rxandroid V/MainPresenter_Imp: main : onNext 4
tiii.com.rxandroid V/MainActivity: main : subscribed 4
tiii.com.rxandroid V/MainPresenter_Imp: main : onNext 5
tiii.com.rxandroid V/MainActivity: main : subscribed 5
```

이번엔 observable에 스레드를 지정해서 다시 실행해보자.

```java
Observable<Integer> observable = Observable.create(subscriber -> {
    for (Integer i : new Integer[]{1, 2, 3, 4, 5}) {
        Log.v(TAG, Thread.currentThread().getName() + " : onNext " + i);
        subscriber.onNext(i);
    }
});
 
observable.subscribeOn(Schedulers.computation())
.subscribe(
        integer -> test(integer),
        e -> Log.v(TAG, Thread.currentThread().getName() + " : onErrorHandler")
);
```

```
tiii.com.rxandroid V/MainPresenter_Imp: RxComputationThreadPool-2 : onNext 1
tiii.com.rxandroid V/MainActivity: RxComputationThreadPool-2 : subscribed 1
tiii.com.rxandroid V/MainPresenter_Imp: RxComputationThreadPool-2 : onNext 2
tiii.com.rxandroid V/MainActivity: RxComputationThreadPool-2 : subscribed 2
tiii.com.rxandroid V/MainPresenter_Imp: RxComputationThreadPool-2 : onNext 3
tiii.com.rxandroid V/MainActivity: RxComputationThreadPool-2 : subscribed 3
tiii.com.rxandroid V/MainPresenter_Imp: RxComputationThreadPool-2 : onNext 4
tiii.com.rxandroid V/MainActivity: RxComputationThreadPool-2 : subscribed 4
tiii.com.rxandroid V/MainPresenter_Imp: RxComputationThreadPool-2 : onNext 5
tiii.com.rxandroid V/MainActivity: RxComputationThreadPool-2 : subscribed 5
```
작업 스레드가 Computation 스레드로 변경 된 것을 확인 할 수 있다.

안드로이드에서 작업 할 때 일반적으로 사용되는 방법은 별도의 스레드에서 작업 후 결과 값을 UI에 적용하거나 변경한다.
결과를 받는 스레드를 메인 스레드로 변경해보자.

```java
Observable<Integer> observable = Observable.create(subscriber -> {
    for (Integer i : new Integer[]{1, 2, 3, 4, 5}) {
        Log.v(TAG, Thread.currentThread().getName() + " : onNext " + i);
        subscriber.onNext(i);
    }
});
 
observable
    .subscribeOn(Schedulers.computation())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(
        integer -> test(integer),
        e -> Log.v(TAG, Thread.currentThread().getName() + " : onErrorHandler")
    );    
```

```
tiii.com.rxandroid V/MainPresenter_Imp: RxComputationThreadPool-1 : onNext 1
tiii.com.rxandroid V/MainPresenter_Imp: RxComputationThreadPool-1 : onNext 2
tiii.com.rxandroid V/MainPresenter_Imp: RxComputationThreadPool-1 : onNext 3
tiii.com.rxandroid V/MainPresenter_Imp: RxComputationThreadPool-1 : onNext 4
tiii.com.rxandroid V/MainPresenter_Imp: RxComputationThreadPool-1 : onNext 5
tiii.com.rxandroid V/MainActivity: main : subscribed 1
tiii.com.rxandroid V/MainActivity: main : subscribed 2
tiii.com.rxandroid V/MainActivity: main : subscribed 3
tiii.com.rxandroid V/MainActivity: main : subscribed 4
tiii.com.rxandroid V/MainActivity: main : subscribed 5
```

subscibeOn이 여러번 사용됐을 경우는 어떤지 확인해 보자.

```java
Observable<Integer> observable = 
    Observable.create(subscriber -> {
        for (Integer i : new Integer[]{1, 2, 3, 4, 5}) {
            Log.v(TAG, 
                Thread.currentThread().getName() + " : onNext " + i);
            subscriber.onNext(i);
        }
    });
 
observable
    .subscribeOn(Schedulers.computation())   
    .observeOn(Schedulers.newThread())
    .concatMap(integer -> {
        Integer newinteger = integer * 10;
        Log.v(TAG, 
            Thread.currentThread().getName() + 
                " : concatMap value :" + newinteger);
        return Observable.just(newinteger);
    })
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(
        integer -> test(integer),
        e -> Log.v(TAG, 
            Thread.currentThread().getName() + " : onErrorHandler")
    );
```

```
tiii.com.rxandroid V/MainPresenter_Imp: RxComputationThreadPool-4 : onNext 1
tiii.com.rxandroid V/MainPresenter_Imp: RxComputationThreadPool-4 : onNext 2
tiii.com.rxandroid V/MainPresenter_Imp: RxComputationThreadPool-4 : onNext 3
tiii.com.rxandroid V/MainPresenter_Imp: RxComputationThreadPool-4 : onNext 4
tiii.com.rxandroid V/MainPresenter_Imp: RxComputationThreadPool-4 : onNext 5
tiii.com.rxandroid V/MainPresenter_Imp: RxNewThreadScheduler-2 : concatMap value :10
tiii.com.rxandroid V/MainPresenter_Imp: RxNewThreadScheduler-2 : concatMap value :20
tiii.com.rxandroid V/MainPresenter_Imp: RxNewThreadScheduler-2 : concatMap value :30
tiii.com.rxandroid V/MainPresenter_Imp: RxNewThreadScheduler-2 : concatMap value :40
tiii.com.rxandroid V/MainPresenter_Imp: RxNewThreadScheduler-2 : concatMap value :50
tiii.com.rxandroid V/MainActivity: main : subscribed 10
tiii.com.rxandroid V/MainActivity: main : subscribed 20
tiii.com.rxandroid V/MainActivity: main : subscribed 30
tiii.com.rxandroid V/MainActivity: main : subscribed 40
tiii.com.rxandroid V/MainActivity: main : subscribed 50
```

subscibeOn이 여러번 선언 됐을 경우 위와 같이 각각의 스레드에서 실행되는 것을 확인 할 수 있다.