RxJava를 사용해서 좋았던 점 2가지
=============

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square) ![badge](https://img.shields.io/badge/manasobi-RxAndroid-yellowgreen.svg?style=flat-square)

###### Qiita [oxsoft Edited at 2016-05-12](http://qiita.com/oxsoft/items/9ae07c5512449b15b923)

# RxAndroid란?
RxAndroid는, ReactiveX의 안드로이드 버전으로, 옵저버 패턴을 간단히 구현한 버전이다. 

- 비동기처리가 쉬움.
- 복수개수(View 등에서의)에서 최신 정보를 표시 및 갱신등이 쉬움.

> build.gradle
```java
compile 'io.reactivex:rxjava:1.1.5'
compile 'io.reactivex:rxandroid:1.2.0'
```

# 비동기처리가 간단함

## AsyncTask의 경우
대부분의 앱에서 통신 처리를 하지만 AsyncTask드을 사용하면 번잡한 프로그래밍이 되어버린다.

```java
new AsyncTask<Params, Progress, Result>() {
    @Override
    protected Result doInBackground(Params... params) {
        // 非同期処理
        return new Result();
    }

    @Override
    protected void onPostExecute(Result result) {
        // 終わった後の処理
    }
}.execute();
```

왜 번잡하고 복잡해지는지는 아래와 같다.
- 비동기처리와 종료 후의 처리를 같은 빌딩안에 적지 않으면 안된다.
- 종료 후의 처리에 참조가 남은 채로 있다(= Null Pointer Exception이 발생할 확률이 높음)

## RxAndroid 

```java
Single.create(new Single.OnSubscribe<Result>() {
    @Override
    public void call(
        SingleSubscriber<? super Result> singleSubscriber) {
            // 非同期処理
            singleSubscriber.onSuccess(new Result());
        }
    })
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Action1<Result>() {
        @Override
        public void call(Result result) {
        // 終わった後の処理
        }
    });
```    

얼핏 보기엔 AsyncTask의 경우와 같은 것처럼 보이지만, 이렇게 같은 곳에서 처리를 하는 것은 드믈고, 많은 경우 Model과 Activity로 나누의 사용할 때 그 가치가 빛을 발한다.

#### 비동기처리(Model)

```java
public Single<Result> request(Params... params) {
    return Single.create(
        new Single.OnSubscribe<Result>() {
            @Override
            public void call(
                SingleSubscriber<? super Result> singleSubscriber) {
                // 非同期処理
                singleSubscriber.onSuccess(new Result());
            }
        })
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread());
}
```

#### 終わった後の処理(Activity)
```java
request().subscribe(new Action1<Result>() {
    @Override
    public void call(Result result) {
        // 終わった後の処理
    }
});
```

## 람다식
[Retrolambda](https://github.com/evant/gradle-retrolambda)나 [Jack (Java Android Compiler Kit)](https://source.android.com/source/jack.html)을 사용하는 것으로 Java8의 람다식이 사용 가능하다.

#### 비동기처리(Model)
```java
public Single<Result> request(Params... params) {
    return Single.create((Single.OnSubscribe<Result>) singleSubscriber -> {
        // 非同期処理
        singleSubscriber.onSuccess(new Result());
    }).subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread());
}
```

#### 終わった後の処理(Activity)
```java
request().subscribe(result -> {
    // 終わった後の処理
});
```

## 예외처리
통신처리의 경우, 도중에 네트워크가 끊기거나 하는등 예외처리는 필수이다. 이것도 간단하게 처리 가능하다.

#### 비동기처리(Model)
```java
public Single<Result> request(Params... params) {
    return Single.create(
        (Single.OnSubscribe<Result>) singleSubscriber -> {
            // 非同期処理
            try {
                singleSubscriber.onSuccess(new Result());
            } catch (Exception e) {
                singleSubscriber.onError(e);
            }
        })
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread());
}
```
#### 終わった後の処理(Activity)
```java
request().subscribe(result -> {
    // 終わった後の処理
}, throwable -> {
    // 例外処理
});
```

## NPE 회피
AsyncTask에서 자주 발생하는 NullPointerException도 RxAndroid를 사용하면 비교적 쉽게 피할 수 있다.

종료 후 처리에서 subscribe를 하면 subscription이라는 반환 값을 취득한다.

```java
Subscription subscription = request().subscribe();
```

RxAndroid에는 `CompositeSubscription`이라는 `List<Subscription>`클래스가 있기 때문에, 그것에 `add`해두고, onDestroy등에서 `clear`하면, 그 뒤로는 subscribe의 처리가 되지 않기때문에 NPE를 방지할 수 있다.

```java
CompositeSubscription compositeSubscription = 
    new CompositeSubscription();

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    compositeSubscription.add(request().subscribe());
}

@Override
protected void onDestroy() {
    compositeSubscription.clear();
    super.onDestroy();
}
```
(그러나, 다이얼로그 계의 경우에는 onStop~onDestroy 사이에서 실행하면 에러가 나는 경우가 있으므로 이 경우에는 주의가 필요하다.)

# 복수 장소(View 등에서)에서 최신 정보를 표시
RxAndroid의 또하나의 이점은 복수 장소에서 최신 정보를 표시하는 것이 가능하다는 것이다. Observer 패턴을 기본으로 하고 있다는 것을 생각하면 이것이야말로 가장 큰 이점이라 할 수 있겠다.

구체적은 유스케이스는 아래와 같다.

![alt](https://qiita-image-store.s3.amazonaws.com/0/92361/12bb9b4d-d56a-74d0-5377-0af355842cda.png) ![alt](https://qiita-image-store.s3.amazonaws.com/0/92361/d74f3303-d161-6fdd-f933-304cdeb4ae66.png) 

![alt](https://qiita-image-store.s3.amazonaws.com/0/92361/4f8412e6-2eec-123f-b2eb-8fb542245427.png) ![alt](https://qiita-image-store.s3.amazonaws.com/0/92361/7e2ae521-205b-625f-8972-0a2ec5d27f94.png)

리스트에서 요소를 탭해서 상세화면을 본 후, 즐겨찾기 버튼을 누른 다음, 백 키로 리스트에 돌아왔을 때, 즐겨찾기 버튼을 누른 것이 반영되어 있는 것을 알 수 있다.

Android의 경우 백 키로 전 화면에 돌아가는데 이런 과정 중에 정보가 오래되었거나, 매번 통신을 해야하거나 해서 [Web 같다~]라고 평가받을 수 있다.

이것을 Android로 구현하고자하면, 직접함수를 호출하거나 broadcast를 사용하는 등 번잡하다. RxAndroid에서는 `BehaviorSubject`를 사용하는 것으로 간단하게 해결가능하다.

## BehaviorSubject
BehaviorSubject는 간단히 말하면, 변수의 강화판이다.

통상, 변수는 set, get이 가능하고 아래와 같이 표현한다.
```java
// set
student.setName("foo");

// get
String name = student.getName();
```

그러나, 이런 경우에 `name`은 그 다음의 student의 setName이 호출되어도 변경되지 않는다.

```java
student.setName("foo");
String name = student.getName();
student.setName("bar");
Log.d("name", name); // fooのまま！
```

안드로이드의 경우 표시 화면은 DataBinding을 사용해도 되지만, RxAndroid로도 구현 가능하다.

```java
BehaviorSubject<String> studentName = BehaviorSubject.create();
// set
studentName.onNext("foo");
// get（即時）
String name = studentName.getValue();
// get（Observableとして）
Observable<String> observable = studentName.asObservable();
```

`BehaviorSubject`의 경우, 변수처럼 값을 꺼내는 `getValue` 외에 `Observable`로서 꺼내는 것도 가능하다. 추출한 `Observable`은 아래와 같이 사용한다.

```java
observable.subscribe(name -> {
    Log.d("name", name);
});
```

이렇게 하는 것으로, set 함수인 onNext를 호출할 때 마다 상기의 처리가 수행, 표시 화면의 값이 갱신된다.

## 주의사항

#### 상태(데이터)를 전부 모델에 둘 것
데이터를 전부 모델에 두고, Activity의 표시는 항상 모델을 따르도록 하지 않으면 현재의 표시가 어디에서 흘러왔는지 모르게 되어 버리는 경우가 있다. 또한, 여기저기 BehaviorSubject를 두고, 값의 갱신이 있으면 별도의 값도 바뀐다고 하면 무한 루프로 빠져버릴 수도 있으므로 주의가 필요하다.

#### 갱신을 수행한 후의 처리는 가능한 가볍게 할 것
값의 갱신이 있으면 수행되는 처리는 꽤 자주 호출되리라 생각되는데, 화면 갱신되는 메소드등에 정리해 두는 것이 무난하다.
통신처리가 있으면 뒤에서 쓸데없는 통신을 해버리기 때문에 피하는 것이 좋다.

또, Dialog나 Toast등을 표시하는 처리를 하면 Activity가 메인 화면이 아님에도 불구하고, Dialog나 Toast등이 표시되어 버리는 기묘한 상태가 되어버리므로 피하는 것이 좋다.







