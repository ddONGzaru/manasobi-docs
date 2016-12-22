Rx에서 알아두면 편리한 Subject들
=============

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square) ![badge](https://img.shields.io/badge/manasobi-RxAndroid-yellowgreen.svg?style=flat-square)

###### Qiita [ralph Edited at 2016-09-05](http://qiita.com/ralph/items/f7205c8171826cc2153b)

RxJava/RxAndroid의 소개 기사중에는 자주 [귀찮은 AsyncTask를 사용하지 않고도 해결]이라고 RxJava를 비동기처리의 대체로서 소개하고 있다.

물론 비동기처리를 대체할 수 있다는 점은 RxJava의 유효한 사용방법 중의 하나이지만, RxJava에는 그 밖에도 편리하게 사용할 수 있는 기능들이 있다.

이 기사에는 그것들의 기능 중의 하나인 Subject에 관하여 소개한다.

# Subject란?

간단히 말하면 `Subscriber`와 `Observable` 두가지 기능을 모아서 가지고 있는 것이라고 하겠다.
`Subscriber`에 있는 `onNext`, `onError`, `onComplete`등의 메소드를 호출하고, `Observable`처럼 `subscribe`메소드를 호출하는 것도 가능하다.

```java
Subject<String, String> subject = BehavorSubject.create();
subject.onNext("Hoge");
subject.subscribe(System.out::println); // "Hoge"が標準出力へ出力される
```

`Observable.create()`를 사용한 Observable의 생성 방법이라면, 임의의 타이밍에서 `onNext`등을 호출하는 것이 상당히 귀찮게 되어 있지만, Subject에서는 자신에 대해서 `onNext`등을 호출하는 것이 가능하기 때문에 임의의 타이밍에서의 호출이 간단해진다.

# Subject의 종류와 특징
Subject에는 `onNext`에서 발행된 데이터들을 어떻게 처리할 것인지에 대해 여러 Subject가 존재한다. 각각에 대한 사용법을 알아봄으로써 지금까지 복잡하게 기술할 수 밖에 없었던 것들을 짧고 명료하게 적는것이 가능해진다.

## AsyncSubject
![alt](https://qiita-image-store.s3.amazonaws.com/0/29459/92d3d793-f883-98ff-8681-0f17438abfcf.png)

AsyncSubject에서는 AsyncSubject측의 `onComplete`가 호출된 직후에, 최후의 `onNext`에서 전달받은 값만 Subscriber의 `onNext`의 인수로 전달된다.

![alt](https://qiita-image-store.s3.amazonaws.com/0/29459/631fa0fd-d918-2685-f1c2-58071068613f.png)

도중에 AsyncSubject측의 `onError`가 호출되는 경우에는 Subscriber의 `onNext`는 호출되지 않고 `onError`가 호출된다.

#### 사용 예
onNext에서 호출되는 것이 최후의 요소뿐으로, onCompleted를 부르지않으면 요수가 흘러들어오지 않는다는 것이 조금 특징이라고 할수있으나, 값이 하나만 흘러들어오지 않는다거나 최후의 하나밖에 필요 없을때, 콜백을 수신하는 구현을 하기에 편리하다.

예를 들어 아래와 같은 경우를 AsyncSubject로 구현하고 싶다고해보자. (구현은 기본적으로 RxAndroid를 사용)

> 화면이 표시되는 것보다 전에 데이터 취득을 시작함.<br>
> 화면이 표시되는 순간에 데이터 취득이 끝나있으면 그 데이터를 표시함.<br>
> 끝나있지 않으면 로드 중의 인디케이터를 표시하고, 데이터 취득이 완료하자마자 데이터를 표시.

`Initializer.java`
```java
// Applicationクラス(HogeApplicationとする)のonCreateでインスタンス化しておく。
public class Initializer {
    public AsyncSubject<Data> asyncSubject = AsyncSubject.create();

    public Initializer(){
        generateObservable()
                .subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(data -> {
                    asyncSubject.onNext(data);
                    asyncSubject.onCompleted();
                });
    }

    public Observable<Data> generateObservable(){
        return Observable.create(subscriber -> 
                            subscriber.onNext(longTask()));
    }

    public Data longTask() {
        //時間のかかる処理 (ネットワークからのデータ取得など)
    }
}

```

`SubActivity.java`
```java
// MainActivityでボタンをタップしたりした後に呼ばれると仮定
public class SubActivity extends RxActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_sub);

        // ここでダイアログを表示させる
        AlertDialog dialog = ......;
        dialog.show();

        // HogeApplication#getInitializer() が上のInitializerのGetter
        ((HogeApplication) getApplication()).getInitializer().asyncSubject
                .compose(bindToLifecycle()) // AndroidのライフサイクルにBind
                .subscribe(data -> {
                    // dataをリストに表示したりする
                    dialog.dismiss();
                }, throwable -> {
                    // エラーが出たことを表示する
                    dialog.dismiss();
                });
    }
}
```

화면을 표시하기 전에 데이터 취득이 끝난 경우와, 그렇지 않은 경우 모두 하나의 코드에 대응하도록 만들어졌다.

## BehavorSubject

![alt](https://qiita-image-store.s3.amazonaws.com/0/29459/9322a7ef-325a-068f-7202-ddf39aa54ccb.png)

직전에 `onNext`에서 넘겨받은 값을 가지고 있고, `subscribe()`한 직후에 보존하고 있던 값을 넘긴다. 그 후의 동작은 뒤에 나올 PublishObject와 동일하다.

![alt](https://qiita-image-store.s3.amazonaws.com/0/29459/f57cb952-f802-243c-f98e-1dafb6bcb359.png)

한번 `onError`에서 예외가 던져지면, `onNext`는 무시되고 `subscribe()`후에 Subscriber측의 `onError`가 호출된다.

#### 사용 예
직전에 `onNext`에서 넘긴 데이터가 넘어 온다는 점에서, 사용 범위가 조금 한정되어지지만 RxLifecycle에서 Activity, Fragment의 라이브사이클을 취득, 판별하는 부분에서 사용되고 있다는 것이 가장 확실한 사용방법이라 할 수 있을것이다.

[RxLifecycleの紹介](http://qiita.com/hide92795/items/02533c8593ce51070b57)

`RxLifecycle.java`
```java
return new Observable.Transformer<T, T>() {
    @Override
    public Observable<T> call(Observable<T> source) {
        return source.takeUntil(
            Observable.combineLatest(
                lifecycle.take(1).map(correspondingEvents),
                lifecycle.skip(1),
                new Func2<R, R, Boolean>() {
                    @Override
                    public Boolean call(R bindUntilEvent, R lifecycleEvent) {
                        return lifecycleEvent == bindUntilEvent;
                    }
                })
                .onErrorReturn(RESUME_FUNCTION)
                .takeFirst(SHOULD_COMPLETE)
        );
    }
};
```
상기 코드는 RxLifecycle의 RxLifecycle.java로부터 일부 발췌, 수정을 더한 것이다.

`source`는 라이프사이클을 적용한 Observable. `correspondingEvents`은 Activity, Fragment의 이벤트에 대해서, 이벤트에 변환을 수행하는 Function(예: `OnCreate`라면 `OnDestroy`로 변환)

`lifecycle`은 대상 Activity의 라이브사이클 상태가 흘러들어오는 **BehavorSubject**이다.

코드를 보면 `lifecycle.take(1).map(correspondingEvents)`에서 바인드한 라이프사이클에 대해 종료 이벤트를 취득하고, `lifecycle.skip(1)`와 `combineLatest`로 연결하는 것으로 `lifecycle`에 

