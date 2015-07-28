[Dependency Injection (DI)](http://en.wikipedia.org/wiki/Dependency_injection)は、クライアントとサービスの結合を緩やかにできる人気のパターンです。

このガイドで説明するのは、できるだけScalaでDIする方法です。それと[MacWire](https://github.com/adamw/macwire)のヘルプです。

Dependency Injection はシンプルなコンセプトで比較的シンプルに作ることができますが、コストを考えずにDIのコンテナやフレームワークを多用したり複雑にするのは避けるべきです。

<p class="message">
  このガイドは日本語訳したものです(<a href="https://github.com/ryoppy/di-in-scala.github.io-sources">GitHub</a>)。
  間違いや、十分でないところ、ベターじゃないとこあれば、どしどしプルリクおなしゃす。
</p>

## DIとは?

DIは、クライアントとサービスのコードを分離します(クライアントは、他のサービスかもしれません)。サービスは、何に依存しているか分かるようにする必要があります。サービスの中で依存したサービスを作る代わりに、依存してるサービスの参照が"注入"されます。そうすることでコードの理解が早まるうえ、テストビリティや再利用性が高まります。

依存性を注入するのは、色々やり方があります。しかし、私たちはコンストラクタパラメータで渡す方法を使います。他にもやり方はあり、setter/fieldやservice locatorなどがあります。

DIは、[Inversion of Control](https://ja.wikipedia.org/wiki/%E5%88%B6%E5%BE%A1%E3%81%AE%E5%8F%8D%E8%BB%A2)という需要な側面がある。実装したあるサービスは他のサービスの"外側"で作られるべきで、あるサービスの中で直接`new`で依存したサービスを作るのは許されません。

もしDIがよく分かってなければ、[motivation behind Guice](https://github.com/google/guice/wiki/Motivation)を読むことをお勧めします。これはJavaですが、アイデアは同じなので参考になるでしょう。

## 他のアプローチ

DIのフレームワークはたくさんあります。下記は、pure Scala+MacWireの代わりになるものを示しています。

Using frameworks:

* [Subcut](https://github.com/dickwall/subcut): service locator/dependency injectionをmixしたパターンです。
* [Scaldi](http://scaldi.org/): Subcutと似てます。
* [Spring](http://spring.io/): Springは人気のJavaフレームワークでScalaでも使えます。
* [Guice](https://github.com/google/guice): 人気のJavaのDIフレームワークです。

Using pure Scala:

* [Cake pattern](http://jonasboner.com/2008/10/06/real-world-scala-dependency-injection-di/)
* [Reader monad](http://blog.originate.com/blog/2013/10/21/reader-monad-for-dependency-injection/)

## Running example

## 実行例

駅のシステムを作ってると仮定します。ゴールは、prepareメソッド (load, compose cars)を呼ぶことと、次の電車を送り出すことです。そのためには、以下のサービスクラスをインスタンス化する必要があります。

````scala
class PointSwitcher()
class TrainCarCoupler()
class TrainShunter(
   pointSwitcher: PointSwitcher, 
   trainCarCoupler: TrainCarCoupler)

class CraneController()
class TrainLoader(
   craneController: CraneController, 
   pointSwitcher: PointSwitcher)

class TrainDispatch()

class TrainStation(
   trainShunter: TrainShunter, 
   trainLoader: TrainLoader, 
   trainDispatch: TrainDispatch) {

   def prepareAndDispatchNextTrain() { ... }
}
````

各クラスの依存関係はコンストラクタパラメータで表してます。
