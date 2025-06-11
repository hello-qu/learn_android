# Retrofit 使用指南

## 1. Retrofit 概述

### 1.1 什么是 Retrofit？
Retrofit 是一个类型安全的 HTTP 客户端，用于 Android 和 Java 平台：
- 将 HTTP API 转换为 Kotlin/Java 接口
- 支持同步和异步请求
- 支持多种数据格式转换
- 提供强大的拦截器机制

### 1.2 主要特性
- 支持 RESTful API
- 支持多种数据格式（JSON、XML等）
- 支持文件上传下载
- 支持请求/响应拦截
- 支持请求取消

## 2. 基本使用

### 2.1 添加依赖
```gradle
dependencies {
    def retrofit_version = "2.9.0"
    implementation "com.squareup.retrofit2:retrofit:$retrofit_version"
    implementation "com.squareup.retrofit2:converter-gson:$retrofit_version"
    implementation "com.squareup.okhttp3:logging-interceptor:4.9.0"
}
```

### 2.2 创建 API 接口
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

### 2.3 配置 Retrofit 实例
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

## 3. 请求方法

### 3.1 GET 请求
```kotlin
interface ApiService {
    // 基本 GET 请求
    @GET("users")
    suspend fun getUsers(): List<User>
    
    // 带查询参数的 GET 请求
    @GET("users")
    suspend fun getUsersByRole(@Query("role") role: String): List<User>
    
    // 带路径参数的 GET 请求
    @GET("users/{id}")
    suspend fun getUserById(@Path("id") id: Int): User
    
    // 带多个查询参数的 GET 请求
    @GET("users")
    suspend fun searchUsers(
        @Query("name") name: String,
        @Query("age") age: Int,
        @Query("role") role: String
    ): List<User>
}
```

### 3.2 POST 请求
```kotlin
interface ApiService {
    // 基本 POST 请求
    @POST("users")
    suspend fun createUser(@Body user: User): User
    
    // 带表单数据的 POST 请求
    @FormUrlEncoded
    @POST("login")
    suspend fun login(
        @Field("username") username: String,
        @Field("password") password: String
    ): LoginResponse
    
    // 带文件上传的 POST 请求
    @Multipart
    @POST("upload")
    suspend fun uploadFile(
        @Part("description") description: RequestBody,
        @Part file: MultipartBody.Part
    ): UploadResponse
}
```

### 3.3 PUT 和 DELETE 请求
```kotlin
interface ApiService {
    // PUT 请求
    @PUT("users/{id}")
    suspend fun updateUser(
        @Path("id") id: Int,
        @Body user: User
    ): User
    
    // DELETE 请求
    @DELETE("users/{id}")
    suspend fun deleteUser(@Path("id") id: Int)
}
```

## 4. 数据转换

### 4.1 使用 Gson 转换器
```kotlin
// 添加 Gson 转换器
private val retrofit = Retrofit.Builder()
    .baseUrl(BASE_URL)
    .addConverterFactory(GsonConverterFactory.create())
    .build()

// 自定义 Gson 配置
private val gson = GsonBuilder()
    .setDateFormat("yyyy-MM-dd HH:mm:ss")
    .registerTypeAdapter(Date::class.java, DateDeserializer())
    .create()

private val retrofit = Retrofit.Builder()
    .baseUrl(BASE_URL)
    .addConverterFactory(GsonConverterFactory.create(gson))
    .build()
```

### 4.2 自定义转换器
```kotlin
class CustomConverterFactory : Converter.Factory() {
    override fun responseBodyConverter(
        type: Type,
        annotations: Array<Annotation>,
        retrofit: Retrofit
    ): Converter<ResponseBody, *>? {
        return CustomResponseConverter<Any>(type)
    }
    
    override fun requestBodyConverter(
        type: Type,
        parameterAnnotations: Array<Annotation>,
        methodAnnotations: Array<Annotation>,
        retrofit: Retrofit
    ): Converter<*, RequestBody>? {
        return CustomRequestConverter<Any>(type)
    }
}
```

## 5. 拦截器

### 5.1 日志拦截器
```kotlin
class LoggingInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        
        // 打印请求信息
        Log.d("Retrofit", "Request: ${request.method} ${request.url}")
        Log.d("Retrofit", "Headers: ${request.headers}")
        
        val response = chain.proceed(request)
        
        // 打印响应信息
        Log.d("Retrofit", "Response: ${response.code}")
        Log.d("Retrofit", "Response Body: ${response.body?.string()}")
        
        return response
    }
}
```

### 5.2 认证拦截器
```kotlin
class AuthInterceptor(private val token: String) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request().newBuilder()
            .addHeader("Authorization", "Bearer $token")
            .build()
        return chain.proceed(request)
    }
}
```

## 6. 错误处理

### 6.1 自定义错误处理
```kotlin
class ApiErrorHandler {
    fun handleError(throwable: Throwable): String {
        return when (throwable) {
            is HttpException -> {
                when (throwable.code()) {
                    401 -> "未授权，请重新登录"
                    403 -> "拒绝访问"
                    404 -> "请求的资源不存在"
                    500 -> "服务器内部错误"
                    else -> "网络错误: ${throwable.code()}"
                }
            }
            is IOException -> "网络连接错误"
            is JsonParseException -> "数据解析错误"
            else -> "未知错误: ${throwable.message}"
        }
    }
}
```

### 6.2 统一错误处理
```kotlin
class ErrorHandlingInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        val response = chain.proceed(request)
        
        if (!response.isSuccessful) {
            val errorBody = response.errorBody?.string()
            throw ApiException(response.code, errorBody ?: "Unknown error")
        }
        
        return response
    }
}

class ApiException(val code: Int, override val message: String) : Exception(message)
```

## 7. 实践练习

### 7.1 用户管理 API
```kotlin
interface UserApi {
    @GET("users")
    suspend fun getUsers(): List<User>
    
    @POST("users")
    suspend fun createUser(@Body user: User): User
    
    @PUT("users/{id}")
    suspend fun updateUser(@Path("id") id: Int, @Body user: User): User
    
    @DELETE("users/{id}")
    suspend fun deleteUser(@Path("id") id: Int)
}

class UserRepository(private val api: UserApi) {
    suspend fun getUsers(): Result<List<User>> = try {
        Result.success(api.getUsers())
    } catch (e: Exception) {
        Result.failure(e)
    }
    
    suspend fun createUser(user: User): Result<User> = try {
        Result.success(api.createUser(user))
    } catch (e: Exception) {
        Result.failure(e)
    }
}
```

## 8. 最佳实践

### 8.1 性能优化
- 使用连接池
- 实现请求缓存
- 合理设置超时时间
- 使用协程进行异步操作

### 8.2 代码组织
- 使用 Repository 模式
- 实现依赖注入
- 统一错误处理
- 使用接口定义 API

## 9. 注意事项

### 9.1 网络请求
- 处理网络异常
- 实现请求重试
- 管理请求生命周期
- 处理并发请求

### 9.2 安全性
- 使用 HTTPS
- 实现请求签名
- 加密敏感数据
- 处理证书验证

## 10. 下一步
完成 Retrofit 学习后，您可以：
1. 学习 [JSON 解析](./docs/18_json_parsing.md)
2. 了解 [协程基础](./docs/19_coroutines_basics.md)
3. 开始实践项目开发

## 11. 参考资料
- [Retrofit 官方文档](https://square.github.io/retrofit/)
- [OkHttp 官方文档](https://square.github.io/okhttp/)
- [Android 网络请求最佳实践](https://developer.android.com/training/basics/network-ops) 