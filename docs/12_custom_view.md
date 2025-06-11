# 自定义 View

## 1. 自定义 View 概述

### 1.1 什么是自定义 View？
自定义 View 是 Android 中用于：
- 创建独特的 UI 组件
- 封装复杂的视图逻辑
- 提高代码复用性
- 实现特定的交互效果

### 1.2 自定义 View 类型
- **继承现有视图**：扩展现有视图的功能
- **组合视图**：组合多个视图创建新组件
- **完全自定义**：从零开始创建视图

## 2. 基本步骤

### 2.1 创建自定义 View 类
```kotlin
class CustomView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    // 初始化代码
    init {
        // 设置默认属性
        setBackgroundColor(Color.WHITE)
    }

    // 测量视图大小
    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        val width = getDefaultSize(suggestedMinimumWidth, widthMeasureSpec)
        val height = getDefaultSize(suggestedMinimumHeight, heightMeasureSpec)
        setMeasuredDimension(width, height)
    }

    // 绘制视图
    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        // 自定义绘制代码
    }
}
```

### 2.2 定义自定义属性
```xml
<!-- res/values/attrs.xml -->
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="CustomView">
        <attr name="customColor" format="color"/>
        <attr name="customText" format="string"/>
        <attr name="customSize" format="dimension"/>
    </declare-styleable>
</resources>
```

### 2.3 使用自定义属性
```kotlin
class CustomView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    private var customColor: Int = Color.BLACK
    private var customText: String = ""
    private var customSize: Float = 0f

    init {
        val typedArray = context.obtainStyledAttributes(
            attrs,
            R.styleable.CustomView,
            defStyleAttr,
            0
        )

        customColor = typedArray.getColor(
            R.styleable.CustomView_customColor,
            Color.BLACK
        )
        customText = typedArray.getString(
            R.styleable.CustomView_customText
        ) ?: ""
        customSize = typedArray.getDimension(
            R.styleable.CustomView_customSize,
            0f
        )

        typedArray.recycle()
    }
}
```

## 3. 绘制基础

### 3.1 基本图形绘制
```kotlin
override fun onDraw(canvas: Canvas) {
    super.onDraw(canvas)
    
    // 绘制矩形
    val paint = Paint().apply {
        color = Color.BLUE
        style = Paint.Style.FILL
    }
    canvas.drawRect(0f, 0f, width.toFloat(), height.toFloat(), paint)
    
    // 绘制圆形
    paint.color = Color.RED
    canvas.drawCircle(width / 2f, height / 2f, 50f, paint)
    
    // 绘制文本
    paint.color = Color.WHITE
    paint.textSize = 40f
    canvas.drawText("Hello", width / 2f, height / 2f, paint)
}
```

### 3.2 路径绘制
```kotlin
override fun onDraw(canvas: Canvas) {
    super.onDraw(canvas)
    
    val path = Path().apply {
        moveTo(0f, 0f)
        lineTo(width.toFloat(), 0f)
        lineTo(width / 2f, height.toFloat())
        close()
    }
    
    val paint = Paint().apply {
        color = Color.GREEN
        style = Paint.Style.FILL
    }
    
    canvas.drawPath(path, paint)
}
```

## 4. 触摸事件处理

### 4.1 基本触摸事件
```kotlin
class CustomView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    private var lastX: Float = 0f
    private var lastY: Float = 0f

    override fun onTouchEvent(event: MotionEvent): Boolean {
        when (event.action) {
            MotionEvent.ACTION_DOWN -> {
                lastX = event.x
                lastY = event.y
                return true
            }
            MotionEvent.ACTION_MOVE -> {
                val dx = event.x - lastX
                val dy = event.y - lastY
                translationX += dx
                translationY += dy
                lastX = event.x
                lastY = event.y
                return true
            }
        }
        return super.onTouchEvent(event)
    }
}
```

### 4.2 手势检测
```kotlin
class CustomView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    private val gestureDetector = GestureDetector(context,
        object : GestureDetector.SimpleOnGestureListener() {
            override fun onDown(e: MotionEvent): Boolean {
                return true
            }

            override fun onFling(
                e1: MotionEvent?,
                e2: MotionEvent,
                velocityX: Float,
                velocityY: Float
            ): Boolean {
                // 处理滑动事件
                return true
            }
        })

    override fun onTouchEvent(event: MotionEvent): Boolean {
        return gestureDetector.onTouchEvent(event)
    }
}
```

## 5. 动画效果

### 5.1 属性动画
```kotlin
class CustomView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    private var progress: Float = 0f
        set(value) {
            field = value
            invalidate()
        }

    fun startAnimation() {
        val animator = ObjectAnimator.ofFloat(this, "progress", 0f, 1f).apply {
            duration = 1000
            interpolator = AccelerateDecelerateInterpolator()
            start()
        }
    }
}
```

### 5.2 自定义动画
```kotlin
class CustomView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    private val paint = Paint()
    private var angle: Float = 0f

    fun startRotation() {
        val animator = ValueAnimator.ofFloat(0f, 360f).apply {
            duration = 1000
            repeatCount = ValueAnimator.INFINITE
            addUpdateListener { animation ->
                angle = animation.animatedValue as Float
                invalidate()
            }
            start()
        }
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        canvas.rotate(angle, width / 2f, height / 2f)
        // 绘制内容
    }
}
```

## 6. 实践练习

### 6.1 自定义进度条
```kotlin
class CustomProgressBar @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    private var progress: Float = 0f
    private val paint = Paint().apply {
        color = Color.BLUE
        style = Paint.Style.FILL
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        val progressWidth = width * progress
        canvas.drawRect(0f, 0f, progressWidth, height.toFloat(), paint)
    }

    fun setProgress(value: Float) {
        progress = value.coerceIn(0f, 1f)
        invalidate()
    }
}
```

### 6.2 自定义开关
```kotlin
class CustomSwitch @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    private var isChecked: Boolean = false
    private var thumbX: Float = 0f
    private val paint = Paint()

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        // 绘制背景
        paint.color = if (isChecked) Color.GREEN else Color.GRAY
        canvas.drawRoundRect(
            0f, 0f, width.toFloat(), height.toFloat(),
            height / 2f, height / 2f, paint
        )
        // 绘制滑块
        paint.color = Color.WHITE
        canvas.drawCircle(
            thumbX, height / 2f, height / 2f, paint
        )
    }

    override fun onTouchEvent(event: MotionEvent): Boolean {
        when (event.action) {
            MotionEvent.ACTION_DOWN -> return true
            MotionEvent.ACTION_UP -> {
                isChecked = !isChecked
                animateThumb()
                return true
            }
        }
        return super.onTouchEvent(event)
    }

    private fun animateThumb() {
        val targetX = if (isChecked) width - height else 0f
        ObjectAnimator.ofFloat(this, "thumbX", thumbX, targetX).apply {
            duration = 200
            start()
        }
    }
}
```

## 7. 最佳实践

### 7.1 性能优化
- 避免在 onDraw 中创建对象
- 使用 clipRect 限制绘制区域
- 合理使用硬件加速
- 优化触摸事件处理

### 7.2 代码组织
- 分离绘制逻辑
- 使用接口定义回调
- 提供自定义属性
- 添加文档注释

## 8. 下一步
完成自定义 View 学习后，您可以：
1. 学习 [SharedPreferences](./docs/13_shared_preferences.md)
2. 了解 [Room 数据库](./docs/14_room_database.md)
3. 开始实践项目开发

## 9. 参考资料
- [Android 自定义视图指南](https://developer.android.com/guide/topics/ui/custom-components)
- [Android 绘制指南](https://developer.android.com/training/custom-views/custom-drawing)
- [Android 开发者指南](https://developer.android.com/guide) 