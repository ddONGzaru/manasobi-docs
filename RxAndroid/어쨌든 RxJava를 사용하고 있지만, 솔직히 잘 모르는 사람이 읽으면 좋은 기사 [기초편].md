어쨌든 RxJava를 사용하고 있지만, 솔직히 잘 모르는 사람이 읽으면 좋은 기사 [기초편]
=============

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square) ![badge](https://img.shields.io/badge/manasobi-RxAndroid-yellowgreen.svg?style=flat-square)

###### Qiita [k-mats Edited at 2015-12-14](http://qiita.com/k-mats/items/4d374460a3f6284dd09f)

# RxJava, RxAndroid, ReactiveX, Rx, Reactive Extensions, FRP의 차이

## RxJava
ReactiveX를 Java(JVM)에서 돌리기 위해 Netflix에서 제작.

[https://github.com/ReactiveX/RxJava](https://github.com/ReactiveX/RxJava)

## RxAndroid
~~RxJava를 Android에서 사용하기 위함으로 [Android에서 RxJava를 사용하는 사람]은 [RxAndroid]를 사용하고 있다.~~

[https://github.com/ReactiveX/RxAndroid](https://github.com/ReactiveX/RxAndroid)

[Android에서 RxAndroid는 필수가 아니다]라는 코멘트를 받았기 때문에, RxAndroid의 README를 다시 읽어 보았다.

> This module adds the minimum classes to RxJava that make writing reactive components in Android applications easy and hassle-free. More specifically, it provides a Scheduler that schedules on the main UI thread or any given Handler.<p>
이 모듈은 Android 앱에서 리액티브한 컴포넌트를 작성하기 위해 필요한 최저한의 클래스를 RxJava에 제공한다. 좀더 구체적으로 말하면, 메인 UI쓰레드, `Handler` 등의 위에서 스케쥴러를 수행하는 `Scheduler`를 제공한다.

RxAndroid는 즉 `Scheduler`를 RxJava에 추가한 것으로 생각할 수 있다.

## ReactiveX
역설적인 설명이지만, **RxJava같은 디자인 패턴**이라고 할 수 있다. 이 ReactiveX에 대한 구체적인 구현으로서 RxJava등 여러가지 Rx가 있다.

[http://reactivex.io/](http://reactivex.io/)

## Reactive Extentions (Rx)
기본적으로는 ReactiveX와 같은 문맥으로 사용되고 있다. 그러나, 정확히는 [MS사의 오픈소스 프로젝트 또는 마이크로소프트사의 ReactiveX 구현]이라고 보는것이 타당하다.

MS사의 프로젝트 페이지를 보면 같은 로고가 사용되고있고, ReactiveX로부터 파생된 Rx.Net이나 RxCpp등의 README에는 *Microsoft OSS Project*에 관한 언급이 있다.

반면, 우리가 사용하는 RxJava는 Netflix에서 제작하였다. 여기에서, Rx.Net이나 RxCpp는 Organization이 Reactive-Extensions, RxJava는 ReactiveX로 되어있다.

## Functional Reactive Programming
[Rx와 비슷하지만 조금 다른 정도]라고 생각해 두자.

[Reactive-X의 공식 페이지](http://reactivex.io/intro.htm)에는 아래와 같이 정리되어 있다.

> It is sometimes called “functional reactive programming” but this is a misnomer. ReactiveX may be functional, and it may be reactive, but “functional reactive programming” is a different animal.<p>
가끔씩 ReactiveX는 FRP라고 불려지지만 이것은 틀리다. ReactiveX는 함수형일지도 모르고, 리액티브일지도 모르지만, 그래도 FRP와는 다른 것이다.

## observe와 subscribe의 차이
RxJava를 살펴보면 [observe와 subscribe의 개념상 차이가 무엇인지 모르겠다]라고 생각되지는 않는가?

[당신이 원하던 리액티브 프로그래밍 입문](http://ninjinkun.hatenablog.com/entry/introrxja)에도 아래와 같은 구절이 있다.

> `Rx.Observable.create()`는 observer(다른 언어에서는 `subscriber`)에 데이타 이벤트(`onNext()`)나 에러(`onError()`)의 정보를 전달하는 것에 의해 커스텀 스트림을 만들어 낸다.

RxJava에 있어서 observe(r)과 subscribe(r)은 같은 의미로 사용되는 것일까? 그 의미에서는 예를 들면 `observeOn()`과 `subscribeOn()`은 같아야하지만 다른 모양이다.

[RxJava：Scheduler、비동기처리、subscribe/unsubscribe - subscribeOn()와observeOn()의 차이란？](http://qiita.com/yuya_presto/items/c8c3d77ac958c9c8f67b#subscribeon%E3%81%A8observeon%E3%81%AE%E9%81%95%E3%81%84%E3%81%A3%E3%81%A6)

또, RxJava 클래스인 `Observer`와 `subscriber`의 차이점을 조사해보면 명확한 차이가 있는 것을 알 수 있다.

> `public abstract class Subscriber<T> implements Observer<T>, Subscription`<p> So a Subscriber is the implementation of the Observer,

Rx의 원류인 [Observer 패턴](https://ja.wikipedia.org/wiki/Observer_%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3)이 있는 것을 생각하면, `Observer`가 인터페이스고 `Subscriber`가 그 구현인 것은 관계성으로써는 이해가 가는 부분이다.

한편, `Observalble`은 있는데 `Subscribable`은 없다. `Observable#subscribe()`은 있는데, `Observable#observe()`은 없다. `Subscription`은 있는데 `Observation`은 없다.

이 부분이 일관성이 없다고 느껴지지만, 이것은 RxJava의 역사를 이해하면 자연스럽게 납득이 될 것이다. 결국 양쪽의 명확한 차이점은 확인하지 못하고 마무리하지만, 문맥에 의해 [같은 것], [다른 것]으로 나뉜다고 할 수 있겠다.

## RxJava는 Promise 같은 것
[RxJava는 Promise같은 것]이라는 설명을 자주 보지만, 뭐 Promise도 모르니까!라고 생각하는 사람들도 많을 것이다. 모처럼이니까 여기에서 가볍게 다뤄보도록 하자.

[今更だけどPromise入門](http://qiita.com/koki_cheese/items/c559da338a3d307c9d88)










