# Android Flow 使用指南

## 1. Flow 概述

### 1.1 什么是 Flow
- 冷流数据流
- 响应式编程
- 协程集成
- 背压处理

### 1.2 Flow 的优势
- 异步数据流处理
- 结构化并发
- 取消支持
- 异常处理

## 2. 基础使用

### 2.1 添加依赖
```gradle
dependencies {
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.1"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.1"
}
```

### 2.2 创建 Flow
```kotlin
// 基本 Flow 创建
val flow = flow {
    for (i in 1..5) {
        emit(i)
        delay(100)
    }
}

// 使用 flowOf
val flow = flowOf(1, 2, 3, 4, 5)

// 使用 asFlow
val flow = (1..5).asFlow()
```

### 2.3 收集 Flow
```kotlin
// 在协程作用域中收集
viewModelScope.launch {
    flow.collect { value ->
        // 处理每个值
    }
}

// 使用 collectLatest
viewModelScope.launch {
    flow.collectLatest { value ->
        // 只处理最新的值
    }
}
```

## 3. Flow 操作符

### 3.1 转换操作符
```kotlin
// map 转换
flow.map { it * 2 }

// filter 过滤
flow.filter { it % 2 == 0 }

// transform 复杂转换
flow.transform { value ->
    emit(value * 2)
    emit(value * 3)
}
```

### 3.2 组合操作符
```kotlin
// zip 组合
val flow1 = flowOf(1, 2, 3)
val flow2 = flowOf("a", "b", "c")
flow1.zip(flow2) { number, letter ->
    "$number$letter"
}

// combine 组合
val flow1 = flowOf(1, 2, 3)
val flow2 = flowOf("a", "b", "c")
flow1.combine(flow2) { number, letter ->
    "$number$letter"
}
```

### 3.3 缓冲操作符
```kotlin
// buffer 缓冲
flow.buffer()

// conflate 合并
flow.conflate()

// collectLatest 收集最新值
flow.collectLatest { value ->
    // 处理最新值
}
```

## 4. 状态流

### 4.1 StateFlow
```kotlin
class UserViewModel : ViewModel() {
    private val _userState = MutableStateFlow<User?>(null)
    val userState: StateFlow<User?> = _userState.asStateFlow()
    
    fun updateUser(user: User) {
        _userState.value = user
    }
}

// 在 UI 中收集
viewLifecycleOwner.lifecycleScope.launch {
    viewModel.userState.collect { user ->
        // 更新 UI
    }
}
```

### 4.2 SharedFlow
```kotlin
class EventBus {
    private val _events = MutableSharedFlow<Event>()
    val events = _events.asSharedFlow()
    
    suspend fun emit(event: Event) {
        _events.emit(event)
    }
}

// 在 UI 中收集
viewLifecycleOwner.lifecycleScope.launch {
    eventBus.events.collect { event ->
        // 处理事件
    }
}
```

## 5. 实践练习

### 5.1 网络请求
```kotlin
class UserRepository {
    fun getUsers(): Flow<List<User>> = flow {
        try {
            val users = api.getUsers()
            emit(users)
        } catch (e: Exception) {
            throw e
        }
    }
}

class UserViewModel : ViewModel() {
    private val _users = MutableStateFlow<List<User>>(emptyList())
    val users: StateFlow<List<User>> = _users.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            userRepository.getUsers()
                .catch { e ->
                    // 处理错误
                }
                .collect { users ->
                    _users.value = users
                }
        }
    }
}
```

### 5.2 实时数据更新
```kotlin
class StockRepository {
    fun getStockUpdates(): Flow<StockUpdate> = flow {
        while (true) {
            val update = api.getLatestStockPrice()
            emit(update)
            delay(1000)
        }
    }
}

class StockViewModel : ViewModel() {
    private val _stockPrice = MutableStateFlow<BigDecimal?>(null)
    val stockPrice: StateFlow<BigDecimal?> = _stockPrice.asStateFlow()
    
    fun startUpdates() {
        viewModelScope.launch {
            stockRepository.getStockUpdates()
                .collect { update ->
                    _stockPrice.value = update.price
                }
        }
    }
}
```

## 6. 最佳实践

### 6.1 性能优化
- 使用适当的操作符
- 实现背压处理
- 避免不必要的转换
- 使用缓冲操作符

### 6.2 错误处理
```kotlin
flow
    .catch { e ->
        // 处理错误
        emitAll(fallbackFlow)
    }
    .onEach { value ->
        // 处理每个值
    }
    .collect()
```

## 7. 注意事项

### 7.1 生命周期
- 在适当的生命周期收集
- 处理配置更改
- 避免内存泄漏
- 正确取消收集

### 7.2 线程安全
- 使用适当的调度器
- 避免并发修改
- 处理线程切换
- 使用线程安全的数据结构

## 8. 下一步
完成 Flow 学习后，您可以：
1. 开始 [待办事项应用](./docs/22_todo_app.md) 的开发
2. 深入学习 [协程基础](./docs/19_coroutines_basics.md)
3. 了解 [LiveData 和 ViewModel](./docs/20_livedata_viewmodel.md)

## 9. 参考资料
- [Kotlin Flow 官方文档](https://kotlinlang.org/docs/flow.html)
- [Android 协程最佳实践](https://developer.android.com/kotlin/coroutines/coroutines-best-practices)
- [Flow 设计文档](https://github.com/Kotlin/KEEP/blob/master/proposals/kotlin-flow.md) 