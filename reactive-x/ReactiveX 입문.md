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



