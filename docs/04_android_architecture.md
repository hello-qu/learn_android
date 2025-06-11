# Android 应用架构

## 1. 架构概述

### 1.1 为什么需要架构？
- 提高代码可维护性
- 便于团队协作
- 降低测试难度
- 提高代码复用性

### 1.2 架构原则
- **关注点分离**：每个组件只负责自己的功能
- **单一职责**：每个类只做一件事
- **依赖倒置**：依赖抽象而不是具体实现
- **可测试性**：便于单元测试

## 2. 传统架构

### 2.1 MVC 架构
```
Model（模型）
├── 数据模型
├── 业务逻辑
└── 数据访问

View（视图）
├── 布局文件
└── 自定义视图

Controller（控制器）
├── Activity
└── Fragment
```

### 2.2 MVC 示例
```kotlin
// Model
data class User(
    val id: Int,
    val name: String,
    val email: String
)

// View (activity_main.xml)
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout>
    <EditText android:id="@+id/nameInput" />
    <Button android:id="@+id/saveButton" />
</LinearLayout>

// Controller
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        saveButton.setOnClickListener {
            val name = nameInput.text.toString()
            // 处理数据
        }
    }
}
```

## 3. 现代架构（MVVM + Clean Architecture）

### 3.1 架构组件
```
UI Layer（表现层）
├── Activity/Fragment
├── ViewModel
└── UI State

Domain Layer（领域层）
├── Use Cases
├── Repository Interfaces
└── Domain Models

Data Layer（数据层）
├── Repositories
├── Data Sources
└── Data Models
```

### 3.2 各层职责

#### UI Layer
```kotlin
// UI State
data class UserUiState(
    val name: String = "",
    val isLoading: Boolean = false,
    val error: String? = null
)

// ViewModel
class UserViewModel(
    private val getUserUseCase: GetUserUseCase
) : ViewModel() {
    private val _uiState = MutableStateFlow(UserUiState())
    val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()
    
    fun loadUser(userId: Int) {
        viewModelScope.launch {
            _uiState.value = _uiState.value.copy(isLoading = true)
            try {
                val user = getUserUseCase(userId)
                _uiState.value = _uiState.value.copy(
                    name = user.name,
                    isLoading = false
                )
            } catch (e: Exception) {
                _uiState.value = _uiState.value.copy(
                    error = e.message,
                    isLoading = false
                )
            }
        }
    }
}

// Activity
class MainActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        lifecycleScope.launch {
            viewModel.uiState.collect { state ->
                // 更新 UI
            }
        }
    }
}
```

#### Domain Layer
```kotlin
// Use Case
class GetUserUseCase(
    private val userRepository: UserRepository
) {
    suspend operator fun invoke(userId: Int): User {
        return userRepository.getUser(userId)
    }
}

// Repository Interface
interface UserRepository {
    suspend fun getUser(userId: Int): User
    suspend fun saveUser(user: User)
}
```

#### Data Layer
```kotlin
// Repository Implementation
class UserRepositoryImpl(
    private val userDao: UserDao,
    private val api: UserApi
) : UserRepository {
    override suspend fun getUser(userId: Int): User {
        return try {
            // 先尝试从本地获取
            userDao.getUser(userId) ?: run {
                // 如果本地没有，从网络获取
                val user = api.getUser(userId)
                userDao.insertUser(user)
                user
            }
        } catch (e: Exception) {
            // 如果网络请求失败，返回本地数据
            userDao.getUser(userId) ?: throw e
        }
    }
}

// Data Source
@Dao
interface UserDao {
    @Query("SELECT * FROM users WHERE id = :userId")
    suspend fun getUser(userId: Int): User?
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUser(user: User)
}
```

## 4. 依赖注入

### 4.1 Hilt 使用示例
```kotlin
// 模块定义
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    @Singleton
    fun provideUserRepository(
        userDao: UserDao,
        api: UserApi
    ): UserRepository {
        return UserRepositoryImpl(userDao, api)
    }
}

// ViewModel 注入
@HiltViewModel
class UserViewModel @Inject constructor(
    private val getUserUseCase: GetUserUseCase
) : ViewModel()

// Activity 注入
@AndroidEntryPoint
class MainActivity : AppCompatActivity()
```

## 5. 数据流

### 5.1 单向数据流
```kotlin
// UI Event
sealed class UserEvent {
    data class LoadUser(val userId: Int) : UserEvent()
    data class UpdateName(val name: String) : UserEvent()
}

// ViewModel
class UserViewModel : ViewModel() {
    private val _events = MutableSharedFlow<UserEvent>()
    
    fun onEvent(event: UserEvent) {
        viewModelScope.launch {
            _events.emit(event)
        }
    }
}
```

## 6. 最佳实践

### 6.1 错误处理
```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val exception: Exception) : Result<Nothing>()
    object Loading : Result<Nothing>()
}

class UserViewModel : ViewModel() {
    private val _state = MutableStateFlow<Result<User>>(Result.Loading)
    
    fun loadUser(userId: Int) {
        viewModelScope.launch {
            _state.value = Result.Loading
            try {
                val user = repository.getUser(userId)
                _state.value = Result.Success(user)
            } catch (e: Exception) {
                _state.value = Result.Error(e)
            }
        }
    }
}
```

### 6.2 测试策略
```kotlin
// ViewModel 测试
@Test
fun `when loading user succeeds, state is updated with user data`() = runTest {
    // Given
    val user = User(1, "Test User")
    coEvery { repository.getUser(1) } returns user
    
    // When
    viewModel.loadUser(1)
    
    // Then
    assert(viewModel.state.value is Result.Success)
    assertEquals(user, (viewModel.state.value as Result.Success).data)
}
```

## 7. 项目实战建议

### 7.1 项目结构
```
app/
├── data/
│   ├── local/
│   ├── remote/
│   └── repository/
├── domain/
│   ├── model/
│   ├── repository/
│   └── usecase/
└── ui/
    ├── common/
    ├── feature1/
    └── feature2/
```

### 7.2 模块化
- 按功能模块拆分
- 使用动态特性模块
- 共享核心模块

## 8. 下一步
完成架构学习后，您可以：
1. 学习 [Activity 生命周期](./05_activity_lifecycle.md)
2. 了解 [Intent 和 Intent Filter](./06_intent_and_filter.md)
3. 开始实践项目开发

## 9. 参考资料
- [Android 架构指南](https://developer.android.com/topic/architecture)
- [Android Jetpack 架构组件](https://developer.android.com/jetpack/guide)
- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) 