#【Android】Jetpack之LifeCycle

## 前言

LifeCycle是Google于2017年I/O大会上推出的一个生命周期感知组件，一般用来响应Activity、Fragment等组件的生命周期变化，并将变化通知到已注册的观察者。有助于更好地组织代码，让代码逻辑符合生命周期规范，减少内存泄漏，增强稳定性。

## 引入

```
dependencies {
    ......
    implementation "android.arch.lifecycle:runtime:1.1.1"
    implementation "android.arch.lifecycle:extensions:1.1.1"
    // 如果你使用java8开发，可以添加这个依赖，里面只有一个类
    implementation "android.arch.lifecycle:common-java8:1.1.1"
}
```

## 例子

比如说我们有一个类，需要在`onCreate`中初始化，在`onDestory`中释放资源。他可能长这样

```kotlin
class Test{
    
    fun init(){
        //
    }
    
    fun destroy(){
        //
    }
    
    fun connect(){
        //
    }
    
    fun disconnect(){
        //
    }
    
}
```

那么，其在Activity中使用大概是这样子的。

```kotlin
class TestActivity():AppCompatActivity{
	private lateinit var mTest:Test
    
    override fun onCreate(savedInstanceState:Bundle?){
        super.onCreate(saveInstaceState)
        mTest = Test()
        mTest.create()
    }
    
   	override fun onStart(){
        super.onStart()
        mTest.connect()
   	}
   	
   	override fun onStop(){
        super.onStop()
        mTest.disconnect()
   	}
   	
   	override fun onDestory(){
        super.onDestory()
        mTest.destory()
   	}
    
}
```

想这样的写法数量少还好，但是一旦多了起来，就会让里面变得混乱且臃肿。而LifeCycle就是来解决此问题的。

## 用法

LifeCycle把相关的生命周期通过把相关组件（Activity / Fragment）的生命周期整合起来，并允许其他对象观察次状态，把相关方法从中分理出来。

首先得先确认Activity继承自SupportLibrary26.1.0以后的AppCompatActivity，否则可以自行实现LifecycleOwner接口。

比如像下面这样

```kotlin

class InfoActivity : Activity(),LifecycleOwner {

    private val lifecycleRegistry by lazy { LifecycleRegistry(this) }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_info)
        lifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE)
        lifecycle.addObserver(TestLifeCycle()) 
    }

    override fun onDestroy() {
        super.onDestroy()
        lifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE)
    }

	// 还有其他的生命周期方法也重写下

    override fun getLifecycle(): Lifecycle = lifecycleRegistry

}
```

其中`lifecycle.addObserver(TestLifeCycle())`中，`TestLifeCycle`的代码如下。通过`addObserver()`方法，即可与生命周期进行绑定。

```kotlin
private class TestLifeCycle:LifecycleObserver{

        @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
        fun create(){
            System.out.println("onCreate")
        }

        @OnLifecycleEvent(Lifecycle.Event.ON_START)
        fun start(){
            System.out.println("onStart")
        }

        @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
        fun resume(){
            System.out.println("onResume")
        }

        @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
        fun pause(){
            System.out.println("onPause")
        }

        @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
        fun stop(){
            System.out.println("onStop")
        }

        @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
        fun destroy(){
            System.out.println("onDestroy")
        }

        @OnLifecycleEvent(Lifecycle.Event.ON_ANY)
        fun any(){
            System.out.println("onAny")
        }

    }
```

如果你使用Java8进行开发，则推荐使用以下写法，以下为Java代码

```java
public class Java8Observer implements DefaultLifecycleObserver {
    private static final String TAG = Java8Observer.class.getSimpleName();

    @Override
    public void onCreate(@NonNull LifecycleOwner owner) { Log.d(TAG, "onCreate"); }

    @Override
    public void onStart(@NonNull LifecycleOwner owner) { Log.d(TAG, "onStart"); }

    @Override
    public void onResume(@NonNull LifecycleOwner owner) { Log.d(TAG, "onResume"); }

    @Override
    public void onPause(@NonNull LifecycleOwner owner) { Log.d(TAG, "onPause"); }

    @Override
    public void onStop(@NonNull LifecycleOwner owner) { Log.d(TAG, "onStop"); }

    @Override
    public void onDestroy(@NonNull LifecycleOwner owner) { Log.d(TAG, "onDestroy"); }
}
```

运行程序就能看到，当我们旋转屏幕、切换任务，也能看到日志随着生命周期的变化打印出来。

## 参考资料

[Android-Lifecycle超能解析-生命周期的那些事儿 - XBaron](https://www.jianshu.com/p/2c9bcbf092bc)

[Handling Lifecycles with Lifecycle-Aware Components   Part of Android Jetpack. - Android Developer](https://developer.android.google.cn/topic/libraries/architecture/lifecycle#kotlin)

