ReactiveX 입문
=============

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square) 

###### [source](http://bati11blog.hatenablog.com/entry/2015/04/30/170343)

## 전체 내용
- Observable, Observer
- Observable 조작
- Scheduler
- 역사와 이론적 이야기
- 무엇을 Observable 할 것인가
- GUI에서도 Rx 
- Subject

## Observable과 Observer

> 연속적인 데이터를 이벤트 핸들러로 처리하는 프로그래밍 스타일을 `리액티브 프로그래밍`이라고 부른다. 리액티브 프로그래밍에서는 데이터의 생성, 그 데이터 생성에 따른 이벤트 통지, 이벤트의 처리를 분리해서 기술한다.

#### 간단한 예
간단한 샘플로 정리해보자. 중요한 것은 [데이터의 생성], [데이터 생성에 따른 이벤트 통지], [이벤트 처리] 3개.

연속적인 데이터를 나타내는 것이 `Observable` 오브젝트이다. `Observable` 오브젝트가 [데이터의 생성], [데이터의 생성에 따른 이벤트 통지]를 수행한다.

[이벤트 처리]를 수행하는 것은 `Observer` 오브젝트이다.

```java
Observable<Integer> observable = Observable.create(observer -> {
    IntStream.range(1, 10).forEach(x -> observer.onNext(x));
    observer.onCompleted();
});
Observer<Integer> observer = new Observer<Integer>() {
    @Override
    public void onCompleted() {
        System.out.print("COMPLETE!");
    }

    @Override
    public void onError(Throwable e) {
        System.out.print("ERROR!");
    }

    @Override
    public void onNext(Integer integer) {
        if (integer % 2 == 0) System.out.println(integer * 3);
    }
};
observable.subscribe(observer);
```

위 예에서는 1부터 10까지의 수치 데이터를 생성. 람다식의 인수를 가지는 `observer`의 `onNext` 메소드를 호출하는 것으로 Observer 오브젝트에 이벤트를 통지한다. 데이터 전부가 생성되면 `observer`의 `onCompleted`메소드를 호출해서 [완료]를 통지한다.

`observable.subscribe(observer)`에서 Observer 오브젝트를 Observable 오브젝트에에 등록하면, 아까 람다식 내에서 observer.onNext(x)를 호출한 타이밍에서, Observer 오브젝트에 등록한 `onNext`메소드가 실행된다. 같은 것처럼 `onCompleted`도 실행된다.

`subscribe`메소드를 호출하고나서 처음으로 Observable은 데이터를 생성하기 시작한다.

`[실행결과]`
```java
6
12
18
24
COMPLETE!
```

#### 좀 더 자세히 알아보자
`Observable` 클라스에는 `Observable` 오브젝트 생성을 편하게 하기 위한 메소드가 준비되어 있다.
[http://reactivex.io/documentation/operators.html#creating](http://reactivex.io/documentation/operators.html#creating)

또한, `subscribe`메소드에는 람다식을 넘겨서 `onNext` 처리를 정의하는 것이 가능하다.

이것들을 사용하면 아까 작성한 예보다 심플하게 작성할 수 있다.

```java
Observable.range(1, 10)
    .subscribe(x -> {
        if (x % 2 == 0) System.out.println(x * 3);
    });
```

`onCompleted`와 `onError`도 구현하고 싶은 경우는 람다식을 3개 넘기면 된다.

```java
Observable.range(1, 10)
    .subscribe(
        x -> {
            if (x % 2 == 0) System.out.println(x * 3);
        },
        (e) -> System.out.print("ERROR!"),
        ()  -> System.out.print("COMPLETE!")
);
```

## Observable의 조작

`Observable` 오브젝트에는 java8의 `Stream`에 있는 람다식을 인수로 받아들이는 메소드들이 있다.

```java
Observable.range(1, 10)
    .filter(x -> x % 2 == 0)
    .map(x -> x * 3)
    .subscribe(
        (x) -> System.out.println(x),
        (e) -> System.out.print("ERROR!"),
        ()  -> System.out.print("COMPLETE!")
    );
```

이 `Observable` 오브젝트에 관한 메소드는 여러가지가 있으며, 공식 문서에는 아래와 같이 분류되어있다.

- 변환
    - [http://reactivex.io/documentation/operators.html#transforming](http://reactivex.io/documentation/operators.html#transforming)
- 필터링
    - [http://reactivex.io/documentation/operators.html#filtering](http://reactivex.io/documentation/operators.html#filtering)
- 합성
    - [http://reactivex.io/documentation/operators.html#combining](http://reactivex.io/documentation/operators.html#combining)    
- 집계
    - [http://reactivex.io/documentation/operators.html#mathematical](http://reactivex.io/documentation/operators.html#mathematical)    

## Scheduler

여기까지라면 Java8의 Stream을 사용하는 것고 별반 다를바가 없다고 느껴질 것이다. Stream으로 처리 가능한 것을 일부러 이벤트 핸들러로 처리하는 것이 무슨 메리트가 있을까?

메리트라고 한다면, Rx를 사용하면 비동기처리가 된다는 점이다.

Rx에서는 데이터의 생성이나 이벤트에 대한 처리를 메인쓰레드는 별도 쓰레드에서 움직이게하기 위한 구조가 준비되어있다. 그것이 Scheduler이다.

기본으로는 전부 메인쓰레드에서 동작한다. `observable.subscribeOn` 메소드를 사용하면 Observable이 데이터를 생성하는 부분(Observable.create에 넘긴 람다식)이나 Observable을 가공하는 처리(map, filter등), Observer가 동작하는 쓰레드를 지정하는 것이 가능하다.

지정가능한 쓰레드의 종류는 아래를 참고한다.

[http://reactivex.io/documentation/scheduler.html](http://reactivex.io/documentation/scheduler.html)

아래와 같이 `subscribeOn(Schedulers.computation())`을 지정하면 메인쓰레드와는 별도의 쓰레드에서 동작하는 것을 알수있다.

```java
Observable.range(1, 10)
        .subscribeOn(Schedulers.computation())
        .filter(x -> {
            System.out.println("filter: " + Thread.currentThread().getName());
            return x % 2 == 0;
        })
        .map(x -> {
            System.out.println("map: " + Thread.currentThread().getName());
            return x * 3;
        })
        .subscribe(x -> {
            System.out.println("observer: " + Thread.currentThread().getName());
            System.out.println(x);
        });
```

더욱이, 아래처럼 도중에 `observeOn` 메소드를 호출하면, 그 이후의 처리를 또다른 별도 쓰레드에도 동작시킨다.
```java
Observable.range(1, 10)
        .subscribeOn(Schedulers.computation())
        .filter(x -> {
            System.out.println("filter: " + Thread.currentThread().getName());
            return x % 2 == 0;
        })
        .observeOn(Schedulers.newThread())
        .map(x -> {
            System.out.println("map: " + Thread.currentThread().getName());
            return x * 3;
        })
        .subscribe(x -> {
            System.out.println("observer: " + Thread.currentThread().getName());
            System.out.println(x);
        });
```

## Observer와 Subscriber

Observable을 만들때 `create`메소드에 람다식을 넘겼었다. 실태는 OnSubscribe라는 함수형 인터페이스. 그렇기에 아래와 같이 작성할 수 도 있다.

```java
Observable<Integer> observable = 
    Observable.create(new Observable.OnSubscribe<Integer>() {
        @Override
        public void call(Subscriber<? super Integer> subscriber) {
            IntStream.range(1, 10).forEach(x -> subscriber.onNext(x));
            subscriber.onCompleted();
        }
    });
```

람다식의 인수는 실제는 Observer가 아니라 Subscriber. Subscriber는 Observer 인터페이스와 Subscription 인터페이스를 구현하고 있다.

`observable.subscribe(observer)`의 실행 반환 값도 Subscription.

Subscription에는 `unsubscribe`메소드와 `isUnsubscribe`메소드가 준비되어 있다. subscribe했을 때의 반환 값에 대해 `unsubscribe`메소드를 호출하면 통지를 수신하는 것을 그만둘 수 있다.

`isUnsubscribe`메소드에서 observer가 unsubscribe했는지 안했는지를 체크할 수 있다. 그리고, 명시적으로 `unsubscribe`메소드를 호출하지 않고도 unsubscribe될때가 있다. 예들 들어, 아래와 같이 `take`메소드를 사용해서 Observable로부터 5건만 취득하는 경우.

```java
Observable.range(1, 10)
    .take(5)
    .filter(x -> x % 2 == 0)
    .map(x -> x * 3)
    .subscribe(
        (x) -> System.out.println(x),
        (e) -> System.out.print("ERROR!"),
        ()  -> System.out.print("COMPLETE!")
    );
```

그렇기에, `Observable.create` 메소드를 사용할 때는, `onNext` 메소드 앞에 `isUnsubscribe`메소드에서 unsubscribe되고 있는지 어떤지를 체크하는 것이 좋다. 계속 발행되는 데이터인 경우의 Observable인 경우에는 체크하지 않으면 계속 동작하고 만다.

```java
Observable<Integer> observable = Observable.create(subscriber -> {
    int i = 0;
    while(true) {
        if (subscriber.isUnsubscribed()) {
            break;
        } else {
            if (i == Integer.MAX_VALUE) {
                subscriber.onCompleted();
            } else {
                subscriber.onNext(i);
                i = i + 1;
            }
        }
    }
});
observable.subscribeOn(Schedulers.computation())
    .take(5)
    .filter(x -> x % 2 == 0)
    .map(x -> x * 3)
    .subscribe(
        (x) -> System.out.println(x),
        (e) -> System.out.print("ERROR!"),
        ()  -> System.out.print("COMPLETE!")
    );
```

## RxJava를 사용한 예제
[7つのサンプルプログラムで学ぶRxJavaの挙動](http://techlife.cookpad.com/entry/2015/04/13/170000)

## 개념과 역사적 배경
[Reactive Porn - steps to phantasien](http://steps.dodgson.org/b/2014/12/07/reactive-porn/)

[関数型プログラマのための Rx 入門（前編）](http://okapies.hateblo.jp/entry/2015/03/04/031148)

[関数型プログラマのための Rx 入門（後編）](http://okapies.hateblo.jp/entry/2015/03/15/184247)

> Future/Promise가 단일 비동기 이벤트를 하나씩 처리하는 모델인것에 반하여 Rx의 Observable은 (시간, 순서가 있는)복수의 이벤트 스트림을 취급하는 처리를 대상으로 하고 있는 점이 다르다.

## HTTP 리퀘스트를 Observable로 만듬

- 단수 & 동기 => 값
- 복수 & 동기 => 배열
- 단수 & 비동기 => Future/Promise
- 복수 & 비동기 => Observable

HTTP 리퀘스트를 보내는 것을 상기의 4가지 패턴으로 해본다. HTTP 클라이언트는 OkHttp를 사용.

#### 1. 단수 & 동기
```java
public void exec(String url) throws IOException {
    Response = syncHttpCall(url);
    System.out.println("[" + url + "] OK!");
}

public Response syncHttpCall(String url) throws IOException {
    OkHttpClient client = new OkHttpClient();
    Request request = new Request.Builder()
            .url(url)
            .build();
    return client.newCall(request).execute();
}
```

#### 2. 복수 & 동기
```java
public void exec(List<String> urls) throws IOException {
    for (String url : urls) {
        syncHttpCall(url);
        System.out.println("[" + url + "] OK!");
    }
}
```

이것으로 복수로서 http 통신이 가능하다.

1개의 쓰레드에서 동작하고 있기때문에 response를 가지고 있는 시간이 아깝다. 리퀘스트를 던지면 결과가 올때까지 다른 HTTP 리퀘스트도 던지고 싶다. 따라서 비동기로 HTTP 통신을 해보도록 한다.

우선은 [단수 & 비동기]부터 생각해보자. Java8부터 사용가능해진 CompletableFuture로 비동기처리를 작성하는 것이 가능하다. CompletableFuture에 전달한 람다식은 메인쓰레드와는 별도의 쓰레드에서 동작한다.

```java
public void exec(List<String> urls) throws IOException {
    final String url = urls.get(0);
    CompletableFuture<Response> future = asyncHttpCall(url);
    future.thenAccept(response -> {
        System.out.println("[" + url + "] OK!");
    });
}

private CompletableFuture<Response> asyncHttpCall(String url) {
    CompletableFuture<Response> future = CompletableFuture.supplyAsync(() -> {
        OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder()
                .url(url)
                .build();
        try {
            return client.newCall(request).execute();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    });
    return future;
}
```

그럼, [복수 & 비동기]를 생각해보자. 똑같이 CompletableFuture을 사용해보자.

```java
public void exec(List<String> urls) throws IOException {
    for (final String url : urls) {
        CompletableFuture<Response>future = asyncHttpCall(url);
        future.thenAccept(response -> {
            System.out.println("[" + url + "] OK!");
        });
    }
}
```

이것으로 복수의 HTTP 리퀘스트를 비동기로 호출하는 것이 가능해졌다. 1개의 HTTP 리퀘스트를 던진 후, 리스펀스를 기다리지 않고, 별도의 쓰레드에서 두번째의 HTTP 리케스트를 던지는 것이 가능하다.

어라? [복수 & 비동기]도 CompletableFuture로 실현가능했다면 Observable은 불필요한 것일까?

여기에서 아래 문장을 상기해보자.

> Future/Promise가 단일의 비동기 이벤트를 하나씩 처리하는 모델인 것에 대해, Rx의 Observable은 (시간, 순서가 있는)복수 이벤트의 스트림을 취급하는 처리를 대상으로 하고 있는 점이 다르다.

```
(시간, 순서가 있는) 복수의 이벤트 스트림 이라고 적혀있다. 시간과 순서
```

CompletableFuture로 구현한 코드를 실제 동작시켜보면 알지만, 응답을 처리하는 순번은 제각각이다. 응답이 빠르게 돌아온 순으로 처리된다. 즉 순서는 의미가 없어지는 것이다.

그럼, Observable을 사용해서 [시간, 순서가 있는] 복수 & 비동기를 구현해보자. 
```java
public void exec(List<String> urls) throws IOException {
    Observable<Response> observable = Observable.empty();
    for (String url : urls) {
        observable = 
            observable.concatWith(Observable.from(asyncHttpCall(url)));
    }
    observable
            .subscribeOn(Schedulers.io())
            .observeOn(Schedulers.computation())
            .zipWith(Observable.from(urls), (r, s) -> new Pair<>(r, s))
            .map(pair -> "[" + pair._2 + "] OK! ")
            .subscribe(System.out::println);
}
```

Observable 오브젝트는 CompletableFuture로부터 만드는 것이 가능하다. 그리고 Observable 계열에서는 `concatWith`메소드를 사용하면 순번을 유지한채로 합성하는 것도 가능하다.

데이터를 생성하는 부분, 즉 CompletableFuture로부터 데이터를 추출하는 부분은 `Schedulers.io()`를 지정한다. 발생한 이벤트를 처리하는 부분, 즉 추출한 데이터를 처리하는 부분은 `Schedulers.computation()`을 지정한다.

이것으로 [시간, 순서가 있는] 복수 & 비동기 서비스를 만들었다.

게다가, Observable은 Observable을 메인쓰레드에서 사용하는것도 가능하고, 값을 1개씩 가지는 것도 가능하므로 4개의 패턴 모두에 대응한다고 할 수 있다.

## 여기까지의 정리
아래 3개를 나눠서 이해하는 것이 중요하다. 각각 독립적으로 생각할 수 있도록 한다.
- 어떻게 해서 Observable화 할 것인가
- Observable은 어떻게 조작하는가
- Scheduler의 설정은 어떻게 할까

Observable의 조작은 컬렉션에 대해 고차함수에서의 처리 (예를 들어 Java8의 Stream에 대한 처리)의 사고방식을 살릴 수 있기 때문에, 특별히 새로운 것이라고 할 수는 없다. 무엇을 어떻게 해서 Observable로 할까라는 부분이 Rx를 사용할때 중요한 부분이라 할 수 있을 것이다.

## 무엇이든지 Observable
[【翻訳】あなたが求めていたリアクティブプログラミング入門](http://ninjinkun.hatenablog.com/entry/introrxja)

## 새로운 등장인물 Subject
`RxJava의 기능 활용방법은 대체로 아래와 같이 분류할 수 있다.`
- List 처리의 추상화, 스트림화
- Optional
- Promise
- Data Binding
- Event Bus

[List 처리의 추상화, 스트림화]는 처음에 다뤘다. Promise도 CompletableFuture로부터 Observable을 만들어 본데서 다뤘다. [Optional]은 Observable에 값을 1개씩만 다룰까 empty니 할까로 해보면 알 수 있다.

[Data Binding], [Event Bus]는 어떻게 한다?라고 생각했더니 이런 식으로 적혀져 있었다.

> Data Binding, Event Bus는 Subject를 이용하기 때문에, Observable을 알고나서 다루는 것이 좋다.

Subject는 Observable로서도, Observer로서도 동작한다라고 적혀있는데, JavaDoc을 보면 Observable을 상속하고 Observer를 구현하고 있다. Observer 패턴의 Subject와는 다르게 보인다.

#### Cold Observable과 Hot Observable
공식 사이트를 읽어보면, Subject를 사용하면 원래의 Cold Observable을 변형한 Hot Observable을 얻을 수 있다.라고 적혀있다.

[RxのHotとColdについて - Qiita](http://qiita.com/toRisouP/items/f6088963037bfda658d3)

Hot Observable은 Observer가 등록되어 있지 않아도 데이타를 발행한다. 1, 2, 3이라는 데이터를 흐르게하는 Observable이 있다고 하고, 1과 2 사이에 Observer가 subscribe된다라고 한다면 그 Observer는 2, 3을 수신한다. (1은 이미 흘러 지나가 버렸기 때문)

이것이 이해되면 공식 사이트의 Subject 페이지의 그림도 이해가 쉽다. Subject에는 4종류가 있고, 제각각 데이터를 Observer에 전달하는 방법이 다르다.

더욱이 Hot Observable은 분기하는것이 가능하다고 적혀있다.

예를 들면, 아래와 같이 표준입력에서 입력된 문자열을 스트림의 데이터로 하는 Observable을 준비한다. 이것은 Cold Observable이 된다.

```java
Observable<String> observable = Observable.create(subscriber -> {
    InputStreamReader inputStreamReader = 
        new InputStreamReader(System.in);
    try (BufferedReader in = 
        new BufferedReader(inputStreamReader)) {
            while (true) {
                if (subscriber.isUnsubscribed() == false) {
                    System.out.print("> ");
                    String s = new String(in.readLine());
                    subscriber.onNext(s);
                } else {
                    break;
                }
            }
    } catch (IOException e) {
        subscriber.onError(e);
    }
}
```

Cold Observable은 두개의 Observer를 subscribe해도 1곳만 출력된다.

```java
observable.subscribe(System.out::println);
observable.subscribe(System.out::println);
```

그럼 중간에 Subject를 끼워넣어보자. Subject는 Observable이기때문에, Observer를 `subscribe`메소드로 취득가능하다. 또한, Subject는 Observer이기도 하기 때문에 Observable의 `subscribe`메소드로 전해줄 수 도 있다.

```java
PublishSubject<String> subject = PublishSubject.create();
subject.subscribe(System.out::println);
subject.subscribe(System.out::println);
observable.subscribe(subject);
```

Subject를 중간에 끼워넣으면 표준출력에 제대로 2회 출력되는 것을 알수있다. 왜냐하면 Subject를 Cold Observable의 `subscribe` 메소드에 전달하는 것으로, Hot Observable로 바뀌었기 때문이다.

덧붙여서 일부러 Observable과 Subject를 연결하지 않고, Subject 자체로도 같은 것이 가능하다.

```java
PublishSubject<String> subject = PublishSubject.create();
subject.subscribe(System.out::println);
subject.subscribe(System.out::println);
InputStreamReader inputStreamReader = new InputStreamReader(System.in);
try (BufferedReader in = new BufferedReader(inputStreamReader)) {
    while (true) {
        System.out.print("> ");
        String s = new String(in.readLine());
        subject.onNext(s);
    }
} catch (IOException e) {
        subject.onError(e);
}
```

#### PublishSubjectを使ってEvent Bus
그럼, 스트림을 분기하는 것과 PublishSubject를 사용한 것으로 Event Bus처럼 동작한 것을 알 수 있다. 표준입력이라는 이벤트를 Observable에 표현하고, 이벤트 핸들러를 복수 등록하고 있다.

PublishSubject를 사용하는 이류를 적어본다. 아래와 같이 `add`라는 문자열을 표준입력에 입력한 경우에 Observer를 추가하도록 하고있다.

```java
PublishSubject<String> subject = PublishSubject.create();
subject.subscribe(s -> {
    if (s.equalsIgnoreCase("add")) {
        observerCount = observerCount + 1;
        int observerId = observerCount;
        subject.subscribe(string -> 
            System.out.println("[" + observerId + "] " + string));
    }
});
observable.subscribe(subject);
```

아래와 같이 입력하면 Observer가 등록되기 전의 입력문자열은 무시되고, 등록 후의 입력문자열만이 출력되는 것을 알 수 있다.

```java
> hoge
> add
> fuga
[1] fuga
> foo
[1] foo
> add
[1] add
> bar
[1] bar
[2] bar
```

이것은 PublishSubject의 덕분. 이것이 ReplaySubject를 사용하느 경우에는 아래와 같이 된다. Event Bus의 거동치고는 이상하다.

```java
> hoge
> add
[1] hoge
[1] add
> fuga
[1] fuga
> foo
[1] foo
> add
[2] hoge
[2] add
[2] fuga
[2] foo
[2] add
[1] add
> bar
[1] bar
[2] bar
```

#### BehaviorSubject를 사용해서 Data Binding
남은 것이 Data Binding인데, 오브젝트의 프로퍼티를 변경했을 때 이벤트가 발생하는 프레임워크와 잘 사용하면 좋을까나? 어쨌든 변경대상의 오브젝트를 `ViewModel`이라는 클래스로 랩핑해서 작성해보자.

```java
private static class View {
    public String value;
}

private static class ViewModel {
    private BehaviorSubject<String> behaviorSubject;
    private String s;

    public ViewModel(String s) {
        this.s = s;
        this.behaviorSubject = BehaviorSubject.create(s);
    }

    public String get() {
        return s;
    }

    public void set(String s) {
        this.s = s;
        behaviorSubject.onNext(s);
    }

    public void bind(View view) {
        behaviorSubject.subscribe(s -> view.value = s);
    }
}

public static void main(String[] args) {

    ViewModel viewModel = new ViewModel("default");

    View view1 = new View();
    System.out.println("######################");
    System.out.println("view1: " + view1.value);

    viewModel.bind(view1);

    System.out.println("######################");
    System.out.println("view1: " + view1.value);

    viewModel.set("hoge");

    System.out.println("######################");
    System.out.println("view1: " + view1.value);

    View view2 = new View();
    viewModel.bind(view2);
    System.out.println("######################");
    System.out.println("view1: " + view1.value);
    System.out.println("view2: " + view2.value);

    viewModel.set("fuga");

    System.out.println("######################");
    System.out.println("view1: " + view1.value);
    System.out.println("view2: " + view2.value);
}
```

ViewModel에 View 오브젝트를 바인드해 둔다. 그러면 ViewModel의 프로퍼티를 바꿔적으면, 그것이 View에도 반영된다.

```java
######################
view1: null
######################
view1: default
######################
view1: hoge
######################
view1: hoge
view2: hoge
######################
view1: fuga
view2: fuga
```

## 정리
- 리액티브 프로그래밍에는 여러 종류들이 있다. 리액티브 익스텐션이라고 확실히 말하자.
- Rx는 [시간, 순서가 있는 `연속적` 데이터(스트림)]을 이벤트 핸들러로 처리하는 것.
- 데이터의 생성, 그 데이터 생성에 따른 이벤트 통지, 이벤트의 처리를 분리해서 기술한다.
- Observable 오브젝트가 [데이터의 생성]과 [데이터 생성에 따른 이벤트의 통지]를 담당한다.
- Observer 오브젝트가 [이벤트 처리]를 담당한다.
- Observable은 컬렉션에 대해 고차함수와 같은 조작이 가능하다.
- Observable 계열은 합성할 수 있다.
- Scheduler를 사용해서 동작하는 쓰레드를 지정할 수 있다.
- Rx를 사용하면 비동기 처리가 보다 쉽다.
- 비동기 처리와 동기 처리를 같은 것처럼 Observable에 대한 조작이 가능하다.
- Observable에는 Hot Observable과 Cold Observable이 있다.
- Hot Observable은 Subject를 사용하여 만든다.
- Subject를 사용하는 것으로 Event Bus나 Data Binding도 구현 가능하다.
- 모든 것이 Observable화 가능...!!!