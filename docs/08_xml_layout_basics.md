# XML 布局基础

## 1. 布局概述

### 1.1 什么是布局？
布局是 Android 应用中定义用户界面的 XML 文件，用于：
- 定义界面结构
- 排列 UI 组件
- 设置组件属性
- 响应不同屏幕尺寸

### 1.2 布局类型
- **LinearLayout**：线性布局
- **RelativeLayout**：相对布局
- **ConstraintLayout**：约束布局
- **FrameLayout**：帧布局
- **GridLayout**：网格布局
- **TableLayout**：表格布局

## 2. 基本布局属性

### 2.1 通用属性
```xml
<!-- 尺寸属性 -->
android:layout_width="match_parent"  <!-- 宽度填充父容器 -->
android:layout_height="wrap_content" <!-- 高度适应内容 -->

<!-- 边距属性 -->
android:layout_margin="16dp"         <!-- 外边距 -->
android:padding="8dp"                <!-- 内边距 -->

<!-- 背景属性 -->
android:background="@color/white"    <!-- 背景颜色 -->
android:background="@drawable/bg"    <!-- 背景图片 -->

<!-- 可见性 -->
android:visibility="visible"         <!-- 可见性：visible/invisible/gone -->
```

### 2.2 文本属性
```xml
<!-- 文本属性 -->
android:text="Hello World"           <!-- 文本内容 -->
android:textSize="16sp"              <!-- 文本大小 -->
android:textColor="@color/black"     <!-- 文本颜色 -->
android:textStyle="bold"             <!-- 文本样式 -->
android:gravity="center"             <!-- 文本对齐方式 -->
```

## 3. 常用布局

### 3.1 LinearLayout
```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="标题"
        android:textSize="20sp"
        android:layout_marginBottom="16dp"/>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="按钮"/>

</LinearLayout>
```

### 3.2 RelativeLayout
```xml
<RelativeLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="16dp"
        android:text="标题"/>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/title"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="16dp"
        android:text="按钮"/>

</RelativeLayout>
```

### 3.3 ConstraintLayout
```xml
<androidx.constraintlayout.widget.ConstraintLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="标题"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="16dp"/>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="按钮"
        app:layout_constraintTop_toBottomOf="@id/title"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="16dp"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```

## 4. 资源引用

### 4.1 颜色资源
```xml
<!-- colors.xml -->
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="primary">#FF6200EE</color>
    <color name="secondary">#FF03DAC5</color>
    <color name="black">#FF000000</color>
    <color name="white">#FFFFFFFF</color>
</resources>

<!-- 使用颜色资源 -->
android:background="@color/primary"
```

### 4.2 尺寸资源
```xml
<!-- dimens.xml -->
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <dimen name="margin_small">8dp</dimen>
    <dimen name="margin_medium">16dp</dimen>
    <dimen name="margin_large">24dp</dimen>
    <dimen name="text_size_small">12sp</dimen>
    <dimen name="text_size_medium">16sp</dimen>
    <dimen name="text_size_large">20sp</dimen>
</resources>

<!-- 使用尺寸资源 -->
android:layout_margin="@dimen/margin_medium"
android:textSize="@dimen/text_size_medium"
```

### 4.3 字符串资源
```xml
<!-- strings.xml -->
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">我的应用</string>
    <string name="welcome_message">欢迎使用</string>
    <string name="button_submit">提交</string>
</resources>

<!-- 使用字符串资源 -->
android:text="@string/welcome_message"
```

## 5. 样式和主题

### 5.1 样式定义
```xml
<!-- styles.xml -->
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="TextAppearance.App.Headline">
        <item name="android:textSize">20sp</item>
        <item name="android:textColor">@color/black</item>
        <item name="android:textStyle">bold</item>
    </style>

    <style name="Widget.App.Button">
        <item name="android:layout_width">wrap_content</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:padding">12dp</item>
        <item name="android:background">@drawable/button_background</item>
    </style>
</resources>

<!-- 使用样式 -->
android:textAppearance="@style/TextAppearance.App.Headline"
style="@style/Widget.App.Button"
```

### 5.2 主题定义
```xml
<!-- themes.xml -->
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="Theme.App" parent="Theme.MaterialComponents.DayNight">
        <item name="colorPrimary">@color/primary</item>
        <item name="colorSecondary">@color/secondary</item>
        <item name="android:windowBackground">@color/white</item>
    </style>
</resources>

<!-- 在 AndroidManifest.xml 中应用主题 -->
<application
    android:theme="@style/Theme.App">
```

## 6. 响应式布局

### 6.1 权重使用
```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">

    <TextView
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="左侧"/>

    <TextView
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="2"
        android:text="右侧"/>

</LinearLayout>
```

### 6.2 屏幕适配
```xml
<!-- 不同屏幕尺寸的布局 -->
res/layout/main.xml              <!-- 默认布局 -->
res/layout-large/main.xml        <!-- 大屏幕布局 -->
res/layout-xlarge/main.xml       <!-- 超大屏幕布局 -->
res/layout-land/main.xml         <!-- 横屏布局 -->
```

## 7. 最佳实践

### 7.1 布局优化
```xml
<!-- 使用 merge 标签减少视图层级 -->
<merge xmlns:android="http://schemas.android.com/apk/res/android">
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="内容"/>
</merge>

<!-- 使用 include 标签复用布局 -->
<include
    android:id="@+id/toolbar"
    layout="@layout/toolbar"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"/>
```

### 7.2 性能优化
- 避免过深的视图层级
- 使用 ConstraintLayout 减少嵌套
- 合理使用 ViewStub
- 避免过度使用权重

## 8. 实践练习

### 8.1 登录界面布局
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="@dimen/margin_medium">

    <EditText
        android:id="@+id/usernameInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="@string/hint_username"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="32dp"/>

    <EditText
        android:id="@+id/passwordInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="@string/hint_password"
        android:inputType="textPassword"
        app:layout_constraintTop_toBottomOf="@id/usernameInput"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="16dp"/>

    <Button
        android:id="@+id/loginButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/button_login"
        app:layout_constraintTop_toBottomOf="@id/passwordInput"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="24dp"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```

## 9. 下一步
完成 XML 布局学习后，您可以：
1. 学习 [常用 UI 组件](./09_common_ui_components.md)
2. 了解 [ConstraintLayout 详解](./10_constraint_layout.md)
3. 开始实践项目开发

## 10. 参考资料
- [Android 布局指南](https://developer.android.com/guide/topics/ui/declaring-layout)
- [Android 资源管理](https://developer.android.com/guide/topics/resources/providing-resources)
- [Android 样式和主题](https://developer.android.com/guide/topics/ui/look-and-feel/themes) 