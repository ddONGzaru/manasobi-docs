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




