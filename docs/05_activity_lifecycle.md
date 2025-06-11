# Activity 生命周期

## 1. Activity 生命周期概述

### 1.1 什么是 Activity？
Activity 是 Android 应用程序的入口点，代表一个具有用户界面的单一屏幕。每个 Activity 都继承自 `AppCompatActivity` 或 `Activity` 类。

### 1.2 生命周期状态
Activity 在其生命周期中会经历以下状态：
- **Created**：Activity 被创建
- **Started**：Activity 可见但不可交互
- **Resumed**：Activity 可见且可交互
- **Paused**：Activity 部分可见但不可交互
- **Stopped**：Activity 完全不可见
- **Destroyed**：Activity 被销毁

## 2. 生命周期回调方法

### 2.1 基本生命周期方法
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        // Activity 创建时调用
        // 初始化 UI 和变量
    }

    override fun onStart() {
        super.onStart()
        // Activity 变为可见时调用
        // 注册广播接收器等
    }

    override fun onResume() {
        super.onResume()
        // Activity 开始与用户交互时调用
        // 恢复动画、更新 UI 等
    }

    override fun onPause() {
        super.onPause()
        // Activity 失去焦点时调用
        // 暂停动画、保存数据等
    }

    override fun onStop() {
        super.onStop()
        // Activity 完全不可见时调用
        // 注销广播接收器等
    }

    override fun onDestroy() {
        super.onDestroy()
        // Activity 被销毁时调用
        // 清理资源、取消网络请求等
    }
}
```

### 2.2 状态保存和恢复
```kotlin
class MainActivity : AppCompatActivity() {
    private var counter = 0

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        // 保存状态
        outState.putInt("counter", counter)
    }

    override fun onRestoreInstanceState(savedInstanceState: Bundle) {
        super.onRestoreInstanceState(savedInstanceState)
        // 恢复状态
        counter = savedInstanceState.getInt("counter")
    }
}
```

## 3. 生命周期场景

### 3.1 正常启动和关闭
```
onCreate() → onStart() → onResume() → onPause() → onStop() → onDestroy()
```

### 3.2 配置更改（如旋转屏幕）
```
onPause() → onStop() → onSaveInstanceState() → onDestroy() → onCreate() → onStart() → onRestoreInstanceState() → onResume()
```

### 3.3 部分可见（如对话框）
```
onResume() → onPause() → onResume()
```

## 4. 生命周期感知组件

### 4.1 ViewModel 使用
```kotlin
class MainActivity : AppCompatActivity() {
    private val viewModel: MainViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // ViewModel 在配置更改时保持数据
        viewModel.counter.observe(this) { count ->
            updateCounterUI(count)
        }
    }
}

class MainViewModel : ViewModel() {
    private val _counter = MutableLiveData(0)
    val counter: LiveData<Int> = _counter

    fun incrementCounter() {
        _counter.value = (_counter.value ?: 0) + 1
    }
}
```

### 4.2 SavedStateHandle 使用
```kotlin
class MainViewModel(
    private val savedStateHandle: SavedStateHandle
) : ViewModel() {
    var counter: Int
        get() = savedStateHandle.get<Int>("counter") ?: 0
        set(value) = savedStateHandle.set("counter", value)
}
```

## 5. 生命周期最佳实践

### 5.1 避免内存泄漏
```kotlin
class MainActivity : AppCompatActivity() {
    private var job: Job? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // 使用 lifecycleScope 避免内存泄漏
        lifecycleScope.launch {
            // 协程代码
        }
    }

    override fun onDestroy() {
        job?.cancel() // 取消协程
        super.onDestroy()
    }
}
```

### 5.2 处理配置更改
```kotlin
// 在 AndroidManifest.xml 中
<activity
    android:name=".MainActivity"
    android:configChanges="orientation|screenSize">
</activity>

// 在 Activity 中
override fun onConfigurationChanged(newConfig: Configuration) {
    super.onConfigurationChanged(newConfig)
    // 处理配置更改
}
```

## 6. 常见问题解决

### 6.1 状态保存问题
```kotlin
class MainActivity : AppCompatActivity() {
    private var isDataLoaded = false

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        if (savedInstanceState == null) {
            // 首次创建
            loadData()
        } else {
            // 恢复状态
            isDataLoaded = savedInstanceState.getBoolean("isDataLoaded")
            if (!isDataLoaded) {
                loadData()
            }
        }
    }

    private fun loadData() {
        // 加载数据
        isDataLoaded = true
    }
}
```

### 6.2 生命周期感知的数据加载
```kotlin
class MainActivity : AppCompatActivity() {
    private val viewModel: MainViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 使用 repeatOnLifecycle 确保在正确的生命周期状态执行
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.data.collect { data ->
                    // 更新 UI
                }
            }
        }
    }
}
```

## 7. 实践练习

### 7.1 计数器应用
```kotlin
class CounterActivity : AppCompatActivity() {
    private val viewModel: CounterViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_counter)
        
        // 观察计数器变化
        viewModel.counter.observe(this) { count ->
            counterText.text = count.toString()
        }
        
        // 按钮点击事件
        incrementButton.setOnClickListener {
            viewModel.increment()
        }
    }
}
```

## 8. 下一步
完成 Activity 生命周期学习后，您可以：
1. 学习 [Intent 和 Intent Filter](./06_intent_and_filter.md)
2. 了解 [Fragment 基础](./07_fragment_basics.md)
3. 开始实践项目开发

## 9. 参考资料
- [Android Activity 生命周期指南](https://developer.android.com/guide/components/activities/activity-lifecycle)
- [Android 生命周期感知组件](https://developer.android.com/topic/libraries/architecture/lifecycle)
- [Android ViewModel 指南](https://developer.android.com/topic/libraries/architecture/viewmodel) 