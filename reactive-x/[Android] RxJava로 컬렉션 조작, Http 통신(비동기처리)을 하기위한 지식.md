[Android] RxJava로 컬렉션 조작, Http 통신(비동기처리)을 하기위한 지식
=================================================================

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square) ![badge](https://img.shields.io/badge/manasobi-android-orange.svg?style=flat-square)

###### Qiita [Yuki_Yamada Edited at 2016-10-25](http://qiita.com/Yuki_Yamada/items/a0855189988539c18b8f#_reference-e2a2024732be2ead5cc6)

## RxJava를 사용하는 이점
지금까지의 안드로이드 개발에는 없었던 사고방식, 조작이므로 러닝 커브가 높다라고 말하며 멀리하는 이들이 많을지도 모르지만, 지금은 그렇지 않다. 당연한듯이 사용되고 있는 샘플들도 많아졌기 때문에 마음껏 사용하고, 도전해봐도 좋지 않을까 한다. 그럼, RxJava를 사용하는 이점을 레퍼런스로서 소개해 본다.

## 비동기통신 처리
안드로이드는 통신처리를 백그라운드에서 처리하지않으면 안됨에도 불구하고, 스레드 관리가 힘들었다.
AsyncTask나 AsynctaskLoader등에 시달렸던 것도 사실이다. 비동기처리 기능들의 시간 대기등에 힘들었던 것 또한 사실이다.
비동기 처리 방법은 사람마다도 달라서 하나로 통일하기도 힘들었다.
Rx에는 강력한 오퍼레이터가 준비되어있기 때문에 구현 방법 통일뿐만 아니라, 이해도 하기 쉽다.

그리고, 또한가지 강력한 이점이 있다. 그것은 RxAndroid(RxLifecycle)을 이용하는 것으로 인해 Activity나 Fragment를 의식하지 않고 비동기 통신 처리가 가능하다는 점이다.

## Collection 조작
RxJava의 본질은 컬렉션 조작이다. 나중에 상세히 설명하겠지만, 스트림이라는 개념이 있다.
for문에서의 컬렉션 조작은 내용을 이해하기 위해 if문의 중첩을 읽고, 파라미터가 무엇을 의미하는지를 파악하고, 어떤 데이터를 취급하고 있는가, 등을 이해하기 위해 모든 코드를 읽지않으면 안된다.

그러나, RxJava를 이용한 컬렉션 조작에서는 모든 정보를 읽을 필요가 없다. 아까 언급한 operator가 그것을 명시적으로 나타내주기 때문이다.

## View의 event 제어
View의 이벤트 제어도 번잡하다. 자주 보는 예이지만, EditText등에 사용자명 등이 입력되는 것을 감시하고, [잘못된 문자열] [문자수 초과]등을 실시간으로 제어해야 한다. UI의 움직임을 스트림으로 취급함으로써 이런 조작이 가능해진다.

## Rx란 도데체 무엇인가?
무언가 굉장한게 최근에 등장한 것 같지만, 개념적으로서는 마이크로소프트사가 .Net용으로 제공한 Rx.NET이 기원이다.

## Reactive Programming이란?
기본적으로 리액티브 프로그래밍은 [객체지향 프로그래밍], [함수형 프로그래밍], [구조적 프로그래밍]등과 함께 기본적인 패러다임의 하나이다.

그럼 중요한 개념중의 하나인 [리액티브]는 무엇인가?

> dataflow에 따라 자동적으로 변경을 전달한다. 간단히 말하면 이벤트에 반응하는 것이라고 할 수 있다.

* 부하에 반응 
* 장애에 반응
* 유저 조작에 반응

> 부하나 장애에 반응하고, 유저에 대해 항상 인터랙티브한 서비스를 제공

안드로이드에서는 통신처리는 백그라운드에서 실행되어야만하고, 대부분 통신 처리후에는 리스트 조작등이 일어나기 때문에 상성이 꽤 좋다고 볼 수 있다.

이러한 리액티브 프로그래밍에 함수형 프로그래밍의 사고 방식을 더하여 Functional Reactive Programming이라고 부르는 접근 방식. 이러한 사고 방식에 근거한 Reactive Extensions이라고 불리는 라이브러리에서는 비동기 데이터 스트림을 수행하고 있다. 이것이 결과적으로 안드로이드에서의 비동기 처리를 수행하는 것이 되었다고 할 수 있다.

## 컬렉션 조작을 RxJava로 수행
비동기 통신(http) 처리에서 API를 사용한다고 해도 UI를 RxBinding한다고 해도 컬렉션 조작이 불가능하면 의미가 없다.
우선 컬렉션 조작부터 공부해보자.

## Observable
무언가를 설명하기 전에, 공식 문서나 그외 해설등에서 자주 등장하는 그림이 있다. 색, 도형, 화살표등으로 구성된 다이어그램이다. 그것의 의미를 파악하고 있으면 영어를 이해하지 않아도 공식 문서의 그림만으로 어떻게든 이해가 되니 꼭 그림에 대한 이해를 사전에 해두기로 하자.

![alt](https://qiita-image-store.s3.amazonaws.com/0/59803/e2295052-20a7-a5c0-f72c-5060c0fa2f1c.png)

우선 위의 막대와 도형. 그것들 전부해서 [Observable]이라고 한다. 화살표의 방향대로 시간이 흘러가고 있다는 의미다. 중간의 하얀 상자는 [Operator]라고 부른다. 이곳에서 값의 변형 등 여러가지 작업을 수행한다.

Observable은 그밖에도 [이벤트]인 경우도 있다. 이것이 View의 이벤트 처리등에 연관되어 온다. [비동기처리의 종료]등도 이벤트 중의 하나라고 취급할 수 있을 것이다.

## Observable의 생성

### range
```java
range(int, int)
```
첫번째 인수로 부터 두번째 인수까지의 수를 Observable에 흐르게한다.

```java
Observable.range(1,10)
        .subscribe(new Observer<Integer>() {
            @Override
            public void onCompleted() {
                Log.d("onCompleted","owari");
            }

            @Override
            public void onError(Throwable e) {
                e.printStackTrace();
            }

            @Override
            public void onNext(Integer integer) {
                Log.d("onNext",String.valueOf(integer));
            }
        }
);

```

기본적으로 RX의 조작에서는 Observable을 만들고, Operator(어떤 처리를 하는 메소드)를 통과해서 subscribe에서 데이터를 수신한다. 이번 예에서는 Operator가 특별히 없기 때문에, 1~10의 숫자가 순서대로 subscribe로 흘러들어 간다.

### from
iterator로부터 Observable을 생성한다.

```java
String[] items = new String[]{"sasaki","SASAKI","ささき","ササキ","佐々木"};
Observable.from(items)
```

### just
Observable을 지정한다.

```java
Observable.just(1,10,4,21,19,2,13,9)
```

## Operator

### Filter
```java
Observable.just(1,10,4,21,19,2,13,9)
          .filter(new Func1() {
              @Override
              public Boolean call(Integer item) {
                  return (item < 15);
              }
          })
          .subscribe(new Observer() {
              @Override
              public void onCompleted() {
                  Log.d("onCompleted","owari");
              }
              @Override
              public void onError(Throwable e) {
                  e.printStackTrace();
              }

              @Override
              public void onNext(Integer integer) {
                  Log.d("onNext",String.valueOf(integer));
              }
         });       
```










