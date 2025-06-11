# Intent 和 Intent Filter

## 1. Intent 概述

### 1.1 什么是 Intent？
Intent 是 Android 系统中用于组件间通信的消息传递对象，可以用于：
- 启动 Activity
- 启动 Service
- 发送广播
- 传递数据

### 1.2 Intent 类型
- **显式 Intent**：明确指定目标组件
- **隐式 Intent**：指定 action 和 data，由系统匹配合适的组件

## 2. 显式 Intent

### 2.1 基本使用
```kotlin
// 启动新的 Activity
val intent = Intent(this, SecondActivity::class.java)
startActivity(intent)

// 带数据的 Intent
val intent = Intent(this, SecondActivity::class.java).apply {
    putExtra("name", "张三")
    putExtra("age", 25)
}
startActivity(intent)
```

### 2.2 接收数据
```kotlin
class SecondActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_second)
        
        // 获取传递的数据
        val name = intent.getStringExtra("name")
        val age = intent.getIntExtra("age", 0)
    }
}
```

## 3. 隐式 Intent

### 3.1 基本使用
```kotlin
// 打开网页
val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://www.example.com"))
startActivity(intent)

// 发送邮件
val intent = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_EMAIL, arrayOf("recipient@example.com"))
    putExtra(Intent.EXTRA_SUBJECT, "邮件主题")
    putExtra(Intent.EXTRA_TEXT, "邮件内容")
}
startActivity(intent)
```

### 3.2 检查是否有应用可以处理
```kotlin
val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://www.example.com"))
if (intent.resolveActivity(packageManager) != null) {
    startActivity(intent)
} else {
    // 没有应用可以处理此 Intent
    Toast.makeText(this, "没有找到可以处理此请求的应用", Toast.LENGTH_SHORT).show()
}
```

## 4. Intent Filter

### 4.1 在 AndroidManifest.xml 中声明
```xml
<activity android:name=".ShareActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
</activity>
```

### 4.2 处理 Intent
```kotlin
class ShareActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_share)
        
        when (intent.action) {
            Intent.ACTION_SEND -> {
                if (intent.type == "text/plain") {
                    val sharedText = intent.getStringExtra(Intent.EXTRA_TEXT)
                    // 处理共享的文本
                }
            }
        }
    }
}
```

## 5. 启动模式

### 5.1 在 AndroidManifest.xml 中设置
```xml
<activity
    android:name=".MainActivity"
    android:launchMode="singleTop">
</activity>
```

### 5.2 启动模式类型
- **standard**：默认模式，每次启动创建新实例
- **singleTop**：如果目标 Activity 已在栈顶，则复用
- **singleTask**：如果目标 Activity 已在栈中，则清除其上的所有 Activity
- **singleInstance**：创建新的任务栈，且该栈中只有这一个 Activity

## 6. 数据传递

### 6.1 基本数据类型
```kotlin
// 发送数据
val intent = Intent(this, SecondActivity::class.java).apply {
    putExtra("string", "Hello")
    putExtra("int", 100)
    putExtra("boolean", true)
    putExtra("float", 3.14f)
    putExtra("long", 1000L)
}

// 接收数据
val string = intent.getStringExtra("string")
val int = intent.getIntExtra("int", 0)
val boolean = intent.getBooleanExtra("boolean", false)
val float = intent.getFloatExtra("float", 0f)
val long = intent.getLongExtra("long", 0L)
```

### 6.2 复杂数据类型
```kotlin
// 传递 Parcelable 对象
data class User(
    val name: String,
    val age: Int
) : Parcelable {
    constructor(parcel: Parcel) : this(
        parcel.readString()!!,
        parcel.readInt()
    )

    override fun writeToParcel(parcel: Parcel, flags: Int) {
        parcel.writeString(name)
        parcel.writeInt(age)
    }

    override fun describeContents(): Int = 0

    companion object CREATOR : Parcelable.Creator<User> {
        override fun createFromParcel(parcel: Parcel): User = User(parcel)
        override fun newArray(size: Int): Array<User?> = arrayOfNulls(size)
    }
}

// 使用
val user = User("张三", 25)
intent.putExtra("user", user)
```

## 7. 返回结果

### 7.1 启动 Activity 并等待结果
```kotlin
class MainActivity : AppCompatActivity() {
    private val startForResult = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        if (result.resultCode == Activity.RESULT_OK) {
            val data = result.data
            // 处理返回的数据
        }
    }

    private fun startSecondActivity() {
        val intent = Intent(this, SecondActivity::class.java)
        startForResult.launch(intent)
    }
}
```

### 7.2 返回结果
```kotlin
class SecondActivity : AppCompatActivity() {
    private fun returnResult() {
        val intent = Intent().apply {
            putExtra("result", "处理完成")
        }
        setResult(Activity.RESULT_OK, intent)
        finish()
    }
}
```

## 8. 最佳实践

### 8.1 使用常量
```kotlin
companion object {
    const val EXTRA_NAME = "extra_name"
    const val EXTRA_AGE = "extra_age"
    const val REQUEST_CODE = 1001
}
```

### 8.2 类型安全
```kotlin
// 使用密封类定义 Intent 类型
sealed class MainIntent {
    data class OpenDetail(val id: Int) : MainIntent()
    data class ShareContent(val content: String) : MainIntent()
    object Refresh : MainIntent()
}

// 使用
fun handleIntent(intent: MainIntent) {
    when (intent) {
        is MainIntent.OpenDetail -> openDetail(intent.id)
        is MainIntent.ShareContent -> shareContent(intent.content)
        MainIntent.Refresh -> refresh()
    }
}
```

## 9. 实践练习

### 9.1 简单的分享应用
```kotlin
class ShareActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_share)
        
        // 处理分享意图
        when (intent.action) {
            Intent.ACTION_SEND -> {
                if (intent.type?.startsWith("text/") == true) {
                    val sharedText = intent.getStringExtra(Intent.EXTRA_TEXT)
                    // 显示共享的文本
                    textView.text = sharedText
                }
            }
        }
        
        // 分享按钮点击事件
        shareButton.setOnClickListener {
            val intent = Intent(Intent.ACTION_SEND).apply {
                type = "text/plain"
                putExtra(Intent.EXTRA_TEXT, "分享的内容")
            }
            startActivity(Intent.createChooser(intent, "分享到"))
        }
    }
}
```

## 10. 下一步
完成 Intent 学习后，您可以：
1. 学习 [Fragment 基础](./07_fragment_basics.md)
2. 了解 [XML 布局基础](./08_xml_layout_basics.md)
3. 开始实践项目开发

## 11. 参考资料
- [Android Intent 指南](https://developer.android.com/guide/components/intents-filters)
- [Android Activity 结果 API](https://developer.android.com/training/basics/intents/result)
- [Android 启动模式](https://developer.android.com/guide/components/activities/tasks-and-back-stack) 