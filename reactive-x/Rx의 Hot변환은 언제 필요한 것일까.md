Rx의 Hot변환은 언제 필요한 것일까?
=============

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square) ![badge](https://img.shields.io/badge/manasobi-RxAndroid-yellowgreen.svg?style=flat-square)

###### Qiita [toRisouP Edited at 2015-06-10](http://qiita.com/toRisouP/items/c955e36610134c05c860)

# Hot 변환해야하는 포인트
여러 상황이 있지만, 제일 Hot 변환이 중요한 상황은 **1개의 스트림을 여러번 Subscribe하는 경우**이다. 

#### 입력된 문자열이 특정 키워드와 일치하는지 조사
- Hot 변환이 필요한 예로 [입력된 키 입력을 감시해서 4문자의 특정 키워드가 입력되었는지 조사하는 스트림]을 작성한다.

```java
//入力文字を4文字ずつにまとめるkeyBufferStream
var keyBufferStream
    = Observable.FromEvent<KeyEventHandler, KeyEventArgs>(
        h => (sender, e) => h(e),
        h => KeyDown += h,
        h => KeyDown -= h)
        .Select(x => x.Key.ToString()) //入力キーを文字に変換
        .Buffer(4, 1) //4つずつにまとめる
        .Select(x => x.Aggregate((p, c) => p + c)); //文字から文字列に変換

//結果を表示してみる
keyBufferStream.Subscribe(Console.WriteLine);
```

```java
//実行結果の例(ABCDEFGHとキー入力した結果)
ABCD
BCDE
CDEF
DEFG
EFGH
```

#### keyBufferStream을 사용하여 "HOGE" 또는 "FUGA"의 입력을 감시
Where을 끼워넣어 HOGE판과 FUGA판 2회 Subscribe하는 것으로 한다.

```java
"HOGE"と"FUGA"をそれぞれ監視する
keyBufferStream
    .Where(x => x == "HOGE")
    .Subscribe(_ => Console.WriteLine("Input HOGE"));

keyBufferStream
    .Where(x => x == "FUGA")
    .Subscribe(_ => Console.WriteLine("Input FUGA"));
```

![alt](https://qiita-image-store.s3.amazonaws.com/0/47146/5f761bc8-720d-7dac-1c60-88ed6a533583.png)

## 무엇이 문제인것인가?
위 스트림은 무엇이 문제인것일까? 그것은 keyBufferStream이 거의 `Cold Observable`로 형성되어 있는 것이 문제이다. Cold Observable은 분기되지 않는다. Subscribe할때마다 매번 새로운 스트림을 생성하는 성질이 있다.

그렇기에 위와 같이 작성하면 아래와 같은 문제가 발행해 버리는 것이다.
- 뒤에서 다중으로 스트림이 발생해 버려 메모리, CPU를 낭비.
- Subscribe한 타이밍에 의해 흘러들어오는 결과가 상이.

```java
ストリームが2重になっている証拠
var keyBufferStream
    = Observable.FromEvent<KeyEventHandler, KeyEventArgs>(
        h => (sender, e) => h(e),
        h => KeyDown += h,
        h => KeyDown -= h)
        .Select(x => x.Key.ToString())
        .Buffer(4, 1)
        // BufferがOnNextを放出したタイミングでコンソールに出す
        .Do(_=>Console.WriteLine("Buffered")) 
        .Select(x => x.Aggregate((p, c) => p + c));

keyBufferStream
    .Where(x => x == "HOGE")
    .Subscribe(_ => Console.WriteLine("Input HOGE"));

keyBufferStream
    .Where(x => x == "FUGA")
    .Subscribe(_ => Console.WriteLine("Input FUGA"));
```
```
実行結果(AAAAとBufferが1回だけ動くようにキー入力)
Buffered
//Bufferは1回しか動いていないはずなのに
//2回出力されている = ストリームが2重で動いている
Buffered 
```

![alt](https://qiita-image-store.s3.amazonaws.com/0/47146/1ff154f3-38b2-03d7-41b1-974f1bf7addf.png)





