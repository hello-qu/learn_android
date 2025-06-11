# SharedPreferences 使用指南

## 1. SharedPreferences 概述

### 1.1 什么是 SharedPreferences？
SharedPreferences 是 Android 中用于：
- 存储简单的键值对数据
- 保存用户设置和偏好
- 存储小型数据集合
- 提供数据持久化

### 1.2 主要特点
- 轻量级存储方案
- 支持多种数据类型
- 数据持久化
- 线程安全

## 2. 基本使用

### 2.1 获取 SharedPreferences 实例
```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var sharedPreferences: SharedPreferences

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 获取默认的 SharedPreferences
        sharedPreferences = getSharedPreferences(
            "my_preferences",
            Context.MODE_PRIVATE
        )
    }
}
```

### 2.2 存储数据
```kotlin
// 存储字符串
sharedPreferences.edit().apply {
    putString("username", "John")
    putInt("age", 25)
    putBoolean("is_logged_in", true)
    putFloat("score", 95.5f)
    putLong("timestamp", System.currentTimeMillis())
    apply()
}

// 存储字符串集合
sharedPreferences.edit().apply {
    putStringSet("tags", setOf("android", "kotlin", "development"))
    apply()
}
```

### 2.3 读取数据
```kotlin
// 读取各种类型的数据
val username = sharedPreferences.getString("username", "")
val age = sharedPreferences.getInt("age", 0)
val isLoggedIn = sharedPreferences.getBoolean("is_logged_in", false)
val score = sharedPreferences.getFloat("score", 0f)
val timestamp = sharedPreferences.getLong("timestamp", 0L)
val tags = sharedPreferences.getStringSet("tags", emptySet())
```

## 3. 高级用法

### 3.1 使用扩展函数
```kotlin
// 定义扩展函数
fun SharedPreferences.edit(operation: (SharedPreferences.Editor) -> Unit) {
    val editor = this.edit()
    operation(editor)
    editor.apply()
}

// 使用扩展函数
sharedPreferences.edit {
    putString("key", "value")
    putInt("number", 42)
}
```

### 3.2 数据监听
```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var sharedPreferences: SharedPreferences
    private val listener = SharedPreferences.OnSharedPreferenceChangeListener { _, key ->
        when (key) {
            "username" -> {
                val newUsername = sharedPreferences.getString(key, "")
                updateUI(newUsername)
            }
            "is_logged_in" -> {
                val isLoggedIn = sharedPreferences.getBoolean(key, false)
                handleLoginState(isLoggedIn)
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        sharedPreferences = getSharedPreferences("my_preferences", Context.MODE_PRIVATE)
        
        // 注册监听器
        sharedPreferences.registerOnSharedPreferenceChangeListener(listener)
    }

    override fun onDestroy() {
        super.onDestroy()
        // 注销监听器
        sharedPreferences.unregisterOnSharedPreferenceChangeListener(listener)
    }
}
```

## 4. 数据迁移

### 4.1 版本升级
```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var sharedPreferences: SharedPreferences

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        sharedPreferences = getSharedPreferences("my_preferences", Context.MODE_PRIVATE)
        
        // 检查版本并迁移数据
        migrateDataIfNeeded()
    }

    private fun migrateDataIfNeeded() {
        val currentVersion = sharedPreferences.getInt("version", 1)
        if (currentVersion < 2) {
            // 执行数据迁移
            sharedPreferences.edit {
                // 迁移数据
                putString("new_key", getString("old_key", ""))
                remove("old_key")
                // 更新版本号
                putInt("version", 2)
            }
        }
    }
}
```

## 5. 最佳实践

### 5.1 数据封装
```kotlin
class UserPreferences(context: Context) {
    private val sharedPreferences = context.getSharedPreferences(
        "user_preferences",
        Context.MODE_PRIVATE
    )

    var username: String
        get() = sharedPreferences.getString("username", "") ?: ""
        set(value) = sharedPreferences.edit {
            putString("username", value)
        }

    var isLoggedIn: Boolean
        get() = sharedPreferences.getBoolean("is_logged_in", false)
        set(value) = sharedPreferences.edit {
            putBoolean("is_logged_in", value)
        }

    fun clear() {
        sharedPreferences.edit {
            clear()
        }
    }
}
```

### 5.2 使用委托属性
```kotlin
class PreferencesDelegate<T>(
    private val sharedPreferences: SharedPreferences,
    private val key: String,
    private val defaultValue: T
) {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return when (defaultValue) {
            is String -> sharedPreferences.getString(key, defaultValue) as T
            is Int -> sharedPreferences.getInt(key, defaultValue) as T
            is Boolean -> sharedPreferences.getBoolean(key, defaultValue) as T
            is Float -> sharedPreferences.getFloat(key, defaultValue) as T
            is Long -> sharedPreferences.getLong(key, defaultValue) as T
            else -> throw IllegalArgumentException("Unsupported type")
        }
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        sharedPreferences.edit {
            when (value) {
                is String -> putString(key, value)
                is Int -> putInt(key, value)
                is Boolean -> putBoolean(key, value)
                is Float -> putFloat(key, value)
                is Long -> putLong(key, value)
                else -> throw IllegalArgumentException("Unsupported type")
            }
        }
    }
}
```

## 6. 实践练习

### 6.1 用户设置管理
```kotlin
class SettingsManager(context: Context) {
    private val sharedPreferences = context.getSharedPreferences(
        "settings",
        Context.MODE_PRIVATE
    )

    // 主题设置
    var isDarkTheme: Boolean by PreferencesDelegate(
        sharedPreferences,
        "is_dark_theme",
        false
    )

    // 通知设置
    var isNotificationEnabled: Boolean by PreferencesDelegate(
        sharedPreferences,
        "is_notification_enabled",
        true
    )

    // 字体大小
    var fontSize: Int by PreferencesDelegate(
        sharedPreferences,
        "font_size",
        16
    )

    // 语言设置
    var language: String by PreferencesDelegate(
        sharedPreferences,
        "language",
        "zh"
    )
}
```

## 7. 注意事项

### 7.1 性能考虑
- 避免存储大量数据
- 使用 apply() 而不是 commit()
- 合理使用监听器
- 及时清理不需要的数据

### 7.2 安全性
- 不要存储敏感信息
- 使用 MODE_PRIVATE 模式
- 考虑数据加密
- 定期清理过期数据

## 8. 下一步
完成 SharedPreferences 学习后，您可以：
1. 学习 [Room 数据库](./docs/14_room_database.md)
2. 了解 [文件存储](./docs/15_file_storage.md)
3. 开始实践项目开发

## 9. 参考资料
- [Android SharedPreferences 文档](https://developer.android.com/training/data-storage/shared-preferences)
- [Android 数据存储指南](https://developer.android.com/guide/topics/data)
- [Android 开发者指南](https://developer.android.com/guide) 