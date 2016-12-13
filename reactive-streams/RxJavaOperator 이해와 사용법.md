RxJavaOperator 이해와 사용법
===========================

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square)

###### Qiita [Yuki_Yamada Edited at 2016-10-27](http://qiita.com/Yuki_Yamada/items/627c4bc25956d421cb6a)

# 1. 생성

## 1.1 create
![img](https://qiita-image-store.s3.amazonaws.com/0/59803/6f134358-2b95-6a0d-8d5c-67f2da6ef5cc.png)

[ReactiveX - Create operator][create]
각각의 Observable의 subscribe에 의해 Observable을 생성하는 오퍼레이터.
비동기처리에서 에러 핸들링이 필요할 때 사용.

``` 
Observable<String> observable = Observable.create(
    new Observable.OnSubscribe<String>() {
        @Override
        public void call(Subscriber<? super String> subscriber) {
            try {
                FileInputStream fileInputStream;
                fileInputStream = openFileInput("MyFile.txt");
                byte[] readBytes = new byte[fileInputStream.available()];
                fileInputStream.read(readBytes);
                subscriber.onNext(new String(readBytes));

                fileInputStream = openFileInput("2ndMyFile.txt");
                byte[] readBytes2nd = new byte[fileInputStream.available()];
                fileInputStream.read(readBytes2nd);
                subscriber.onNext(new String(readBytes2nd));

                subscriber.onCompleted();
            } catch (Exception e) {
                subscriber.onError(e);
            }
        }
    }
```

create를 사용하는 경우는 예최처리가 발생하는 경우라고 생각한다. 자기가 subscribe 발화를 제어하는 느낌이다.
공식 샘플에는 for문에서 observable을 이용하고 있는데 Iterator를 가지고 있지 않는 연속된 데이타를 Observable로 하고 싶은 경우에 사용한다고 할 수 있겠다.

## 1.2 just
Observable로 하는 값을 직접 지정.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/ad87affa-e655-913c-8e3e-f5ee3efff13f.png)

[ReactiveX - Just operator][just]
그림에는 하나밖에 Observable에 흐르고 있지 않지만, 10개까지 지정 가능하다.

``` java
Observable.just(1,2,3,4,5,6,7,8,9)
```
테스트 등에 사용될 수 있다.

## 1.3 repeat
onCompleted를 통과했지만 한번더 Subscribe를 해준다.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/38cf97a6-4dd7-3da0-11a8-9c02865043b7.png)

빨강, 초록, 파랑을 Observable로 흐르게하고, repeatOperator를 통과했기때문에 같은 데이터가 반복해서 subscribe된다.

```
Observable.just(1,2,3,4,5,6,7,8,9)
          .repeat(5)
```

## 1.4 repeatWhen
repeat와 유사 함수로 repeat하는 간격을 지정.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/bed81c86-2f40-d8a0-16fe-f8cc4b1d89f1.png)

데이터는 노란, 빨간 공. 이것이 막대기에서 종료하면 파란 마름모가 발행되고 delay시간이 포함되어 한번 더 subscribe된다.

```
Observable
    .just(1,2,3,4,5,6,7,8,9)
    .repeatWhen(new Func1<Observable<? extends Void>, Observable<?>>() {
        @Override
        public Observable<?> call(Observable<? extends Void> observable) {
            return observable.delay(5, TimeUnit.DAYS);
        }
    });
```

## 1.5 Empty/Never/Throw
 - Empty는 onComplete만 발행한다.
 - Never는 그것조차 호출하지 않는 공백의 Observable이다.
 - Throw는 onError를 발행하는 Observable를 발행한다.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/1e56363c-d625-ab0e-e914-f125148c611b.png) 
최후에 발화하고 있다. 이것이 OnCompleted이다.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/db24a34e-b92e-4fc5-8747-f274c25450f8.png)
보는 것처럼 발화조차 하지 않는다.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/ac37a879-4a6c-5d4e-169e-45a2f5ed4342.png)
에러의 발화를 하고있다.

## 1.6 from
Iterator를 가진 오브젝트로부터 Observable을 생성한다.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/4dfc1476-9970-b4bb-1efc-c3717210c17a.png)

```
String[] array = new String[]{"sasaki","ササキ","ささき","佐々木"};
Observable.from(array)
```

맵으로부터 데이터를 취득하는 경우는 아래와 같다.

```
Observable.from(map.entrySet())
```

## 1.7 range
int로 시작, 종료를 지정하고 그 사이의 숫자의 Observable을 생성한다.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/c4a898a9-35c7-f1e9-4f3e-88408c52cf12.png)

```
Observable.range(1,10);
```

# 2. 필터링
흘러들어온 데이터에 대하여 onNext를 발행할까 어떨까라는 처리를 주로하는 오퍼레이터이다.

## 2.1 filter

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/ea70f5e1-567b-75cf-6043-71fd27d7387e.png)

[ReactiveX - Filter operator][filter]
이 그림에서는 전달된 Observer로부터 동그라미만을 필터링해서, 동그라미 모양만을 onNext한다.

```
Observable.just(1,10,4,21,19,2,13,9)
    .filter(new Func1<Integer, Boolean>() {
        @Override
        public Boolean call(Integer item) {
            return (item < 15);
        }
    })
```

함수 Func1에서 반환 값이 true이면 그 값이 onNext에 도달한다.

## 2.2 take
개수를 지정해서 꺼낸다.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/2fd96bcd-967b-37cc-9810-ce69cf80d26d.png)

[ReactiveX - Take operator][take]
```
Observable.just(1,10,4,21,19,2,13,9)
          .take(2)
```

takeLast를 이용하면 마지막 데이터로부터 데이터를 취득한다.
```
Observable.range(1,10)
          .takeLast(3)
```

## 2.3 first/last/elementAt ..OrDefault
최초(최후, 또는 지정한 곳)만을 취득해서 onNext한다. 요소 하나만을 onNext하는 것이 특징이다.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/c04da8d4-b54d-0bbe-887c-e903eba9acba.png)
최초의 빨간색만을 Observable에 흘려보낸다.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/d3f4af4f-9322-7e8a-53ad-0947a015964d.png)
제일 마지막 핑크색만을 Observable로 보낸다.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/359ca566-488b-b2bf-a7a2-5a902814556e.png)
지정한 요소 번호 2번만을 Observable로 보낸다.

```
Observable.just(1,2,3,4,5,6).first();
Observable.just(1,2,3,4,5,6).last();
Observable.just(1,2,3,4,5,6).firstOrDefault(1);
```

OrDefault가 있어서 안전한 설계가 가능하다.
```
Observable.just(1,2,3,4,5,6).elementAt(3);
Observable.just(1,2,3,4,5,6).elementAtOrDefault(2,11);
```

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/074f2f3d-14e0-8ce6-a0a9-0731991d324c.png)
요소 5번이 없더라도 예외가 발생하지않고 Default 값을 Observable에 값을 전달한다.

## 2.4 sample
일정 시간마다 스트림의 값을 취득하여 전달한다. 인수는 `sample(long,TimeUnit)`이다.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/7354765c-bada-7283-d649-24a9a96e6a02.png)
일정 시간이 흐르면 직전 스트림에서 흐르고있던 값을 subscribe한다.

```
Observable.interval(1,TimeUnit.DAYS)
          .sample(2,TimeUnit.DAYS)
```
매일마다 Observable에 데이터가 흘러들어오는 것을 이틀에 한번 onNext한다는 의미이다.

## 2.5 throttle First/Last
인수는 sample과 같은 `throttleFirst(long,TimeUnit)`이다. 일정시간, onNext는 하지않는 경우에 사용한다.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/b4d3ec90-6758-4d91-9c09-f545d8f697d9.png)

```
Observable.interval(1,TimeUnit.DAYS)
          .throttleFirst(3,TimeUnit.DAYS)
```

## 2.6 distinct
중복은 제거하고 onNext로 흘려보낸다.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/27faff06-9132-5f1c-87a1-ae58225efc0e.png)

```
Observable.just(1,2,2,2,1,3,4,1)
    .distinct()
    .subscribe(
        new Observer<Integer>() {
            @Override
            public void onCompleted() {
                Log.d("onCompleted", "owari");
            }

            @Override
            public void onError(Throwable e) {
                e.printStackTrace();
            }

            @Override
            public void onNext(Integer integer) {
                Log.d("onNext", String.valueOf(integer));
            }
        }
    );
```

## 2.7 DistinctUntilChanged
중복이있는 경우에는 하나로 묶는다(여기까지는 Distinct와 동일). 그러나, 다른 값의 입력이 있는 경우에는 별도로 한다.

![img](https://qiita-image-store.s3.amazonaws.com/0/59803/dc808388-01ea-a8b3-2553-56d868106334.png)

```
Observable.just(1,2,2,2,1,2,2,3,3,1)
          .distinctUntilChanged()
```
```
[결과]
10-07 21:10:14.459 23959-23959/com.example.yukin.rxjava1st D/onNext: 1
10-07 21:10:14.459 23959-23959/com.example.yukin.rxjava1st D/onNext: 2
10-07 21:10:14.459 23959-23959/com.example.yukin.rxjava1st D/onNext: 1
10-07 21:10:14.459 23959-23959/com.example.yukin.rxjava1st D/onNext: 2
10-07 21:10:14.459 23959-23959/com.example.yukin.rxjava1st D/onNext: 3
10-07 21:10:14.459 23959-23959/com.example.yukin.rxjava1st D/onNext: 1
```

## 2.8 Debounce





