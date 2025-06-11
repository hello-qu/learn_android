# Kotlin 基础语法

## 1. Kotlin 简介
Kotlin 是一种现代编程语言，由 JetBrains 开发，被 Google 推荐为 Android 开发的首选语言。它具有以下特点：
- 完全兼容 Java
- 简洁的语法
- 空安全
- 函数式编程支持
- 协程支持

## 2. 基本语法

### 2.1 变量声明
```kotlin
// 可变变量
var name: String = "张三"
name = "李四"  // 可以修改

// 不可变变量（常量）
val age: Int = 25
// age = 26  // 错误：不能修改 val 变量

// 类型推断
var score = 95  // 自动推断为 Int 类型
```

### 2.2 基本数据类型
```kotlin
// 数字类型
val intNumber: Int = 100
val longNumber: Long = 100L
val floatNumber: Float = 100.0f
val doubleNumber: Double = 100.0

// 布尔类型
val isTrue: Boolean = true

// 字符类型
val char: Char = 'A'

// 字符串
val string: String = "Hello"
// 字符串模板
val message = "姓名：$name，年龄：$age"
```

### 2.3 空安全
```kotlin
// 可空类型
var nullableName: String? = null

// 安全调用
val length = nullableName?.length

// Elvis 操作符
val nameLength = nullableName?.length ?: 0

// 非空断言
val forceLength = nullableName!!.length  // 不推荐使用
```

## 3. 控制流

### 3.1 条件语句
```kotlin
// if 表达式
val max = if (a > b) a else b

// when 表达式（类似 switch）
val grade = when (score) {
    in 90..100 -> "A"
    in 80..89 -> "B"
    in 70..79 -> "C"
    else -> "D"
}
```

### 3.2 循环
```kotlin
// for 循环
for (i in 1..5) {
    println(i)
}

// while 循环
var i = 0
while (i < 5) {
    println(i)
    i++
}
```

## 4. 函数

### 4.1 基本函数
```kotlin
// 基本函数定义
fun add(a: Int, b: Int): Int {
    return a + b
}

// 单表达式函数
fun multiply(a: Int, b: Int) = a * b

// 默认参数
fun greet(name: String = "World") = "Hello, $name!"
```

### 4.2 扩展函数
```kotlin
// 为 String 类添加扩展函数
fun String.addExclamation() = "$this!"

// 使用扩展函数
val greeting = "Hello".addExclamation()  // 结果：Hello!
```

## 5. 类和对象

### 5.1 基本类
```kotlin
// 数据类
data class User(
    val name: String,
    val age: Int
)

// 普通类
class Person {
    var name: String = ""
    var age: Int = 0
    
    fun introduce() = "我叫 $name，今年 $age 岁"
}
```

### 5.2 构造函数
```kotlin
class Student(
    val name: String,
    val age: Int,
    var score: Int = 0
) {
    init {
        println("创建学生：$name")
    }
}
```

## 6. 集合

### 6.1 列表
```kotlin
// 不可变列表
val numbers = listOf(1, 2, 3, 4, 5)

// 可变列表
val mutableNumbers = mutableListOf(1, 2, 3)
mutableNumbers.add(4)
```

### 6.2 映射
```kotlin
// 不可变映射
val map = mapOf(
    "name" to "张三",
    "age" to 25
)

// 可变映射
val mutableMap = mutableMapOf(
    "name" to "李四",
    "age" to 30
)
mutableMap["score"] = 95
```

## 7. 协程基础
```kotlin
// 启动协程
GlobalScope.launch {
    delay(1000L)
    println("协程执行完成")
}

// 异步函数
suspend fun fetchData(): String {
    delay(1000L)
    return "数据"
}
```

## 8. 实践练习

### 练习 1：创建简单的计算器
```kotlin
fun calculate(a: Int, b: Int, operation: String): Int {
    return when (operation) {
        "+" -> a + b
        "-" -> a - b
        "*" -> a * b
        "/" -> if (b != 0) a / b else throw IllegalArgumentException("除数不能为0")
        else -> throw IllegalArgumentException("不支持的运算")
    }
}
```

### 练习 2：学生管理系统
```kotlin
data class Student(
    val id: Int,
    val name: String,
    var score: Int
)

class StudentManager {
    private val students = mutableListOf<Student>()
    
    fun addStudent(student: Student) {
        students.add(student)
    }
    
    fun findStudent(id: Int): Student? {
        return students.find { it.id == id }
    }
    
    fun getAverageScore(): Double {
        return students.map { it.score }.average()
    }
}
```

## 9. 下一步
完成 Kotlin 基础学习后，您可以：
1. 继续学习 [Android Studio 使用指南](./03_android_studio_guide.md)
2. 开始了解 [Android 应用架构](./04_android_architecture.md)
3. 尝试编写更多 Kotlin 练习程序

## 10. 参考资料
- [Kotlin 官方文档](https://kotlinlang.org/docs/home.html)
- [Kotlin 协程指南](https://kotlinlang.org/docs/coroutines-guide.html)
- [Kotlin 编码规范](https://kotlinlang.org/docs/coding-conventions.html) 