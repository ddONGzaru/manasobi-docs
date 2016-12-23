Subject 알아보기
=============

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square) 

###### [site](https://brunch.co.kr/@tilltue/4)

## Subject는 언제 써야하나?

#### Imperative eventing
어떤 값의 이벤트를 발생하고 싶다고 가정하자. 얼마나 많은 객체에게 그 이벤트를 전달해야 할지 모를 수 있다. 이럴때 subject를 만들어서 원하는 이벤트를 subscription(observer) 존재 여부와 관계없이 이벤트를 발행할 수 있다.

ReactiveX Observable에는 `Hot, Cold Observable` 개념이 있는데, Subject는 Cold Observable을 Hot하게 변형하는 효과를 얻을 수 있다.

## 1. PublishSubject
subject는 subscribe된 시점 이후부터 해당 observer에게 이벤트들을 전달한다. 이전의 이벤트를 전달하거나, subscribe된 시점에 이벤트를 전달하지 않는다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/1YN0/image/MBBplT-FTxASbHX2GGZCXI-B58g.png)

PublishSubject는 subscribe가 시작되면 이벤트를 생성하기 시작하며, 그림과 같이 observer들은 구독 시점부터 발생하는 이벤트를 받을 수 있다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/1YN0/image/90sgdbgc2bqQfvd7Cau9RMUwf4M.png)

subject가 error에 의해 종료되면, 이벤트 생성 시점 이후 발생한 subscribe는 이벤트를 받지 않고 에러를 받게 된다.

## 2. BehaviorSubject
PublishSubject와 동일하지만 초기 값을 가진 subject이다. 초기 값을 가지고 subscribe가 발생하면 초기 값에 대한 이벤트를 전달하고, 이후 값을 전달한다. 또한 내부적으로 마지막 이벤트를 저장하고 있다. 어떤 변수에 값을 저장하고 관찰하고 싶을 때 사용하면 된다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/1YN0/image/_SAIwDsbYImHFn2XICWm86wZXY4.png)

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/1YN0/image/lz1qk8o0paLG6kgv6ETURWjamKw.png)

## 3. ReplaySubject
n(Buffer Size)개의 이벤트를 저장하고 subscribe가 되면 저장된 이벤트들을 모두 전달하는 subject이다. RxSwift에서는 create(bufferSize bufferSize:Int)와 createUnbounded의 생성함수를 가진다. createUnbounded는 Subject의 생성 이후 발생하는 모든 이벤트를 저장한다.

![alt](https://t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/1YN0/image/NzAsB-qvW7KwfQBmYzxvcEC29-g.png)

