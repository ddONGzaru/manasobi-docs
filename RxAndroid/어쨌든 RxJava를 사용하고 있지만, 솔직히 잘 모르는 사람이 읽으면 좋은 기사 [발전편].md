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



    
 
