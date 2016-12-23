Hot & Cold Observable
=============

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square) 

###### [site](https://brunch.co.kr/@tilltue/18)

ReactiveX 관련 자료들을 보다보면, "Hot 또는 Cold" Observable이 종종 언급되는데 이 둘의 차이점과 관련 함수들을 통해 이해를 해보자.

## Hot Observable
생성과 동시에 이벤트를 방출하기 시작한다. 또, 이후 subscribe 되는 시점과 상관없이 옵저버들에게 이벤트를 중간부터 전송해준다. ReactiveX에서 다른 말로, Connectable Observable이라고 부르기도 한다.

관련 메소드는 아래와 같고, [subscription의 공유](https://brunch.co.kr/@tilltue/15)에서 확인 할 수 있다.

- publish / multicast / connect 
- replay / replayAll
- share / shreaReplay
- shareReplayLatestWhileConnected

## Cold Observable
옵저버가 subscribe되는 시점부터 이벤트를 생성하여 방출하기 시작한다. 기본적으로 Hot Observable로 생성하지 않은 것들은 Cold Observable이라고 이해하면 된다.

이해를 돕기 우해 예를 들면, 유투브의 실시간 방송과 일반 VOD 방송의 개념으로 이해하면 좋겠다. 유투브의 실시간 방송은 시정자(Observer)가 어느 시점에 방송을 청취하던 상관없이 방송이 진행되고, 시청자는 방송을 시청(subscribe)하는 시점부터(middle emit event) 방송을 볼 수 있다.(이벤트를 받을 수 있다) 이 개념이 **_Hot Observable_** 이다.

일반 VOD의 경우 어떤 시청자건(observer) 시청을 시작하면(subscribe) 처음부터 방송이 시작된다.(emit 1, 2, 3, ...) 이 개념이 **_Cold Observable_** 이다.


