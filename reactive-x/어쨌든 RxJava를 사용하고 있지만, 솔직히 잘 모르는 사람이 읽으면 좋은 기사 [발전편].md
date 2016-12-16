어쨌든 RxJava를 사용하고 있지만, 솔직히 잘 모르는 사람이 읽으면 좋은 기사 [발전편]
=============

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square) ![badge](https://img.shields.io/badge/manasobi-RxAndroid-yellowgreen.svg?style=flat-square)

###### Qiita [k-mats Edited at 2015-12-16](http://qiita.com/k-mats/items/3844a08b8958f77c45d0)

# Cold / Hot

> Rx에서의 프로그래밍 요령을 이해하려면, [Cold vs Hot Observables](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/gettingstarted/creating.md#cold-vs-hot-observables)의 컨셉에 대한 이해가 빠질 수 없다.

> Cold한 Observable은 Subscribe가 호출되고 처음으로 값을 Observer에 뱉어낸다. 값은 Subscribers 간에 공유되지 않는다. 한편, Hot한 Observable에서는 (예를 들면 마우스의 move 이벤트나 주가의 티커 등)subscription이 유효하기 전에 이미 값이 발생되어 있는 것을 나타낸다. Observer가 Hot한 Observable을 subscribe하면, Observer는 그때까지 발생값을 전부 받아들인다.<br>
값은 모든 Subscribers 사이에서 공유(복수의 Subscribers에 분기, 분배)되어, 각 Subscribers에 항상 최신 값을 취득한다. 예를 들면, 어떤 특정 주식의 티커를 subscribe하고 있는 Subscribers가 한개도 없다고 해도, 티커는 주식 동향에 따라 값을 계속 계속 뱉어낸다. 따라서, Subscriber가 이 티커를 subscribe하면 수신 값은 그 다음에 오는 최신의 값이다.

- Cold: 스트림의 전후를 연결할 뿐인 파이프. 간단해서 의미가 없음. 대부분의 오퍼레이터가 해당.
- Hot: 스트림으로부터 값을 계속 발행하는 수도꼭지. 항상 값이 흐름. 뒤로는 파이프가 몇개라도 붙을 수 있음.

#### [RxのHotとColdについて](http://qiita.com/toRisouP/items/f6088963037bfda658d3)

> Hot >>> subscribe 되지 않고도 값을 emit 함.<br>
Cold >>> subscribe()가 호출될때까지 emit 하지않음. subscribe()마다 새로운 값을 emit함.

#### [Observableは友達](http://chooblarin.com/observable_is_my_friend.html)
즉 [Subscriber에 요구될때까지 값을 발행하지 않는 수동적인 Observalble(stream)은 **_Cold_**], [계속 값을 뱉어내는 능동적인 Observalble은 **_Hot_**]이라고 할 수 있다.

이러한 Hot한 Observalble은 지금까지의 Cold한 Observalble과는 별개의 취급이나 주의가 필요해 보인다.

이번은 아래의 기사를 전체적으로 읽고 Cold/Hot을 조금 상세하게 살펴 보는 것과 함께, [복수의 Subscribers 간에서 공유(분기, 분배)]의 의미에 대해 알아 두자.

Cold/Hot의 설명을 여러가지 읽어 보면, Subject가 몇번이고 언급되는 것을 알수있다. 이것은 알아둬야 할 것 같으므로, 다음으로는 Subject에 대한 이야기로 옮겨보자.

# Subject 

[Rx에서 알아두면 편리한 Subject들](http://qiita.com/hide92795/items/f7205c8171826cc2153b)

`Subject`는 Subscriber도 되고, Observable도 되며, 종류가 다양해서 잘사용하면 편리하다는 것이다.

그럼, 이것이 Cold/Hot과 어떤 관계가 있는것일까? 아래 기사를 읽어보자. (코드는 C#이지만, Rx를 사용하고 있기때문에 대강은 읽어진다)

[[Rx입문] Subject 해설 / Hot, Cold한 Observable](http://qiita.com/acple@github/items/8d3a4d3414fa59adff70)

[Subject가 취급하는 값은 Rx의 외부로 부터 오기때문에, Subject로부터 온 값은 Rx 세계에서 제어되지 않은 값이고 이것을 Rx 언어로 표현하면 'Hot'이라고 말한다. 따라서, Subject는 Hot한 Observable이다]라는 것이다. 샘플 코드에서도 Hot한 Observable을 반환하는 `Timer`메소드는 `return subject.asObservable()`형태를 띠고 있다.

그럼, Subject가 Hot한 Observable인 것은 알았는데, 역으로 Hot한 Observable은 Subject뿐인 것일까? 다른 Hot한 Observable은 없는 것일까?

상기 기사에서는 [Cold -> Hot 변환]으로서 `Observable.publish()`를 열거하고 있다. 즉, `publish()`는 Hot한 Observable을 반환하는 것이다.

이 메소드의 반환 값의 타입은 ConnectableObservable로 되어있다. Subject가 아니다. ConnectableObservable, Subject 모두 Observable의 서브 클래스이고, 형제지만 부자관계는 아니다. 즉, ConnectableObservable은 Subject가 아니기 때문에, Hot한 Observable은 Subject 이외에도 존재한다고 할 수 있다.

> - Hot Observable의 예
>   - Observable.interval(), window()등 시간을 부여해서 얻을 수 있는 Observable
>   - Subject(외부로부터 onNext()를 자유롭게 호출할 수 있기 때문에 하류의 사정에 맞추어 제어하지 못함)
>   - publish()등을 적용해서 얻을 수 있는 ConnectableObservable (Hot변환이라고 부를 수 있음)
>   - Hot Observable에 operator를 적용해서 얻을 수 있는 Observable (일부 제외)

# Backpressure

Cold Observable은 [Subscriber에 값을 요청받을 때까지 값을 발행하지 않는다]라고 설명했다. 이 구조를 자세하게 보면 Backpressure라는 중요한 개념이 등장한다.

Subscriber는 `subscribe()`에 값을 요구할 때, [자신이 몇개 정도 값을 처리 가능한지]를 Cold Observable에 가르쳐 줄 수 있다. 이것에 의해, Cold Observable의 값의 전송속도가 Subscriber의 처리속도를 넘지못하게 제어하는것이 가능해 진다. 이때에 Subscriber는 `request()`라는 메소드에서 값의 개수를 Observable에 전달한다. 이 메소드의 코멘트를 읽어보자.

> Request a certain maximum number of emitted items from the Observable this Subscriber is subscribed to.
This is a way of requesting backpressure. To disable backpressure, pass {@code Long.MAX_VALUE} to this method.<p>
Observable로부터 발행된 Subscriber가 subscribe하는 item의 수의 최대치를 request한다. 이 방법은 Backpressure라고 불린다. Backpressure를 무한히 하는 것으로 이 메소드에 Long.MAX_VALUE를 전달하면 된다.

또한, `request()`에는 디폴트로 Long.MAX_VALUE가 전달되기 때문에 디폴트로서 Backpressure는 무한이다. 또 당연하지만 Hot Observable은 Cold Observable 같이 Backpressure에 의한 제어는 불가능하다.

그럼 Hot Observable의 전송속도가 Subscriber의 처리속도를 넘은 경우에 Backpressure를 사용하면 어떻게 될까? [Subscriber는 언제나 최신의 값만 이용하고 나머지는 버린다] [버퍼를 준비해서 어느 정도 값을 모아두도록 한다] 등의 대응책을 사용하도록 하고 있다.

디폴트로서 Backpressure가 무한이라면 언제 사용하는가? Hot Observable에서의 구동은 그 밖에도 무엇이 있는가? 라는 질문을 해결하기 위해선 아래의 기사를 읽어 보도록 한다. 다행이도 올해 Advent Calendar에 투고되었다.

[RxJava2：Backpressureで流速制御](http://qiita.com/yuya_presto/items/0e95271bc85efe7f768e)

덧붙여 Backpressure를 이용하여 값의 송수신속도가 제어가능한 스트림(Observable)을 [Reactive Streams](http://www.reactive-streams.org/)라고 부르는 것 같다.

[Reactive Stream란?](http://takezoe.hatenablog.com/entry/20150217/p1)

좀더 정확하게 말하면 Reactive Streams는 [Backpressure가 포함된 비동기 스트림 처리의 표준을 정하는 사용]이다.
아래 기사에 참조되어있는 발표 자료에 매우 자세히 Reactive Streams 뿐만 아니라 Reactive Programming, FRP와의 관계까지 커버하고 있다.

[JJUG ナイトセミナー「Reactive Streams特集」に行ってきた #JJUG](http://mike-neck.hatenadiary.com/entry/2015/06/25/004011)

[RxJava의 에러 핸들링](http://qiita.com/boohbah/items/108b378c5cb593c666e6)






    
 
