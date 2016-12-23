RxJava - Observable과 생명주기
=============

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square) 

###### [site](https://brunch.co.kr/@lonnie/17)

옵저버블에는 생명주기가 있다. 이 생명주기에는 옵저버에 의해 구독, 구독 해지 또는 아이템을 발행하거나 발행이 끝나면 완료가 되는 일련의 상태로 볼 수 있다. Rx에서는 각각의 상태에 대응하는 콜백을 등록할 수 있는데 이런 생명주기 콜백이 호출되는 시점을 정리해보았다. 이 콜백들을 응용하는 방법은 [Grokking Andoid](http://www.grokkingandroid.com/rxjavas-side-effect-methods/)의 글이 도움된다.

## 1. doOnSubscribe
doOnSubscribe() 함수는 옵저버블이 구독될 때 호출되는 콜백 함수를 등록할 수 있다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/9A4tkSEG8goqdhdCZjpw1iB8pdg.png)

```java
Observable<Integer> observable =
    Observable.just(0)
              .doOnSubscribe(() -> log.info("doOnSubscribe"));

observable
    .subscribe(
        i -> log.info("onNext: {}", i),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```

`실행결과`
```
doOnSubscribe
onNext: 0
onCompleted
```

## 2. doOnUnsubscribe
doOnUnsubscribe() 함수는 옵저버블이 구독 해지될 때 호출되는 콜백 함수를 등록할 수 있다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/f1VOvcAdHH7nHfWbS3cl3QvE7Kk.png)

```java
Observable<Integer> observable =
        Observable.just(0)
                  .doOnUnsubscribe(() -> log.info("doOnUnsubscribe"));

Subscription subscription =
        observable.subscribe(
            i -> log.info("onNext: {}", i),
            e -> log.info("onError: {}", e.getMessage()),
            () -> log.info("onCompleted")
        );

subscription.unsubscribe();
```

`실행결과`
```
onNext: 0
onCompleted
doOnUnsubscribe
```

## 3. doOnNext
doOnNext() 함수는 옵저버블이 아이템을 발행할 때 호출되는 콜백 함수를 등록할 수 있다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/AuOKMapdPkKaWbLSl1lH1zyFNG8.png)

```java
Observable<Integer> observable =
            Observable.just(0, 1, 2)
                      .doOnNext(i -> log.info("doOnNext: " + i));

observable
    .subscribe(
        i -> log.info("onNext: {}", i),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```

`실행결과`
```
doOnNext: 0
onNext: 0
doOnNext: 1
onNext: 1
doOnNext: 2
onNext: 2
onCompleted
```

## 4. doOnCompleted
doOnCompleted() 함수는 옵저버블이 완료를 발행할 때 호출되는 콜백 함수를 등록할 수 있다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/RymxC5Zz_MQc4ctxJMZRf-Erl1c.png)

```java
Observable<Integer> observable =
    Observable.just(0, 1, 2)
              .doOnCompleted(() -> log.info("doOnCompleted"));

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
onNext: 1
onNext: 2
doOnCompleted
onCompleted
```

## 5. doOnError
doOnError() 함수는 옵저버블이 에러를 발행할 때 호출되는 콜백 함수를 등록할 수 있다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/zok5OjJbQugJBn1ZIrmmBVOZx90.png)

```java
Observable<Integer> observable =
    Observable.just(0, 1)
              .doOnNext(i -> {
                  if (i > 0) {
                      throw new RuntimeException("Rx 예외");
                  }
              })
              .doOnError(e -> log.error("doOnError: {}", e.getMessage()));

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
doOnError: Rx 예외
onError: Rx 예외
```

## 6. doOnEach
doOnEach() 함수는 옵저버블이 아이템, 에러, 완료를 발행할 때 호출되는 콜백 함수를 등록할 수 있다. 파라미터로 Notification 객체가 전달되고, 이 객체를 통해 아이템과 이벤트 타입을 알 수 있다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/wDibNLihuXHmPyXK-TGwritSniE.png)

```java
Observable<Integer> observable =
    Observable.just(0)
              .doOnEach(notification -> log.info("doOnEach: {}", notification));

observable
    .subscribe(
        i -> log.info("onNext: {}", i),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```

`실행결과`
```
doOnEach: [rx.Notification@5dca858b OnNext 0]
onNext: 0
doOnEach: [rx.Notification@9e89d68 OnCompleted]
onCompleted
```

## 7. doOnTerminate
doOnTerminate() 함수는 옵저버블이 에러, 완료를 발행할 때 호출되는 콜백 함수를 등록할 수 있다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/jimHz2FGGopyZvKArtEB6FsKG0Q.png)

```java
Observable<Integer> observable =
    Observable.just(0)
              .doOnTerminate(() -> log.info("doOnTerminate"));

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
doOnTerminate
onCompleted
```

```java
Observable<Object> observable =
    Observable.error(new Error("Unknown error"))
              .doOnTerminate(() -> log.info("doOnTerminate"));

observable
    .subscribe(
        i -> log.info("onNext: {}", i),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```
`실행결과`
```
doOnTerminate
onError: Unknown error
```

## 8. doAfterTerminate
doAfterTerminate() 함수는 옵저버블 에러, 완료를 발행 후 호출되는 콜백 함수를 등록할 수 있다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/SQo/image/tuH1-bs70NtOeKFsO4gv59T88xA.png)

```java
Observable<Integer> observable =
    Observable.just(0)
              .doAfterTerminate(() -> log.info("doAfterTerminate"));

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
onCompleted
doAfterTerminate
```

```java
Observable<Object> observable =
    Observable.error(new Error("Unknown error"))
              .doAfterTerminate(() -> log.info("doAfterTerminate"));

observable
    .subscribe(
        i -> log.info("onNext: {}", i),
        e -> log.info("onError: {}", e.getMessage()),
        () -> log.info("onCompleted")
    );
```
`실행결과`
```
onError: Unknown error
doAfterTerminate
```




