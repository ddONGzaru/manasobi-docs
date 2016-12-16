Rx의 Hot과 Cold에 관하여
=============

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square) ![badge](https://img.shields.io/badge/manasobi-RxAndroid-yellowgreen.svg?style=flat-square)

###### Qiita [toRisouP Edited at 2016-01-29](http://qiita.com/toRisouP/items/f6088963037bfda658d3)

Rx의 IObservable<T>에는 Hot/Cold라는 큰 두가지 특징이 있다. 이것들을 이해하지 못한채 스트림을 설계하면, 의도한대로 동작하지 않는 경우가 생긴다. 이번 회에서는 Hot/Cold의 성질에 관해 간단하게 정리해 보자.

# 개요

## 한마디로 말해보면...
- **Cold** 스트림의 전후를 연결하는 것이 끝인 파이프. 혼자로서는 의미가 없다. 대체로 오퍼레이터들이 여기에 해당.
- **Hot** 스트림으로부터 값을 계속 발행하는 수도꼭지. 항상 값이 흐름. 뒤로 많은 파이프들 연결가능.

## 좀더 자세한 설명

###### Cold Observable
- 자발적으로는 아무것도 하지않는 Observable
- Observer가 등록되어(Subscribe되어) 최초로 일을 시작함.
- 스트림 전후를 연결하는 것 뿐임. 스트림을 분기, 나누는 것등의 기능은 없음.

###### Hot Observable
- 스스로 값을 발행하는 능동적인 Observable
- 후속 Observer의 존재에 관계없이 메시지를 발행.
- 자기보다 상위의 Cold Observable을 기동시켜, 값의 발행을 요구하는 기능을 가짐.
- 하위의 Observer를 전부 묶어서, 한꺼번에 같은 값을 발행함.(스트림을 분기, 나눔, 브로드캐스팅)

## Hot과 Cold의 분류방법
대부분의 오퍼레이터는 Cold한 성질을 가지고, 자신이 명시적으로 스트림을 Hot 변환하지않는 이상 Cold인채로 있다. Hot 변환용 오퍼레이터에는 Publish 계의 오퍼레이터가 해당한다.

# Hot에 관하여

## Hot Observable의 성질

#### 스트림을 가동시키는 성질
Rx의 스트림은 기본적으로 Subscribe되는 순간에 각 오퍼레이터의 동작이 시작되도록 되어있다. 하지만, Hot Observable을 스트림 도중에 끼워넣는 것으로 인해, Subscribe를 실행하기전에 스트림을 가동시키는 것이 가능하다.

#### 스트림을 분기하는 성질
Hot Observable은 **스트림을 분기**하는 것이 가능하다.

![alt](https://qiita-image-store.s3.amazonaws.com/0/47146/c2417fdd-4c92-2cd8-a3b4-3483a09eba90.png)

# Cold에 관하여

## Subscribe될때까지 도작하지 않는 성질
Cold Observable은 Subscribe(Hot 변환되는)될 때까지 동작하지 않는 Observable이다. 가동하지 않는 Cold Observable에 넘겨진 메시지는 전부 처리되지 않고 사라져버린다.

![alt](https://qiita-image-store.s3.amazonaws.com/0/47146/4c408984-4afa-32a0-ba53-c934e8af6651.png)

특히 값의 발행 타이밍이나 전후관계가 중요한 오퍼레이터를 사용하는 경우에, 어느 타이밍으로부터 처리가 시작되는 것인가를 충분히 인식하지 않고 사용하지 않으면 안된다. 같은 스트림 정의라고 해도, Subscribe한 타이밍에 의해 동작이 변하고 만다.

> Cold 구동
```java
var subject = new Subject<string>();

//subjectから生成されたObservableは【Hot】
var sourceObservable = subject.AsObservable();

//ストリームに流れてきた文字列を連結して新しい文字列にするストリーム
//Scan()は【Cold】
var stringObservable = sourceObservable.Scan((p, c) => p + c);

//ストリームに値を流す
subject.OnNext("A");
subject.OnNext("B");

//ストリームに値を流した後にSubscribe
stringObservable.Subscribe(Console.WriteLine);

//Subscribe後にストリームに値を流す
subject.OnNext("C");

//完了
subject.OnCompleted();

```
```
実行結果
C
```
위 코드를 실행한 결과는 `C`가 출력되는 것을 알 수 있다. 이것은 `Sacn`오퍼레이터가 `Cold`이기 때문에, Subscribe 전에 발행된 A, B는 처리되지 않았기 때문이다.

만약 여기에서 [Subscribe하기 전에 발행된 값도 처리하고 싶다]라고 한다면 Hot 변환 오퍼레이터를 끼워넣어 Subscribe하기 전에 스트림을 기동해두면 된다.

```java
Hot変換を挟んだ場合の挙動
var subject = new Subject<string>();

//subjectから生成されたObservableは【Hot】
var sourceObservable = subject.AsObservable();

//ストリームに流れてきた文字列を連結するストリーム
//Scan()は【Cold】
var stringObservable = sourceObservable
    .Scan((p, c) => p + c)
    .Publish(); //Hot変換オペレータ

stringObservable.Connect(); //ストリーム稼働開始

//ストリームに値を流す
subject.OnNext("A");
subject.OnNext("B");

//ストリームに値を流した後にSubscribe
StringObservable.Subscribe(Console.WriteLine);

//Subscribe後にストリームに値を流す
subject.OnNext("C");

//完了
subject.OnCompleted();
```
```
実行結果
ABC
```

`Publish`라는 Hot 변환 오퍼레이터를 중간에 끼워넣는 것으로 Subscribe하기 전에 스트림을 강제 기동시키는 것이 가능하다.

## 각각의 Observer에 관해 별도의 처리를 함(스트림의 분기점은 되지 않음)
Cold Observable은 스트림을 분기시키는 성질은 가지고 있지 않다. 그렇기에, Cold Observable을 복수 Subscribe한 경우, 각각에 별도의 스트림이 생성되어 할당되어 버린다.

![alt](https://qiita-image-store.s3.amazonaws.com/0/47146/86b8a6ed-c6c1-582c-eef6-7ca4b5ff042d.png)

하지만 스트림에 Hot Observable이 존재한 경우, 제일 말단에서 가까운 Hot Observable에 스트림이 분기되어, 더더욱 별도의 스트림이 생성된다.

![alt](https://qiita-image-store.s3.amazonaws.com/0/47146/37598a0a-fb25-42c5-505e-f83e240b1101.png)

# 마무리
스트림이 어디서 분기해아 하는가를 항상 의식하면서 설계하는 것이 중요하다. [클래스 외부에 Observable를 공개할 때 Cold인 채로 공개하지 않고, 반드시 말단에 Hot 변환하고 공개]등 의도하지 않은 곳에서 분기시켜버리는 실수를 하지 않도록 주의해야 한다.

[【Reactive Extensions】 Hot 변환은 언제 필요한 것일까?](http://qiita.com/toRisouP/items/c955e36610134c05c860)

