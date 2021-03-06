RxJava - Observable 생성하기
=============

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square) 

###### [site](https://brunch.co.kr/@lonnie/18)

RxJava에서는 옵저버블을 생성하는 다양한 방법을 제공한다. 기존 함수의 결과 값을 옵저버블로 발행하거나 리스트의 값을 하나씩 발행할 수도 있으며 옵저버블 생성을 구독이 발생할 때로 지연시킬 수도 있다. 심지어 아무것도 발행하지 않고 종료하거나, 에러를 발행하는 옵저버블을 생성할 수도 있는데 이번 포스트에서는 옵저버블을 생성하는 방법에 대해 알아본다.

## 1. Observable.create()
create() 함수는 옵저버블 생성 시 가장 많이 사용되는 함수 중 하나다. 이 함수는 OnSubscribe 객체를 파라미터로 가지며 구독이 발생하면 이 객체의 call()함수가 실행된다. 옵저버에게 아이템을 발행하기 위해서는 call()함수 내부에서 onNext(), onError(), onCompleted()를 적절히 호출해야 한다. onError()와 onCompleted()는 동시에 호출할 수 없는 상호 배타적 관계로 정의할 수 있는데, onError()가 호출될 때는 onCompleted()를 호출하지 않아야 하고, onCompleted()가 호출될 때는 onError()를 호출하지 않아야 한다. 즉, 이 두 함수 중 하나가 호출된 이 후에는 옵저버의 어떠한 함수도 호출하지 않아야 한다.

![alt](http://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/rRJfoEWA1zixjuCJQbaIF_qvBRc.png)

```java
Observable<Integer> observable = Observable.create(subscriber -> {

    try {
        if (!subscriber.isUnsubscribed()) {
            for (int i = 0; i < 5; i++) {
                subscriber.onNext(i);
            }
            subscriber.onCompleted();
        }
    } catch (Exception e) {
        subscriber.onError(e);
    }
});

observable.subscribe(
    i -> log.info("onNext: {}", String.valueOf(i)),
    e -> log.info("onError: {}", e.getMessage()),
    () -> log.info("onCompleted")
);
```

`실행결과`
```java
onNext: 0
onNext: 1
onNext: 2
onNext: 3
onNext: 4
onCompleted
```

## 2. Observable.defer()
defer()는 지연 초기화(Lazy Initializations)를 제공하는 함수다. 즉, 구독이 발생할 때 비로소 옵저버블을 생성한다. defer()의 파라미터로 Fun0<R>을 가지는데 이 함수는 구독이 발생할 때마다 호출되기 때문에 메번 새로운 옵저버블 객체가생성된다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/j_Iap1d1SuKCw0j40732dRJ9QGc.png)

```java
Observable<Integer> observable = Observable.defer(() -> {
    log.info("Create an observable :: {}", UUID.randomUUID().toString());
    return Observable.just(0);
});

observable
    //.delaySubscription(1, TimeUnit.SECONDS)
    .subscribe(
        i -> log.info("onNext: {}", String.valueOf(i)),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );

observable
    //.delaySubscription(2, TimeUnit.SECONDS)
    .subscribe(
        i -> log.info("onNext: {}", String.valueOf(i)),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```

`실행결과`

```
Create an observable :: 36f7c399-b276-4481-b1ec-49e244c1f594
onNext: 0
onCompleted
Create an observable :: eaff430a-d068-4d44-b575-698d947ee217
onNext: 0
onCompleted
```

## 3. Observable.fromCallable()
파라미터로 Callable을 갖는다. defer와 마찬가지로 구독이 발생할 때 Callable의 call() 함수가 호출되는 지연 초기화를 위한 함수다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/wYNH3jc6pcC9ShUwQkJDjFkFC6c.png)

```java
Observable<String> observable = Observable.fromCallable(() -> {
    String uuid = UUID.randomUUID().toString();
    log.info("Create an observable :: {}", uuid);
    return uuid;
});

observable
    .subscribe(
        i -> log.info("onNext: {}", i),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );

observable
    .subscribe(
        i -> log.info("onNext: {}", i),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```

`실행결과`
```
Create an observable :: 72deb308-8331-4ada-82f7-405352f0d977
onNext: 72deb308-8331-4ada-82f7-405352f0d977
onCompleted
Create an observable :: 064fd4f9-8170-4d27-9595-fbe1f92e57f4
onNext: 064fd4f9-8170-4d27-9595-fbe1f92e57f4
onCompleted
```

## 4. Observable.interval()
특정 시간 간격을 주기로 0부터 증가하는 정수 값을 발행한다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/QOSiSd5zNd5illhozPh_7xpt2tk.png)

```java
Observable<Long> observable = Observable.interval(1, TimeUnit.SECONDS);

observable
    .take(3)
    .subscribe(
        i -> log.info("onNext: {}", String.valueOf(i)),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```

`실행결과`

```
16:48:23.465 [RxComputationScheduler-1] INFO Observable_interval - onNext: 0
16:48:24.462 [RxComputationScheduler-1] INFO Observable_interval - onNext: 1
16:48:25.462 [RxComputationScheduler-1] INFO Observable_interval - onNext: 2
16:48:25.462 [RxComputationScheduler-1] INFO Observable_interval - onCompleted
```

## 5. Observable.range()
특정 범위 내의 정수 값을 순차적으로 발행하는 옵저버블을 생성한다. 파라미터로 시작 값과 개수를 갖는다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/FTr_1rfWuGpY1a-xMGFeRi587UA.png)

```java
Observable<Integer> observable = Observable.range(1, 3);

observable
    .subscribe(
        i -> log.info("onNext: {}", i),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```

`실행결과`
```
onNext: 1
onNext: 2
onNext: 3
onCompleted
```

## 6. Observable.repeat()
아이템을 N번 발행한다. 파라미터로 아무것도 넘기지 않으면 아이템을 무한히 발행한다. 스케줄러로 trampoline을 사용하고, 변경 가능하다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/xZ8p45NU4ZMhmHAT3oICA60sKPM.png)

```java
Observable<Integer> observable = Observable.just(0).repeat(3);

observable
    .subscribe(
        i -> log.info("onNext: {}", i),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```

`실행결과`
```
onNext: 0
onNext: 0
onNext: 0
onCompleted
```

## 7. Observable.timer()
특정 시간 이후에 숫자 0을 발행한다. 스케쥴러로 computation을 사용하고, 변경 가능하다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/n8A97GME720h7RpXJ5L08rympg0.png)

```java
log.info("start...");

Observable<Long> observable = Observable.timer(2, TimeUnit.SECONDS);

observable
    .subscribe(
        i -> log.info("onNext: {}", i),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```

`실행결과`
```
10:08:47.607 start...
10:08:49.680 onNext: 0
10:08:49.684 onCompleted
```

## 8. Observable.just()
just()함수는 파라미터로 주어진 아이템을 옵저버블로 발행한다. 샘플 코드처럼 기존 함수의 반환 값을 옵저버블로 변환할 때 사용할 수 도 있다. 1~10개까지의 아이템을 발행할 수 있도록 함수 오버로딩 되었다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/JgfY4ZK9sTox8xZZunh0uKwk7S8.png)

```java
Observable<String> observable = Observable.just("A", "B", "C");

observable
    .subscribe(
        i -> log.info("onNext: {}", i),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```

`실행결과`
```
onNext: A
onNext: B
onNext: C
onCompleted
```

## 9. Observable.from()
이터러블, 배열의 아이템을 순차적으로 발행하는 옵저버블을 생성한다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/GOJCxhYI8AGaYM5HT2kX3sV9DWU.png)

```java
Observable<Integer> observable = Observable.from(Arrays.asList(1, 2, 3));

observable
    .subscribe(
        i -> log.info("onNext: {}", i),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```

`실행결과`
```
onNext: 1
onNext: 2
onNext: 3
onCompleted
```

## 10. Observable.empty()
아무런 아이템을 발행하지 않고, 완료를 발행하는 옵저버블을 생성한다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/zvRmPa8FL-JKzW2P1VfP_9Gr7ec.png)

```java
Observable<Integer> observable = Observable.empty();

observable
    .subscribe(
        i -> log.info("onNext: {}", i),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```

`실행결과`
```
onCompleted
```

## 11. Observable.never()
아무런 아이템도 발행하지 않고, 완료도 발행하지 않는 옵저버블을 생성한다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/pzm0qJ7y_eDuX3ZXmqyL0_BVHh0.png)

```java
Observable<Integer> observable = Observable.never();

observable
    .subscribe(
        i -> log.info("onNext: {}", i),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```

`실행결과`
```
아무것도 출력되지 않음
```

## 12. Observable.throw()
에러를 발행하는 옵저버블을 생성한다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/XHVz3vmH7PP4S4pEBxKTcr0NzC4.png)

```java
Observable<Integer> observable = Observable.error(new Exception("RX 에러"));

observable
    .subscribe(
        i -> log.info("onNext: {}", i),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```

`실행결과`
```
onError: RX 에러
```

