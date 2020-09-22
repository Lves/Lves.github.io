

我们似乎以前已经达成了共识，“单例模式很好用，但不能滥用”。但是在Apple和第三方Swift框架中开发人员还在大量的使用它。

> "单例就是披着羊皮的全局状态" -- Miško Hevery 

今天我们看一下单例使用的确切问题，并探索如何避免滥用。

### 为什么单例如此受欢迎

单例模式为什么在iOS开发中这么流行呢？如果大多数开发人员都同意应避免滥用它们，为什么仍在被大量使用？

我认为有两个原因。首先最主要原因是Apple内部都在经常使用它，大家就会把苹果的做法当成“最佳实现”模仿、流传。

第二个原因是它确实提供了便利。单例通常为我们访问核心的值和对象提供了捷径，因为它们基本上可以从任何地方访问。看下面这个例子，我们想在`ProfileViewController`中显示当前登录的用户名，并在点击按钮时注销用户：

```swift
class ProfileViewController: UIViewController {
    private lazy var nameLabel = UILabel()

    override func viewDidLoad() {
        super.viewDidLoad()
        nameLabel.text = UserManager.shared.currentUser?.name
    }

    private func handleLogOutButtonTap() {
        UserManager.shared.logOut()
    }
}
```

像上面这样把用户信息和账户处理功能封装在`UserManager`单例中，确实非常方便（而且很常见！）。 那么使用这种模式到底有什么不好呢？ 🤔

### 单例模式有什么不好？

在讨论诸如模式和架构之类的问题时，我们很容易陷入过于理论化的陷阱。使代码在理论上是“正确的”并遵循所有最佳实践和原则，这是很好的做法。但现实往往很残酷，我们需要视情况而定。



那么单例模式通常会引起什么具体问题，为什么要避免这些问题呢？我倾向于避免使用单例的三个主要原因是：

- **它们是全局可变的共享状态**。它们的状态会自动在整个应用程序中共享，并且当状态被意外更改时，通常会发生错误。易错。
- **单例和依赖于它们的代码之间的关系通常没有很好地定义**。由于单例非常方便且易于访问，广泛使用它们通常会导致很难维护，像“意大利面条式代码”，而对象之间没有明确的分隔。隐性依赖。
- **管理其生命周期可能很棘手**。由于单例在应用程序的整个生命周期中都处于活动状态，因此对其进行管理可能非常困难，并且它们通常依靠可能为空的（optional）的值。这也使得依赖单例的代码真的很难测试，因为在每个测试用例中都不能轻易从“初始状态”开始。不便于测试。

> 译者补充：
>
> 1. 仅仅使用一次的模块，可以不使用单例，因为它会一直占用内存，**可以采用在对应的周期内维护成员实例变量进行替换**。
> 2. 单例保存用户数据，退出登陆时需要清楚，这时如果有异步存储任务就可能产生脏数据。



在之前的`ProfileViewController`示例中，我们已经可以看到这3个问题的迹象。我们不能清除的判断页面是依赖`UserManager`的，而且单例包含一个可能为空的对象currentUser，编译时并不能检测出来，即在页面展示是可能currentUser为nil。看起来就是一个等待触发的Bug😬。

### 依赖注入

与其让ProfileViewController通过单例访问依赖数据，不如将它们注入其`init`方法中。 在这里，我们将用户信息以及可用于执行注销操作的LogOutService作为必传参数：

```swift
class ProfileViewController: UIViewController {
    private let user: User
    private let logOutService: LogOutService
    private lazy var nameLabel = UILabel()

    init(user: User, logOutService: LogOutService) {
        self.user = user
        self.logOutService = logOutService
        super.init(nibName: nil, bundle: nil)
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        nameLabel.text = user.name
    }

    private func handleLogOutButtonTap() {
        logOutService.logOut()
    }
}
```

结果更加清晰，更易于管理。 现在，我们的代码安全地依赖始终存在的Model，并且具有清晰的API执行注销操作。 通常，将各种单例和管理器（managers）重构为清晰、分开的`Services`是在`App`核心对象之间创建更清晰关系的一种好方法。

### Services

作为示例，让我们仔细看看如何实现`LogOutService`。 它也将“依赖项”用于其基础服务，并提供了一个漂亮且定义明确的API，仅用于做一件事【注销】。

```swift
class LogOutService {
    private let user: User
    private let networkService: NetworkService
    private let navigationService: NavigationService

    init(user: User,
         networkService: NetworkService,
         navigationService: NavigationService) {
        self.user = user
        self.networkService = networkService
        self.navigationService = navigationService
    }

    func logOut() {
        networkService.request(.logout(user)) { [weak self] in
            self?.navigationService.showLoginScreen()
        }
    }
}
```

### 重构

从大量使用单例改成充分利用Services的代码，依赖项注入和本地状态可能非常困难且耗时。 花费时间不说，有时可能甚至需要巨大的重构。

不用担心，“使用协议来移除单例”的方式可以助你一臂之力。

不需要一次重构所有单例并创建新的`Service`类，我们可以简单地将`Service`定义为协议，如下所示：

```swift
protocol LogOutService {
    func logOut()
}

protocol NetworkService {
    func request(_ endpoint: Endpoint, completionHandler: @escaping () -> Void)
}

protocol NavigationService {
    func showLoginScreen()
    func showProfile(for user: User)
    ...
}
```



然后，我们可以让单例遵循我们的新服务协议，从而轻松地将单例作为“服务”进行改造。 在许多情况下，我们甚至不需要进行任何实现上的修改，只需将单例作为服务传递即可。

同样的技术也可以用于改造我们App中的其他核心对象，而这些核心对象可能一直以“单例模式”的方式使用，例如使用`AppDelegate`进行页面导航。

```swift
extension UserManager: LoginService, LogOutService {}

extension AppDelegate: NavigationService {
    func showLoginScreen() {
        navigationController.viewControllers = [
            LoginViewController(
                loginService: UserManager.shared,
                navigationService: self
            )
        ]
    }

    func showProfile(for user: User) {
        let viewController = ProfileViewController(
            user: user,
            logOutService: UserManager.shared
        )

        navigationController.pushViewController(viewController, animated: true)
    }
}
```



现在，我们可以通过使用“依赖项注入”和`Services`开始把所有的`view controller`中的单例去掉了。无需进行大量的重构和修改🎉！

### 总结

单例并不是无可救药，但是在许多情况下，单例确实带来了一系列问题。我们可以通过在对象之间创建更明确定义的关系并使用依赖注入来避免这些问题。



> 作者 johnsundell
> 翻译整理：乐Coding
> 参考：
> https://satanwoo.github.io/2016/04/11/dispatch-once/
> https://objccn.io/issue-13-2/
> https://www.swiftbysundell.com/articles/avoiding-singletons-in-swift/



![image](https://upload-images.jianshu.io/upload_images/1159872-d2c50de82278e1cc?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
