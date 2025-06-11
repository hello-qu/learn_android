# Android JSON 解析指南

## 1. JSON 解析概述

### 1.1 常用解析库
- **Gson**: Google 开发的 JSON 解析库
- **Moshi**: Square 开发的现代 JSON 解析库
- **Kotlinx Serialization**: Kotlin 官方序列化库
- **Jackson**: 功能强大的 JSON 处理库

### 1.2 选择建议
- 新项目推荐使用 Kotlinx Serialization
- 需要与 Java 互操作时选择 Gson
- 追求性能时考虑 Moshi
- 需要复杂功能时使用 Jackson

## 2. Gson 使用

### 2.1 添加依赖
```gradle
dependencies {
    implementation 'com.google.code.gson:gson:2.10.1'
}
```

### 2.2 基本使用
```kotlin
// 创建 Gson 实例
val gson = Gson()

// 对象转 JSON
data class User(
    val id: Int,
    val name: String,
    val email: String
)

val user = User(1, "张三", "zhangsan@example.com")
val json = gson.toJson(user)

// JSON 转对象
val json = """{"id":1,"name":"张三","email":"zhangsan@example.com"}"""
val user = gson.fromJson(json, User::class.java)
```

### 2.3 高级配置
```kotlin
// 自定义 Gson 配置
val gson = GsonBuilder()
    .setDateFormat("yyyy-MM-dd HH:mm:ss")
    .setPrettyPrinting()
    .serializeNulls()
    .create()

// 使用 TypeToken 处理泛型
val type = object : TypeToken<List<User>>() {}.type
val users = gson.fromJson<List<User>>(json, type)
```

## 3. Kotlinx Serialization

### 3.1 添加依赖
```gradle
plugins {
    id 'org.jetbrains.kotlin.plugin.serialization' version '1.8.0'
}

dependencies {
    implementation "org.jetbrains.kotlinx:kotlinx-serialization-json:1.5.1"
}
```

### 3.2 基本使用
```kotlin
// 定义可序列化类
@Serializable
data class User(
    val id: Int,
    val name: String,
    val email: String
)

// 创建序列化器
val json = Json {
    prettyPrint = true
    isLenient = true
    ignoreUnknownKeys = true
}

// 序列化和反序列化
val user = User(1, "张三", "zhangsan@example.com")
val jsonString = json.encodeToString(user)
val decodedUser = json.decodeFromString<User>(jsonString)
```

### 3.3 自定义序列化
```kotlin
@Serializable
data class User(
    val id: Int,
    val name: String,
    @Serializable(with = DateSerializer::class)
    val createdAt: Date
)

object DateSerializer : KSerializer<Date> {
    override val descriptor = PrimitiveSerialDescriptor("Date", PrimitiveKind.STRING)
    
    override fun serialize(encoder: Encoder, value: Date) {
        encoder.encodeString(value.time.toString())
    }
    
    override fun deserialize(decoder: Decoder): Date {
        return Date(decoder.decodeString().toLong())
    }
}
```

## 4. Moshi 使用

### 4.1 添加依赖
```gradle
dependencies {
    implementation "com.squareup.moshi:moshi-kotlin:1.14.0"
    kapt "com.squareup.moshi:moshi-kotlin-codegen:1.14.0"
}
```

### 4.2 基本使用
```kotlin
// 创建 Moshi 实例
val moshi = Moshi.Builder()
    .add(KotlinJsonAdapterFactory())
    .build()

// 创建适配器
val userAdapter = moshi.adapter(User::class.java)

// 序列化和反序列化
val user = User(1, "张三", "zhangsan@example.com")
val json = userAdapter.toJson(user)
val decodedUser = userAdapter.fromJson(json)
```

### 4.3 自定义适配器
```kotlin
class DateAdapter {
    @ToJson
    fun toJson(date: Date): String {
        return date.time.toString()
    }
    
    @FromJson
    fun fromJson(dateString: String): Date {
        return Date(dateString.toLong())
    }
}

val moshi = Moshi.Builder()
    .add(DateAdapter())
    .add(KotlinJsonAdapterFactory())
    .build()
```

## 5. 复杂 JSON 处理

### 5.1 嵌套对象
```kotlin
@Serializable
data class Address(
    val street: String,
    val city: String,
    val country: String
)

@Serializable
data class User(
    val id: Int,
    val name: String,
    val address: Address
)

// JSON 示例
val json = """
{
    "id": 1,
    "name": "张三",
    "address": {
        "street": "人民路",
        "city": "北京",
        "country": "中国"
    }
}
"""
```

### 5.2 数组处理
```kotlin
@Serializable
data class User(
    val id: Int,
    val name: String,
    val tags: List<String>
)

// JSON 示例
val json = """
{
    "id": 1,
    "name": "张三",
    "tags": ["学生", "开发者", "Android"]
}
"""
```

### 5.3 可选字段
```kotlin
@Serializable
data class User(
    val id: Int,
    val name: String,
    val email: String? = null,
    val phone: String? = null
)

// JSON 示例
val json = """
{
    "id": 1,
    "name": "张三"
}
"""
```

## 6. 性能优化

### 6.1 缓存适配器
```kotlin
class CachedAdapter<T>(
    private val delegate: JsonAdapter<T>
) : JsonAdapter<T>() {
    private val cache = LruCache<String, T>(100)
    
    override fun fromJson(reader: JsonReader): T? {
        val json = reader.nextString()
        return cache.get(json) ?: delegate.fromJson(json)?.also {
            cache.put(json, it)
        }
    }
    
    override fun toJson(writer: JsonWriter, value: T?) {
        delegate.toJson(writer, value)
    }
}
```

### 6.2 流式处理
```kotlin
class JsonStreamProcessor {
    fun processLargeJson(inputStream: InputStream) {
        val reader = JsonReader(InputStreamReader(inputStream))
        reader.beginArray()
        while (reader.hasNext()) {
            val user = userAdapter.fromJson(reader)
            // 处理每个用户对象
        }
        reader.endArray()
    }
}
```

## 7. 实践练习

### 7.1 JSON 工具类
```kotlin
object JsonUtils {
    private val json = Json {
        prettyPrint = true
        isLenient = true
        ignoreUnknownKeys = true
    }
    
    fun <T> toJson(obj: T): String {
        return json.encodeToString(obj)
    }
    
    fun <T> fromJson(jsonString: String, type: KClass<T>): T {
        return json.decodeFromString(jsonString)
    }
    
    fun <T> fromJsonList(jsonString: String, type: KClass<T>): List<T> {
        return json.decodeFromString<List<T>>(jsonString)
    }
}
```

## 8. 最佳实践

### 8.1 性能考虑
- 使用适当的解析库
- 实现缓存机制
- 避免频繁序列化
- 使用流式处理

### 8.2 代码组织
- 统一序列化配置
- 使用工具类封装
- 处理异常情况
- 添加类型安全

## 9. 注意事项

### 9.1 数据安全
- 验证 JSON 数据
- 处理特殊字符
- 防止注入攻击
- 加密敏感数据

### 9.2 错误处理
- 处理解析异常
- 验证数据完整性
- 提供默认值
- 记录错误日志

## 10. 下一步
完成 JSON 解析学习后，您可以：
1. 学习 [协程基础](./docs/19_coroutines_basics.md)
2. 了解 [LiveData 和 ViewModel](./docs/20_livedata_viewmodel.md)
3. 开始实践项目开发

## 11. 参考资料
- [Gson 官方文档](https://github.com/google/gson)
- [Kotlinx Serialization 文档](https://kotlinlang.org/docs/serialization.html)
- [Moshi 官方文档](https://github.com/square/moshi) 