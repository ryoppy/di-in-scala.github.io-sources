ですが、手動DIはもちろん銀の弾丸ではありません。手動で、それぞれ正しい依存関係を書くのはちょっとやってせれません...。

そこでMacWireです。wireメソッドがマクロでインスタンスを作るコードを生成してくれます。

そしてこれがMacWireを使ったコードです。

````scala
object TrainStation extends App {
   lazy val pointSwitcher = wire[PointSwitcher]
   lazy val trainCarCoupler = wire[TrainCarCoupler]
   lazy val trainShunter = wire[TrainShunter]

   lazy val craneController = wire[CraneController]
   lazy val trainLoader = wire[TrainLoader] 
   lazy val trainDispatch = wire[TrainDispatch]

   lazy val trainStation = wire[TrainStation]

   trainStation.prepareAndDispatchNextTrain() 
} 
````

If a new dependency is added to a service or if the order of parameters changes, the object-graph wiring code doesn’t have to be altered; the macro will take care of that. Only when introducing a new service, it must be added to the list.

もしサービスに依存関係を追加しても、マクロが解決してくれるので修正する必要はありません。新しいサービスを追加するときだけは、もちろん追加しないといけません。

The new instance creation code is generated by `wire` at *compile time*, so if you compare the byte code generated by both examples, it will be identical. The generated code is type-checked in the usual way, so we keep the type-safety of the manual approach.

wireによってコンパイル時にインスタンス化するコードが生成されます。なので、生成されたバイトコードは手動で書いたものと同じになります。生成されたコードは通常通り、型チェックされるので型安全を保つことができます。

Usage of the `wire` macro can be mixed with creating new instances by hand; this may be needed if, as discussed earlier, creating a new instance isn’t that straightforward.

wireマクロは、手動でインスタンス化するコードと混ぜても使えます。

To access `wire`, you can either import the `com.softwaremill.macwire._` package object, or extend the `Macwire` trait. For details on how to integrate MacWire into your project, see the [GitHub page](https://github.com/adamw/macwire).

wireは、`com.softwaremill.macwire._`をインポートするか、`Macwire`を継承すると使えます。詳細は [こちら](https://github.com/adamw/macwire)。

## How `wire` works

## wireの仕組み

Given a class, the `wire` macro first inspects the primary constructor, to determine the dependencies needed. For each dependency it then looks for a value which is a subtype of the parameter’s type, in the enclosing method/class/object:

wireにクラスを与えると、コンストラクタを見て何か必要か決めます。それぞれの依存はmethod/class/object

* first it tries to find a value declared in the enclosing methods and anonymous functions
* then it tries to find a unique value declared in the enclosing type
* finally it tries to find a unique value in parent types (traits/classes)
* if the parameter is marked as implicit, additionally the usual implicit lookup mechanism is used

* まず関数から探します。
* 次に、型を探します。
* そして、親の型を探します。
* もしパラメータばかimplicitだったら、通常のimlicit解決されます。

Here value can be either a `val`, `lazy val` or a no-parameter `def`, as long as the return type matches.

ここの値は、valかlazy valか、パラメータなしのdefで、マッチした型が返ります。

A compile-time error occurs if:

コンパイル時のエラーは、

* there are multiple values of a given type in the enclosing methods/anonymous functions, enclosing type, or in parent types
* parameter is marked as implicit and both implicit lookup and searching in enclosing scopes finds a value
* there is no value of a given type

* 
 
*Note*: at the moment, the `wire` macro does not take into account definitions from within a method's body (only from the enclosing class/object/method/function parameters, as described above). For details see these issues: [#11](https://github.com/adamw/macwire/issues/11) and [#15](https://github.com/adamw/macwire/issues/15).

## Using implicit parameters

A similar effect to the one described above can be achieved by using implicit parameters and implicit values. If all constructor parameters are marked as `implicit`, and all instances are marked as `implicit` when the object graph is wired, the Scala compiler will create the proper constructor calls.

The class definitions then become:

````scala
class PointSwitcher()
class TrainCarCoupler()
class TrainShunter(
   implicit
   pointSwitcher: PointSwitcher, 
   trainCarCoupler: TrainCarCoupler)

class CraneController()
class TrainLoader(
   implicit
   craneController: CraneController, 
   pointSwitcher: PointSwitcher)

class TrainDispatch()

class TrainStation(
   implicit
   trainShunter: TrainShunter, 
   trainLoader: TrainLoader, 
   trainDispatch: TrainDispatch) {

   def prepareAndDispatchNextTrain() { ... }
}
````

And the wiring:

````scala
object TrainStation extends App {
   implicit lazy val pointSwitcher = new PointSwitcher
   implicit lazy val trainCarCoupler = new TrainCarCoupler
   implicit lazy val trainShunter = new TrainShunter

   implicit lazy val craneController = new CraneController
   implicit lazy val trainLoader = new TrainLoader

   implicit lazy val trainDispatch = new TrainDispatch

   implicit lazy val trainStation = new TrainStation

   trainStation.prepareAndDispatchNextTrain()
}
````

However, using implicits like that has two drawbacks. First of all, it is intrusive, as you have to mark the constructor parameter list of each class to be wired as `implicit`. That may not be desireable, and can cause the person reading the code to wonder why the parameters are implicit. 

Secondly, implicits are used in many other places in Scala for other, rather different purposes. Adding a large number of implicits as described here may lead to confusion. Still, such a style may be a perfect fit in some use-cases, of course!

## Wiring using implicit values

It is also possible to take into account implict values when wiring using `wireImplicit`, even when looking for instances of non-implicit parameters. In this variant, for each constructor parameter, an implicit lookup will be done in addition to looking in the current scope. In case multiple values are found, a compile-time error will be reported.
