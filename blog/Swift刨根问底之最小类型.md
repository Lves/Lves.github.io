
![icon](https://upload-images.jianshu.io/upload_images/1159872-5b89028142460e89?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

有时候，代码太平常了，以至于我们忘记了实际上它是多么的奇怪。 我的意思是，Swift是一种强类型语言，对吗？ 类型 类型 类型 重要的事情说三遍！ 我们写完代码，然后编译器为我们执行它。 您会看到一些下边这样的代码：

```
func addOne(_ x: Int) -> Int {
    fatalError("Haha! No Int for you!")
}
```

这是一段合法的Swift代码。 我认为很多人都不会感到惊讶。 当然是合法的，对吗？ 但这却令人惊讶。 addOne是一个接受Int并返回Int类型的函数。 它确实接受一个Int，但是没有返回Int 。

你可能这么认为：“它崩溃了。 如果它崩溃，它就不能返回Int。”

好吧，是的。 但是它承诺会返回一个Int类型的值。 它没有承诺“返回Int或崩溃”，是吗？ 强类型的本质应该是：编译器将履行我们的承诺，编译器不会对此代码视而不见。

“编译器不可能知道可能会崩溃的所有内容”有人这么认为。

我不愿意承认这一点，但是即使这样，编译器仍可以清楚地看到此函数没有返回Int。 任何地方都没有“返回 intValue”。这里有些奇怪。这仅仅是编译器魔术，还是还有更深的东西？

提示：确实有更深的东西。

### 从Void类型开始说起

让我们退后一步，先思考一下让我们感到惊讶的另一件事。

```
let printSquid: () -> Void = { print("🦑") }
```

`printSquid`是一个不带参数的闭包，它返回什么呢？

您可能会想说“什么都不返回”。但这不是代码所要表达的。 它说它返回Void。 Void是什么？

```
public typealias Void = ()
```

Void是一个真实的东西。 它是空元组的类型， 并不是什么都没有。在Swift中，类型`()` 和值`()` 碰巧具有相同的拼写。 当我们将`()` 表示为返回类型时，我们通常使用它的重命名Void来代替。 但是`()->()`与`()-> Void`完全一样。 为什么要用`Void`？因为Swift的根源是ObjC，而ObjC是类C语言，在C中有一些`void`函数，它们实际上不返回任何内容。

在Swift中，每个函数都会返回一些东西。 有些函数返回`()`，语法糖在某些情况下会把`-> Void`给替换成了`return()`。 如果愿意，您可以自由替换它们（这不是惯用的Swift用法，但是合法的）。

```
func f() -> Void {}
func g() { return () }
```

就像`print`这样的函数似乎没有返回值，实际上确实会返回Void：

```
let printResult: Void = print("🦑") // 🦑
print(printResult)                  // ()
```

因为您通常不习惯使用Void值，但它们是完全合法的。如果我们不在此处添加`：Void`，则编译器将发出警告，


![1](https://upload-images.jianshu.io/upload_images/1159872-33ece8877547d892?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 类型的度量

每种类型都有一组属于该类型的值。 某些类型具有大量的值，Int的值约为18亿五千万，字符串甚至更多。 某些类型的值很少，大多数枚举只有少数可能的值，布尔类型只有2个。强类型的强大功能是我们可以创建小类型， 类型仅包含那些对我们程序合法的值。 这就是为什么我们更喜欢用String和Int枚举，除非我们意思是“任意文本”或“任意数字”。而我们更喜欢使用结构体，而不是字典，除非我们想表示“键与值的任意映射”。 更多受约束的类型能帮助编译器更好地完成指令。

那么一个类型能小到什么程度呢？

Void只有一个值。 所以它足够小。 在大多数语言中都有这种类型，它称为Unit（单元） — 拥有一个值的类型。

一个类型可以更小吗？

### 没有值的类型

是时候返回我们最初的例子了

```
func addOne(_ x: Int) -> Int {
    fatalError("Haha! No Int for you!")
}
```

看一下`fatalError`感觉很像`print`函数。 语法相似。 但是我们不能把有`fatalError`的地方换成`print`：

```
func addOne(_ x: Int) -> Int {
    print("Haha! No Int for you!") // Cannot convert return expression of type '()' to return type 'Int'
    ^^^^^
}
```

编译器如何知道我们可以在此处使用`fatalError`，而不能使用`print`？ 也许编译器只知道`fatalError`是特殊的。 下面是在Swift 3之前`fatalError`的声明：

```
@noreturn public func fatalError(...)
^^^^^^^^^
```

`@noreturn`关键词告诉编译器该函数不会返回，然后编译器便有了让我们忽略应该返回Int的承诺。 但是... 我讨厌这种解决方案。 既然可以这么搞，如果我们给需要返回Int或者抛出异常的函数添加`@noreturn`将会发生什么？

```
@noreturn func addOne(_ x: Int) -> Int { ... }
@noreturn func runForever() throws { ... }
```

上面第一个可能是错误，编译器应该禁止它。 但是第二个呢？ 实际上是会抛出`return?`。 但是这个属性应该允许吗？ 没有一个真正明确的答案。 函数是否会需要使用`@noreturn`？ 这对函数重载有何影响？

在Swift 3中，系统丢弃了@noreturn，并使用以下类型解决了该问题：`Never`。 现在`fatalError`的签名如下：

```
public func fatalError(...) -> Never
                               ^^^^^
```

`Never`又是什么？ 这是一个新的编译器技巧吗？ 不。 它只是一种类型，没有值的枚举类型：

```
public enum Never {}
```

Never有多少个值？ 是的，没有。 您无法想象， 它具有各种有趣的含义。

```
func f() -> Never { ... }   // f never returns
func g(_: Never) { ... }    // g can never be called

struct S {                  // S can never be constructed
    let nope: Never
    ...
} 

enum E {
    case ok(Int)       // E.ok can be constructed
    case nope(Never)   // E.nope can never be constructed
}

// But also interesting:

struct G<Element> {}
let ok = G<Never>()     // This is fine. Never can be a phantom type.
```

另一个有趣的含义是，Never是一个空数组。

Never可能是最小的类型。 我们称其为“uninhabited type”（无人居住的类型？该怎么翻译？）。Never没有什么特别的地方。您可以创建自己的无值枚举，它们的工作原理相同。

```
// Our own custom uninhabited type
enum NeverReturn {}

func neverReturn() -> NeverReturn {
    // We could call fatalError() here, since generating any uninhabited type
    // is sufficient to create any other. But we can also use an infinite loop.
    // The compiler can prove this will never return. Never isn't just for crashing!
    while true {}
}

func addOne(_ x: Int) -> Int {
    neverReturn()   // It's fine not to return Int, because the compiler knows this doesn't return
}

// While Never can be used to create a NeverReturn, they're not the same type
let never: Never = neverReturn()           // Cannot convert value of type 'NeverReturn' to specified type 'Never'
let neverEver: NeverReturn = fatalError()  // Cannot convert value of type 'Never' to specified type 'NeverReturn'
```

尽管我们可以创建属于我们自己的`无人居住类型` (`uninhabited type`)，但我并不推荐这样做。Swift团队可以选择用“exit”和“abort”类型，但他们故意选择不这样做。 一种`uninhabited type`就足够了。

### You never hit bottom

在类型理论中，无人类型通常称为底部类型，并写为⊥。 底部类型是所有其他类型的子类型。 因此，底部类型永远不会是一个Int、一个String以及一个UIViewController或者其他所有类型。 与底部类型相反的是顶部类型（⊤），它是其他所有类型的超类型。 在Swift中，就是Any。


但是在Swift中，Never实际上并不是底部类型。 如果是这样，您可以编写此代码，但不能：

```
let x: Int = fatalError()   // Cannot convert value of type 'Never' to specified type 'Int'
```

我们使用一些额外的语法可以轻松解决这个问题：

```
let x: Int = { fatalError() }() 
```

因此，当作为函数返回值时，其行为类似于底部类型，但在作为函数形参或将其分配给变量时，就没有这种行为了。 对于`Never`成为真正的底部类型，并使其符合所有非静态协议（即没有静态或初始化要求的协议）会更加一致。 我不确定这是否会产生什么奇怪的极端情况。 但是也许会。



### 有些事情只是没有发生

`Never`是在`stdlib`中我最喜欢的类型。 自从Swift 3引入它以来，它一直是我最喜欢的类型，并且Combine框架(iOS13 系统新添加的框架，配合SwiftUI更加完美)将其应用于泛型，从而完全证明了我对它的热爱。

来看一下Combine框架内部的Publishers协议，它会生成一系列值或报一个类型错误：

```
public protocol Publisher {
    associatedtype Output
    associatedtype Failure : Error
    func receive<S>(subscriber: S) where S : Subscriber, Self.Failure == S.Failure, Self.Output == S.Input
}
```

如果Publisher从未产生错误，Failure应该是什么？ 当然是Never。不需要魔法， 它就能正常工作。 如果仅会产生Failure呢？ 好吧，Output 将会是 Never（请参见IgnoreOutput）。 如果类型正确，特殊情况将消失，那么它可以正常使用。

因此，`Never`类型已经从很少人了解的类型发展为Swift开发者可能每天使用，却无需考虑的东西。 这让我非常高兴。

>作者：robnapier
>
>翻译：乐Coding


![logo](https://upload-images.jianshu.io/upload_images/1159872-35d03b7ee45469af?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

