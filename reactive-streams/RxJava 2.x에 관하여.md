RxJava 2.x에 관하여
====================

###### Edited at 2015-12-21 [site](http://qiita.com/kazy/items/8b1e0bcc5cd9638876d4)

- 2.x에서는 [Reactive Streams](http://www.reactive-streams.org/)에 근거하여 재작성 됨.(Reactive Streams와 [Reactive Manifesto](http://www.reactivemanifesto.org/)는 서로 다름)
- 1.x 버전보다 높은 퍼포먼스를 목표로 설계됨.
- Java 1.8이상을 대상으로 하고, 종래에는 1.9의 [Flow API](http://gee.cs.oswego.edu/dl/jsr166/dist/docs/java/util/concurrent/Flow.html)에도 대응할 예정.
- 당분간은 1.x 버전과 병행해서 개발해 나갈 예정이고, 급하게 2.x 버전으로 갈아 탈 필요는 없음. 아직 실용 단계는 아닌 듯...

## 2.x 개발 이유
Java의 비동기스트림처리의 구현은, `RxJava`, `Akka Strams and Reactor`등 여러 개가 있다. 얼마전에, 이 비동기 스트림 처리의 표준 규격을 정하려고 하는 움직임이 있어 Reactive Strams라는 프로토콜이 만들어졌다. 

이미 [reactive-streams](https://github.com/reactive-streams/reactive-streams-jvm)라는 구현도 있고, 최근 1.0이 릴리즈 되었다. RxJava도 Reactive Streams를 준수하고 싶었지만, 그러기위해서는 Public한 인터페이스도 변경할 필요가 있었다. 그래서 RxJava 팀은 Reactive Streams 준수의 별도 버전을 2.x 버전으로 만들기로 결정 한것 같다는게 만든 경위인것같다.

더불어, 1.x 버전을 Reactive Streams로 변환하는 [RxJavaReactiveStreams](https://github.com/ReactiveX/RxJavaReactiveStreams)라는 라이브러리도 있다. 이것을 사용하면 Reactive Streams와의 상호 변환이 가능하고, Reactive Streams가 제공하는 TCK라고 불리는 TEST SUITE을 패스가능하다.

## Flow API 란?
Reactive Streams 준수의 API가 [Flow](http://gee.cs.oswego.edu/dl/jsr166/dist/docs/java/util/concurrent/Flow.html)라는 이름으로 Java 1.9에 도입 예정이다. 지금의 RxJava 2.x의 개발은 reactive-streams에 의존하고 있지만, java 1.9가 릴리즈되면 1.9+용의 reactive-streams를 Flow API로 교환가능하다. 물론 1.8에는 reactive-streams를 사용한 현재 구현도 계속 제공될 예정이다.


