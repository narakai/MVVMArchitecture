# 简介
Android 的 MVVM 框架，在 GitHub 上也有不少，都蛮优秀的，在我看来其实**框架的好处就是经过尽可能的封装，让开发变得更便捷，同时也更稳定，更安全，代码也更统一，更干净等等**。你只需经过简单的几个步骤，就能很快的上手，MVVM 的分层方式可以看看这篇：[MVVMArchitecture，一款可配置的 MVVM 框架](https://www.jianshu.com/p/87fabd2523b4)，或自行搜索相关的知识点，这里就不展开叙述了。

本框架也是一样的，**目的也是为了方便开发者而对常用的功能进行封装**。目前它的灵感**部分**来源于 [MVVMHabit](https://github.com/goldze/MVVMHabit)，以及 [MVVMLin](https://github.com/AleynP/MVVMLin) 这两个框架，从中吸收了不少优点，如果后续发现其他框架有很好的功能封装点，我也会将其吸收过来，不断完善功能。

框架主体代码使用 Kotlin 编写，小部分工具代码使用 Java 编写，兼容 Kotlin 和 Java，兼容 Rx 和 协程。

下面我们通过一些举例来认识框架的好处，详细使用方法可参考 [Wiki](https://github.com/imyyq-star/MVVMArchitecture/wiki) 或 [MVVMArchitectureSample](https://github.com/imyyq-star/MVVMArchitectureSample)。

# 举例
下面的举例，都是我在从事 Android 开发过程中遇到的实际问题，并不是凭空瞎想的，我们一个一个的来看。

## 1. 应用的大小问题
APK 的大小向来是一个敏感的点，越小则被用户接受的可能性就越大，但通常在开发中，我们不可避免的要引入额外的第三方库，引入额外的库就意味着 APK 的大小也会变大，这也是正常的。

不过有一点我没法忍的就是：如果我的项目很小，只需要用到框架中的一小部分基础功能，并不需要其他的功能，比如我不需要请求网络数据，可是开发框架却因为封装了 Retrofit2 而导致部分或全部相关代码也被打进去 APK 包里了，但是这部分代码我是根本用不到的。或者是，我的项目很大，但我也可能没有用到框架的某一部分，这个问题就很难忍了。如果你没有引入源码，想改都没办法。

所以我期望的是：**框架可以给很小的项目使用，也可以给很大的项目使用，可以在不修改源码的情况下，通过配置文件去禁用某些功能，从而保证 APK 包的干净。**

**本框架的第一个要点就是尽可能的可定制化，不需要的功能，不仅不会打包到 APK 中，甚至连对象实例都不会创建，尽可能为封装的功能提供：全局开启/关闭，和局部开启/关闭**。


## 2. 框架的更新和更改问题
在 Android 中，想使用已经提交到远程仓库的第三方库，根据文档 implementation 即可，这样做的好处当然是很多的，可对于**基础框架类型**的开发库来说，我认为有一定的限制。

首先基础框架是属于应用中的底层架构，很多功能都是基于框架而来的，牵一发而动全身，如果有一天你发现了在某种条件下出现了某些 Bug，而框架却没有处理到（任何框架都有可能有问题），此时你要么把源码引入进来自行修改，要么找作者修改后再更新。这种方式都需要一个过程，不太灵活。

其次是没有源码，你也没法为框架做开源贡献，遇到问题可能自己解决了就懒得去贡献给框架了。

最后是也许你的项目是需要交付源码出去的，客户可能会要求你除去项目中无用的代码或减少 APK 的大小。

**本框架第二个独特的点，在于项目并不通过远程仓库发布，而是通过 git submodule 的方式，让框架以源码引入的方式，成为你项目中的子仓库**。

我认为最好的使用方式是：
1. Fork 框架到你的 GitHub 成为你的仓库。
2. 使用 Fork 后的将作为你的项目子仓库。（后面再说如何引入框架）

如此一来，框架就成为你自己的了，你即可以对框架进行修改后，**发起合并请求合并到我的框架中，也可以从我的框架中获取实时更新**。发现 Bug 后马上能得到修复，不需要等到版本发布。


## 3. 生命周期的问题
在 Jetpack 出来之前，生命周期的处理还是比较麻烦的，有了 Jetpack 后，我们可以通过 LiveData 和 DataBinding 来关联数据和界面，这样生命周期就安全了。

框架的基础就是封装了 Activity/Fragment/XML 和 ViewModel，这是最基础的开发范式。

将 Jetpack 提供的 ViewModel 类作为 MVVM 中的 VM 层，它负责执行耗时操作和更新。Activity/Fragment/XML 作为 V 层，负责展示页面和接收用户操作。

他们之间的关系和数据流动如下：

![图1](https://imyyq.coding.net/p/MyMarkdownImg/d/MyMarkdownImg/git/raw/master/c41a5783e1e1256080e0b92a4bb86234d9f70c42e9e8b242e7b6ca9f86bc780e.png)  

这样就将 V 和 VM 安全的关联起来了，Activity/Fragment/XML 都持有 VM 的实例，VM 的数据流向 V 是通过 LiveData 和 DataBinding。

可以在这个基础上做很多封装，常见的一个例子：打开一个界面后执行网络请求，在请求还没有完成前，用户就退出了界面，此时通过封装可以让 VM 感知 V 的生命周期，在 V destroy 的时候 cancel 掉网络请求，避免资源的浪费。


## 4. 图片加载的问题
图片加载我个人推荐使用 Glide，如果你喜欢用其他的，可以参照 Glide 的封装方式自行封装。在没有经过封装前，可能你需要写这样的代码：

```java
Glide.with(imageView.getContext())
    .load(url)
    .apply(new RequestOptions().placeholder(placeholderRes))
    .into(imageView)
```

而利用 DataBinding，我们可以做到这样：

```xml
<ImageView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    url="@{viewModel.mImageUrl}"
    placeholderRes="@{R.mipmap.ic_launcher}"
/>
```

mImageUrl 是一个 LiveData，或 ObservableField，你只需要控制 mImageUrl 的图片链接，其他的框架帮你实现了。


## 5. 请求服务器数据的问题，使用 Rx 还是 协程 的问题
如果你是用 Kotlin 开发项目，异步操作当然首选协程啦，协程是可以替代 Rx 的。但是如果你使用 Java 开发，那么就只能使用 Rx 了，在请求服务器数据这一块，甚至你都不用使用什么 Rx 或协程，直接用 Retrofit 自带的回调也是可以的，缺点就是对数据的处理没那么灵活了。

使用什么方式去获取服务器数据，其实不是关键，关键是在于请求回来的数据如何处理？数据解析 Retrofit 已经帮我们解析成对象，我们要关注的是如何操作解析后的对象。

一般来说服务器接口返回的数据都有通用字段的（如果没有的话，恐怕你遇到的是远古后端团队），比如如下示例：

```json
{
  "errorCode": 0,
  "errorMsg": "",
  "data": []
}
```

**以上字段名称其实极大可能不同团队是不一样的，字段类型也可能是不一样的，甚至一个应用中不同接口的格式都不统一，这种恶心的情况我也是遇到过的**，但是不管怎么说，通常都会有以上 3 种类型的字段：状态码，状态信息，数据。我们需要对这几种类型进行封装，提供一个通用接口，如下：

```kotlin
/**
 * 实体基类必须实现这个接口并复写其中的方法，这样才可以使用 BaseViewModel 中的协程方法
 */
interface IBaseResponse<T> {
    fun code(): Int?
    fun msg(): String?
    fun data(): T?
    fun isSuccess(): Boolean
}
```

然后你就可以基于这个接口来定义你的数据基类啦，如下：

```kotlin
data class BaseEntity<T>(
    var data: T?,
    var errorCode: Int?,
    var errorMsg: String?
) : IBaseResponse<T> {
    override fun code() = errorCode

    override fun msg() = errorMsg

    override fun data() = data

    override fun isSuccess() = errorCode == 0
}
```

看到没？变化的只是 BaseEntity 的字段，而接口方法将他们统一了，不管服务端接口返回的数据多么妖孽都能 Hold 住。这样我们就可以对数据进行统一的处理啦，具体的去看 Wiki 和示例吧。


## 6. 加载中对话框的问题
通常来说，在进行耗时操作时，都会向用户展示相关的信息，比如：

![图2](https://imyyq.coding.net/p/MyMarkdownImg/d/MyMarkdownImg/git/raw/master/20e193e5a25e3d94c8edf912dfaa2e25d2d94928aa14bcdfeee5d3a0462d7030.png)  

以上这种是对话框形式的，还可能是内嵌形式的，比如：

![图3](https://imyyq.coding.net/p/MyMarkdownImg/d/MyMarkdownImg/git/raw/master/7bd4999c86086bd7fb18dfe0abee7fda587a9c727c66507438bd32d109a39db3.png)  

框架中也进行了封装，可以自定义相关 Ui，**同时也提供了全局开启/关闭，和局部开启/关闭的功能**，为什么呢？因为 Loading 的显示和隐藏，是和界面相关的，通过 LiveData 来操作，可能有些应用根本不需要这个功能，或者说应用中只有某些页面需要，这样就可以最大程度的避免实例创建或引入某些不需要的库。

**注意：全局开启/关闭，和局部开启/关闭的功能是框架的普遍功能，只要你觉得能想到的，都可以尝试去做，比如 Activity 的侧滑返回功能，VM 启动和结束界面的功能等，详见 GlobalConfig 配置类**


## 7. 应用前后台监听的问题
有了 Jetpack，我们可以通过 ProcessLifecycleOwner 来监听应用的前后台，无需再去对 activity 计数了，框架提供了 AppStateTracker 类来监听。

除了 AppStateTracker 外，还有许多工具类可用，比如数据库 RoomUtil 类，AppActivityManager 管理类等。



# 引入框架
经过上述的描述，你应该能认识到使用框架的好处了，如果看到这你还有兴趣，那么去看本框架的 [GitHub Wiki](https://github.com/imyyq-star/MVVMArchitecture/wiki) 吧，根据 Wiki 去引入框架和使用框架。

不要担心，其实就是一些代码封装而已，并没有什么复杂的东西。

**另外，如果你发现问题，欢迎提交合并或提 Issue，或者是有相关的建议或不明确的功能点也可以提**。
