# Android 协程基础指南

## 1. 协程概述

### 1.1 什么是协程
- 轻量级线程
- 结构化并发
- 简化异步编程
- 取消和异常处理

### 1.2 协程的优势
- 代码更简洁
- 避免回调地狱
- 更好的错误处理
- 内存占用更少

## 2. 基础使用

### 2.1 添加依赖
```gradle
dependencies {
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.1"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.1"
}
```

### 2.2 启动协程
```kotlin
// 使用 GlobalScope（不推荐）
GlobalScope.launch {
    // 协程代码
}

// 使用 CoroutineScope
class MyActivity : AppCompatActivity() {
    private val scope = CoroutineScope(Dispatchers.Main + Job())
    
    fun startCoroutine() {
        scope.launch {
            // 协程代码
        }
    }
    
    override fun onDestroy() {
        super.onDestroy()
        scope.cancel() // 取消所有协程
    }
}
```

### 2.3 协程作用域
```kotlin
// 创建作用域
val scope = CoroutineScope(Dispatchers.Main + Job())

// 使用 viewModelScope（在 ViewModel 中）
class MyViewModel : ViewModel() {
    fun doWork() {
        viewModelScope.launch {
            // 协程代码
        }
    }
}

// 使用 lifecycleScope（在 Activity/Fragment 中）
class MyActivity : AppCompatActivity() {
    fun doWork() {
        lifecycleScope.launch {
            // 协程代码
        }
    }
}
```

## 3. 协程调度器

### 3.1 常用调度器
```kotlin
// 主线程调度器
launch(Dispatchers.Main) {
    // UI 操作
}

// IO 调度器
launch(Dispatchers.IO) {
    // 网络请求、文件操作等
}

// 默认调度器
launch(Dispatchers.Default) {
    // CPU 密集型任务
}

// 自定义调度器
val customDispatcher = Dispatchers.IO.limitedParallelism(4)
launch(customDispatcher) {
    // 限制并发数的任务
}
```

### 3.2 切换调度器
```kotlin
launch(Dispatchers.Main) {
    // UI 操作
    withContext(Dispatchers.IO) {
        // 切换到 IO 线程
    }
    // 回到主线程
}
```

## 4. 协程构建器

### 4.1 launch
```kotlin
// 启动一个协程，不返回结果
val job = scope.launch {
    // 协程代码
}

// 带参数的 launch
val job = scope.launch(
    context = Dispatchers.IO,
    start = CoroutineStart.LAZY
) {
    // 协程代码
}
```

### 4.2 async
```kotlin
// 启动一个协程，返回 Deferred 对象
val deferred = scope.async {
    // 返回结果
    "结果"
}

// 等待结果
val result = deferred.await()
```

### 4.3 runBlocking
```kotlin
// 阻塞当前线程直到协程完成
runBlocking {
    // 协程代码
}
```

## 5. 协程上下文

### 5.1 上下文元素
```kotlin
// 组合上下文
val context = Dispatchers.Main + Job() + CoroutineName("MyCoroutine")

// 使用上下文
launch(context) {
    // 协程代码
}
```

### 5.2 上下文继承
```kotlin
val parentJob = Job()
val scope = CoroutineScope(Dispatchers.Main + parentJob)

// 子协程继承父协程的上下文
scope.launch {
    // 子协程
    launch {
        // 孙协程
    }
}
```

## 6. 异常处理

### 6.1 异常捕获
```kotlin
// 使用 try-catch
launch {
    try {
        // 可能抛出异常的代码
    } catch (e: Exception) {
        // 处理异常
    }
}

// 使用 CoroutineExceptionHandler
val handler = CoroutineExceptionHandler { _, exception ->
    // 处理异常
}

launch(handler) {
    // 协程代码
}
```

### 6.2 异常传播
```kotlin
// 异常传播到父协程
val scope = CoroutineScope(Dispatchers.Main + Job())
scope.launch {
    launch {
        throw Exception("错误")
    }
}

// 使用 SupervisorJob 防止异常传播
val scope = CoroutineScope(Dispatchers.Main + SupervisorJob())
```

## 7. 协程取消

### 7.1 取消协程
```kotlin
val job = scope.launch {
    // 协程代码
}

// 取消协程
job.cancel()

// 取消作用域中的所有协程
scope.cancel()
```

### 7.2 取消检查
```kotlin
launch {
    while (isActive) { // 检查协程是否被取消
        // 协程代码
    }
}

// 使用 yield() 让出执行权
launch {
    while (true) {
        yield() // 检查取消状态
        // 协程代码
    }
}
```

## 8. 实践练习

### 8.1 网络请求
```kotlin
class UserRepository {
    suspend fun fetchUser(id: Int): User {
        return withContext(Dispatchers.IO) {
            // 模拟网络请求
            delay(1000)
            User(id, "张三")
        }
    }
}

class UserViewModel : ViewModel() {
    private val repository = UserRepository()
    
    fun loadUser(id: Int) {
        viewModelScope.launch {
            try {
                val user = repository.fetchUser(id)
                // 处理结果
            } catch (e: Exception) {
                // 处理错误
            }
        }
    }
}
```

### 8.2 并发操作
```kotlin
class DataManager {
    suspend fun loadData(): Result {
        return coroutineScope {
            val users = async { fetchUsers() }
            val posts = async { fetchPosts() }
            
            Result(
                users = users.await(),
                posts = posts.await()
            )
        }
    }
}
```

## 9. 最佳实践

### 9.1 性能优化
- 使用适当的调度器
- 避免过度创建协程
- 及时取消不需要的协程
- 使用结构化并发

### 9.2 代码组织
- 使用 ViewModel 管理协程
- 将异步操作封装在 Repository 层
- 使用 suspend 函数
- 正确处理异常

## 10. 注意事项

### 10.1 内存泄漏
- 避免使用 GlobalScope
- 在适当的生命周期取消协程
- 使用 weakReference 持有 Activity/Fragment
- 注意协程作用域的生命周期

### 10.2 线程安全
- 使用 withContext 切换线程
- 避免在协程中直接修改 UI
- 注意共享资源的访问
- 使用线程安全的数据结构

## 11. 下一步
完成协程基础学习后，您可以：
1. 学习 [LiveData 和 ViewModel](./docs/20_livedata_viewmodel.md)
2. 了解 [依赖注入](./docs/21_dependency_injection.md)
3. 开始实践项目开发

## 12. 参考资料
- [Kotlin 协程官方文档](https://kotlinlang.org/docs/coroutines-guide.html)
- [Android 协程最佳实践](https://developer.android.com/kotlin/coroutines/coroutines-best-practices)
- [协程设计文档](https://github.com/Kotlin/KEEP/blob/master/proposals/kotlin-coroutines.md) 