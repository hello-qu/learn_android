# Android LiveData 和 ViewModel 指南

## 1. 概述

### 1.1 什么是 LiveData
- 可观察的数据持有类
- 生命周期感知
- 自动处理生命周期
- 确保 UI 与数据状态一致

### 1.2 什么是 ViewModel
- 存储和管理 UI 相关数据
- 处理配置更改
- 避免内存泄漏
- 实现 MVVM 架构

## 2. 基础使用

### 2.1 添加依赖
```gradle
dependencies {
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.6.2"
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.6.2"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.6.2"
}
```

### 2.2 ViewModel 基本使用
```kotlin
class UserViewModel : ViewModel() {
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    fun loadUsers() {
        viewModelScope.launch {
            // 加载用户数据
            val result = userRepository.getUsers()
            _users.value = result
        }
    }
    
    override fun onCleared() {
        super.onCleared()
        // 清理资源
    }
}

// 在 Activity 中使用
class MainActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // 观察数据变化
        viewModel.users.observe(this) { users ->
            // 更新 UI
        }
        
        // 加载数据
        viewModel.loadUsers()
    }
}
```

### 2.3 LiveData 基本使用
```kotlin
class UserViewModel : ViewModel() {
    // 私有可变 LiveData
    private val _userName = MutableLiveData<String>()
    // 公开不可变 LiveData
    val userName: LiveData<String> = _userName
    
    fun updateUserName(name: String) {
        _userName.value = name
    }
    
    fun updateUserNameAsync(name: String) {
        viewModelScope.launch {
            // 异步操作
            _userName.postValue(name)
        }
    }
}
```

## 3. LiveData 高级特性

### 3.1 数据转换
```kotlin
class UserViewModel : ViewModel() {
    private val _user = MutableLiveData<User>()
    
    // 转换数据
    val userName: LiveData<String> = _user.map { it.name }
    
    // 组合多个 LiveData
    val userInfo: LiveData<UserInfo> = MediatorLiveData<UserInfo>().apply {
        addSource(_user) { user ->
            value = UserInfo(user.name, user.email)
        }
    }
}
```

### 3.2 自定义 LiveData
```kotlin
class StockLiveData : LiveData<BigDecimal>() {
    private val stockManager = StockManager()
    
    private val listener = { price: BigDecimal ->
        value = price
    }
    
    override fun onActive() {
        stockManager.requestPriceUpdates(listener)
    }
    
    override fun onInactive() {
        stockManager.removeUpdates(listener)
    }
}
```

## 4. ViewModel 高级特性

### 4.1 保存状态
```kotlin
class SavedStateViewModel(
    private val savedStateHandle: SavedStateHandle
) : ViewModel() {
    var counter: Int
        get() = savedStateHandle.get<Int>("counter") ?: 0
        set(value) = savedStateHandle.set("counter", value)
}

// 在 Activity 中使用
class MainActivity : AppCompatActivity() {
    private val viewModel: SavedStateViewModel by viewModels {
        SavedStateViewModelFactory(application, this)
    }
}
```

### 4.2 共享 ViewModel
```kotlin
class SharedViewModel : ViewModel() {
    private val _selectedItem = MutableLiveData<Item>()
    val selectedItem: LiveData<Item> = _selectedItem
    
    fun select(item: Item) {
        _selectedItem.value = item
    }
}

// 在 Fragment 中使用
class ListFragment : Fragment() {
    private val model: SharedViewModel by activityViewModels()
}
```

## 5. 实践练习

### 5.1 用户管理
```kotlin
class UserViewModel : ViewModel() {
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    private val _loading = MutableLiveData<Boolean>()
    val loading: LiveData<Boolean> = _loading
    
    private val _error = MutableLiveData<String>()
    val error: LiveData<String> = _error
    
    fun loadUsers() {
        viewModelScope.launch {
            _loading.value = true
            try {
                val result = userRepository.getUsers()
                _users.value = result
            } catch (e: Exception) {
                _error.value = e.message
            } finally {
                _loading.value = false
            }
        }
    }
}
```

### 5.2 表单处理
```kotlin
class LoginViewModel : ViewModel() {
    private val _formState = MutableLiveData<FormState>()
    val formState: LiveData<FormState> = _formState
    
    fun login(username: String, password: String) {
        viewModelScope.launch {
            _formState.value = FormState.Loading
            try {
                val result = authRepository.login(username, password)
                _formState.value = FormState.Success(result)
            } catch (e: Exception) {
                _formState.value = FormState.Error(e.message)
            }
        }
    }
}
```

## 6. 最佳实践

### 6.1 架构设计
- 使用 MVVM 架构
- 分离关注点
- 使用 Repository 模式
- 实现单向数据流

### 6.2 性能优化
- 避免在 ViewModel 中持有 View 引用
- 使用 Transformations 处理数据转换
- 合理使用 SavedStateHandle
- 及时清理资源

## 7. 注意事项

### 7.1 生命周期
- 正确处理配置更改
- 避免内存泄漏
- 注意观察者注册时机
- 处理后台任务

### 7.2 数据更新
- 使用 postValue 在后台线程更新
- 使用 value 在主线程更新
- 避免频繁更新
- 处理数据一致性

## 8. 下一步
完成 LiveData 和 ViewModel 学习后，您可以：
1. 学习 [依赖注入](./docs/21_dependency_injection.md)
2. 了解 [Jetpack Compose](./docs/22_jetpack_compose.md)
3. 开始实践项目开发

## 9. 参考资料
- [Android 架构组件官方文档](https://developer.android.com/topic/libraries/architecture)
- [ViewModel 指南](https://developer.android.com/topic/libraries/architecture/viewmodel)
- [LiveData 指南](https://developer.android.com/topic/libraries/architecture/livedata) 