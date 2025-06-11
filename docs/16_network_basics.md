# Android 网络请求基础指南

## 1. 网络请求概述

### 1.1 权限配置
```xml
<!-- 网络权限 -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

### 1.2 网络安全配置
```xml
<!-- res/xml/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

## 2. Retrofit 使用

### 2.1 添加依赖
```gradle
dependencies {
    def retrofit_version = "2.9.0"
    implementation "com.squareup.retrofit2:retrofit:$retrofit_version"
    implementation "com.squareup.retrofit2:converter-gson:$retrofit_version"
    implementation "com.squareup.okhttp3:logging-interceptor:4.9.0"
}
```

### 2.2 API 接口定义
```kotlin
interface ApiService {
    @GET("users")
    suspend fun getUsers(): List<User>
    
    @GET("users/{id}")
    suspend fun getUserById(@Path("id") id: Int): User
    
    @POST("users")
    suspend fun createUser(@Body user: User): User
    
    @PUT("users/{id}")
    suspend fun updateUser(@Path("id") id: Int, @Body user: User): User
    
    @DELETE("users/{id}")
    suspend fun deleteUser(@Path("id") id: Int)
}
```

### 2.3 Retrofit 配置
```kotlin
object RetrofitClient {
    private const val BASE_URL = "https://api.example.com/"
    
    private val loggingInterceptor = HttpLoggingInterceptor().apply {
        level = HttpLoggingInterceptor.Level.BODY
    }
    
    private val okHttpClient = OkHttpClient.Builder()
        .addInterceptor(loggingInterceptor)
        .connectTimeout(30, TimeUnit.SECONDS)
        .readTimeout(30, TimeUnit.SECONDS)
        .writeTimeout(30, TimeUnit.SECONDS)
        .build()
    
    private val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .client(okHttpClient)
        .addConverterFactory(GsonConverterFactory.create())
        .build()
    
    val apiService: ApiService = retrofit.create(ApiService::class.java)
}
```

## 3. 数据模型

### 3.1 基础模型
```kotlin
data class User(
    val id: Int,
    val name: String,
    val email: String,
    val phone: String
)

data class ApiResponse<T>(
    val code: Int,
    val message: String,
    val data: T
)
```

### 3.2 请求/响应模型
```kotlin
data class LoginRequest(
    val username: String,
    val password: String
)

data class LoginResponse(
    val token: String,
    val user: User
)
```

## 4. 网络请求实现

### 4.1 Repository 层
```kotlin
class UserRepository(private val apiService: ApiService) {
    suspend fun getUsers(): Result<List<User>> = try {
        val users = apiService.getUsers()
        Result.success(users)
    } catch (e: Exception) {
        Result.failure(e)
    }
    
    suspend fun createUser(user: User): Result<User> = try {
        val createdUser = apiService.createUser(user)
        Result.success(createdUser)
    } catch (e: Exception) {
        Result.failure(e)
    }
}
```

### 4.2 ViewModel 层
```kotlin
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    private val _users = MutableStateFlow<List<User>>(emptyList())
    val users: StateFlow<List<User>> = _users.asStateFlow()
    
    private val _error = MutableSharedFlow<String>()
    val error: SharedFlow<String> = _error.asSharedFlow()
    
    fun loadUsers() = viewModelScope.launch {
        repository.getUsers()
            .onSuccess { users ->
                _users.value = users
            }
            .onFailure { e ->
                _error.emit(e.message ?: "Unknown error")
            }
    }
}
```

## 5. 网络状态监控

### 5.1 网络状态检查
```kotlin
class NetworkMonitor(private val context: Context) {
    private val connectivityManager = 
        context.getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
    
    fun isNetworkAvailable(): Boolean {
        val network = connectivityManager.activeNetwork
        val capabilities = connectivityManager.getNetworkCapabilities(network)
        return capabilities != null && (
            capabilities.hasTransport(NetworkCapabilities.TRANSPORT_WIFI) ||
            capabilities.hasTransport(NetworkCapabilities.TRANSPORT_CELLULAR)
        )
    }
}
```

### 5.2 网络状态监听
```kotlin
class NetworkStateMonitor(private val context: Context) {
    private val connectivityManager = 
        context.getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
    
    private val _networkState = MutableStateFlow(false)
    val networkState: StateFlow<Boolean> = _networkState.asStateFlow()
    
    private val networkCallback = object : ConnectivityManager.NetworkCallback() {
        override fun onAvailable(network: Network) {
            _networkState.value = true
        }
        
        override fun onLost(network: Network) {
            _networkState.value = false
        }
    }
    
    fun startMonitoring() {
        val request = NetworkRequest.Builder()
            .addCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
            .build()
        connectivityManager.registerNetworkCallback(request, networkCallback)
    }
    
    fun stopMonitoring() {
        connectivityManager.unregisterNetworkCallback(networkCallback)
    }
}
```

## 6. 错误处理

### 6.1 自定义异常
```kotlin
sealed class NetworkException : Exception() {
    data class NoInternetConnection(override val message: String = "No internet connection") : NetworkException()
    data class ServerError(override val message: String) : NetworkException()
    data class ClientError(override val message: String) : NetworkException()
    data class UnknownError(override val message: String) : NetworkException()
}
```

### 6.2 错误处理工具
```kotlin
object NetworkErrorHandler {
    fun handleError(throwable: Throwable): String {
        return when (throwable) {
            is IOException -> "网络连接错误"
            is HttpException -> "服务器错误: ${throwable.code()}"
            is JsonParseException -> "数据解析错误"
            else -> "未知错误: ${throwable.message}"
        }
    }
}
```

## 7. 实践练习

### 7.1 登录功能
```kotlin
class LoginViewModel(private val repository: UserRepository) : ViewModel() {
    private val _loginState = MutableStateFlow<LoginState>(LoginState.Initial)
    val loginState: StateFlow<LoginState> = _loginState.asStateFlow()
    
    fun login(username: String, password: String) = viewModelScope.launch {
        _loginState.value = LoginState.Loading
        try {
            val response = repository.login(LoginRequest(username, password))
            _loginState.value = LoginState.Success(response)
        } catch (e: Exception) {
            _loginState.value = LoginState.Error(e.message ?: "登录失败")
        }
    }
}

sealed class LoginState {
    object Initial : LoginState()
    object Loading : LoginState()
    data class Success(val response: LoginResponse) : LoginState()
    data class Error(val message: String) : LoginState()
}
```

## 8. 最佳实践

### 8.1 性能优化
- 使用协程进行异步操作
- 实现请求缓存
- 使用连接池
- 压缩请求数据

### 8.2 安全考虑
- 使用 HTTPS
- 实现请求签名
- 加密敏感数据
- 处理证书验证

## 9. 注意事项

### 9.1 网络请求
- 处理超时情况
- 实现重试机制
- 处理并发请求
- 管理请求生命周期

### 9.2 用户体验
- 显示加载状态
- 处理错误提示
- 实现离线模式
- 优化网络提示

## 10. 下一步
完成网络请求基础学习后，您可以：
1. 学习 [图片加载](./docs/17_image_loading.md)
2. 了解 [依赖注入](./docs/18_dependency_injection.md)
3. 开始实践项目开发

## 11. 参考资料
- [Retrofit 官方文档](https://square.github.io/retrofit/)
- [OkHttp 官方文档](https://square.github.io/okhttp/)
- [Android 网络请求最佳实践](https://developer.android.com/training/basics/network-ops) 