RxAndroid 정리
=============

![badge](https://img.shields.io/badge/manasobi-RxAndroid-yellowgreen.svg?style=flat-square)

# 1. RxAndroid의 시작

## 1.1 Rx~Rx? 하는데 그게 뭐에요?
간단히 설명하면 ReactiveX (이하 Rx)는 MS진영에서 먼저 나왔고 넷플릭스가 이를 Java로 컨버팅하였고 (이게 무려 2년 전..)
지금은 Android를 비롯한 여러 언어에서 사용이 가능함.
Rx의 역사에 대해서 설명한들 뭐가 도움이 되겠는가~ 잡 지식이 늘어나겠지.
다른 설명이 필요하신 분들은 [RxAndroid](https://realm.io/kr/news/rxandroid/)에서 참고 바래.
이 자료는 여기저기 요기거기에서 발표자료로 많이 쓰여 익히 보신 분들도 있을 꺼야.
(국내 자료는 김용욱님께 거의 유일무이하고...찾아봐도 다 비슷한 내용이다.)

공식 사이트는 [http://reactivex.io/](http://reactivex.io/)

## 1.2 근데 어디에 쓸 수 있는거에요?
본인의 Rx를 시작하게 된 경험으로 비추어 설명을 해보면
 
    1. 비동기 통신을 순차적으로 보내고 싶다. (A작업이 끝나고 B가 시작됐으면 좋겠다.)
    2. 콜백을 받았는데 받는 화면이 사라져서 null에러를 뿜으면서 죽는다.
    3. 핸들러랑 콜백 지옥에 빠져서 디버깅도 힘들고 햇깔린다.
    4. 두개의 비동기 처리가 완료된 후에 결과 값을 합쳐서 하나로 만들고 싶다.
    5. 버튼을 연타로 눌러서 이벤트가 중복실행 된다.
    6. 쓰레드 관리가 힘듭니다.

뭐 이런 이유로 기술을 찾아보던 중 Rx를 발견하였고 시작했음. 처음엔 삽질을 무진장해서 힘들었어 ㅠㅠ
이것 말고도 더 쓰는 방법이 있을 것 같은데 정말 사용 방법은 무궁무진할 것 같은데...아직도 잘 모르겠음.

## 1.3 한번 배워볼께요!! 뭐 부터 알아야 하나요?
기본적으로 사용된 디자인 패턴에 의해서 있는거니까 꼭 알아야하는 개념을 설명해 보면 첫 번째는 __Observable__. 
애네들은 아이템을 발행하는 일을 해.

영어 문서를 보면 **item** 과 **emit** 이라는 단어가 많이 나오는데 **item**은 내가 지금 다루고 있는 **Data**라고 할수 있음.
이게 배열 일 수도 있고 스트링일 수도 있고 통틀어 아이템이라고 함.
emit은 item을 내보내는 거야

두 번째는 **Subscriber**. 애네들은 발행한 아이템을 소비하는 애들임.<br>
영어문서에는 consume이라고 자주 나옴.<br>
Subscriber는 **onNext()**, **onComplete()**, onError()를 각각 가지고 있는데 
onNext()는 여러번 호출 될 수 있고 이후에 onComplete(), onError()이 처리 돼.
(맞겠지..? 테스트코드 작성할 때 TEST해보자.)<br>
**Observable** 안에는 **Subscriber**를 가지고 있어서 onNext를 호출하면 다음에 있는 **Observable** 이나 **Subscriber**가 받을 수 있음

세번째는 Observable 으로 시작해서 Subscriber으로 끝나는 완성된 하나의 로직을 **Subscription**이라고 해.
수식으로 설명하면 아래와 같음.

> Subscription = Observable + Observable + Observable + Subscriber

그리고 **Operator**가 있어 [operators](http://reactivex.io/documentation/operators.html) 이건 미리 만들어 둔 
Observable 이라고 생각하면 됨. 우리는 이걸 가지고 데이터를 조합하고 변형 시키고 필터링하고 할 수 있지.


마지막으로 **Observable**과 **Subscriber**를 합쳐 놓은 것 같은 **Subject**라는게 있어.

