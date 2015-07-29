
簡単なアプローチとして、手動でDIする方法かあります。これは入力が少し多くなりますが、フレームワークの制約を受けません。とにかく、とても柔軟です。

DIを使った時、クラスの依存関係をちゃんと考える必要があります。ですがフレームワークを使えばコンテナにまかせることができます。しかし、それではシンプルなコードを保てないんです!

オブジェクトの関係は、できるだけ後で作られるべきです。例えばメインクラスとか(普通はメインクラスはモジュールが出来てから書くよね?みたいなことを行っているのかな..?)。もしDIコンテナを使う前だったら、手動でのDIをやってみてください。MacWireは使っても使わなくてもいいので。そうすればすぐにメインメソッドで再発見があるでしょう。

具体的に見てみましょう。これが手動DIの例です。

````scala
object TrainStation extends App {
   val pointSwitcher = new PointSwitcher()
   val trainCarCoupler = new TrainCarCoupler()
   val trainShunter = new TrainShunter(
      pointSwitcher, trainCarCoupler)

   val craneController = new CraneController()
   val trainLoader = new TrainLoader(
      craneController, pointSwitcher) 

   val trainDispatch = new TrainDispatch()

   val trainStation = new TrainStation(
     trainShunter, trainLoader, trainDispatch)

   trainStation.prepareAndDispatchNextTrain()
}
````

## 手動DIの利点

最初の利点は、型安全になることです。つまりコンパイル時に依存関係が解決されます。

コンパイル時であれば、起動時間を無視できるという利点があります(cクラスパスをスキャンしなくていい等)。さらに黒魔術的なコードを取り除くことができる。そしてスキャンするためのアノテーションはありません。私たちは、素のScalaとコンストラクタパラメータだけを使います。それはどこでインスタンスを作ればいいか分かりやすくなり、オブジェクトの関係が明確になります。そのアプリケーションもまたシンプルに使えますし、例えばfat-jarのようなパッケージングも簡単になります。コンテナはいらないし、フレームワークと戦う必要もないんです。

もし複雑なインスタンス生成だったり、設定に依存してたりするなら、柔軟な手動DIの恩恵が受けられます。依存関係は簡単に書きたいですよね。

## `val` vs. `lazy val` 

valには欠点があり、インスタンス化前に参照するとnullになってしまいます。valは上から下に評価されるからです。

ですが、lazy valを使えは解決します。

というわけで、手動DIのサンプルは以下のようになります。

````scala
object TrainStation extends App {
   lazy val pointSwitcher = new PointSwitcher()
   lazy val trainCarCoupler = new TrainCarCoupler()
   lazy val trainShunter = new TrainShunter(
      pointSwitcher, trainCarCoupler)

   lazy val craneController = new CraneController()
   lazy val trainLoader = new TrainLoader(
      craneController, pointSwitcher) 

   lazy val trainDispatch = new TrainDispatch() 

   lazy val trainStation = new TrainStation(
      trainShunter, trainLoader, trainDispatch) 

   trainStation.prepareAndDispatchNextTrain() 
}
````
