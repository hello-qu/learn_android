# 常用 UI 组件

## 1. 文本组件

### 1.1 TextView
```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="普通文本"
    android:textSize="16sp"
    android:textColor="@color/black"
    android:textStyle="bold"
    android:gravity="center"
    android:maxLines="2"
    android:ellipsize="end"/>
```

### 1.2 EditText
```xml
<EditText
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="请输入内容"
    android:inputType="text"
    android:maxLines="1"
    android:imeOptions="actionDone"
    android:background="@drawable/edit_text_background"/>
```

### 1.3 AutoCompleteTextView
```xml
<AutoCompleteTextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="自动完成输入"
    android:completionThreshold="1"/>
```

## 2. 按钮组件

### 2.1 Button
```xml
<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="按钮"
    android:textColor="@color/white"
    android:background="@drawable/button_background"
    android:padding="12dp"/>
```

### 2.2 ImageButton
```xml
<ImageButton
    android:layout_width="48dp"
    android:layout_height="48dp"
    android:src="@drawable/ic_add"
    android:background="?attr/selectableItemBackgroundBorderless"
    android:contentDescription="添加按钮"/>
```

### 2.3 MaterialButton
```xml
<com.google.android.material.button.MaterialButton
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Material 按钮"
    app:icon="@drawable/ic_add"
    app:cornerRadius="8dp"/>
```

## 3. 图片组件

### 3.1 ImageView
```xml
<ImageView
    android:layout_width="200dp"
    android:layout_height="200dp"
    android:src="@drawable/image"
    android:scaleType="centerCrop"
    android:contentDescription="图片描述"/>
```

### 3.2 ShapeableImageView
```xml
<com.google.android.material.imageview.ShapeableImageView
    android:layout_width="200dp"
    android:layout_height="200dp"
    android:src="@drawable/image"
    app:shapeAppearanceOverlay="@style/CircleImageView"/>
```

## 4. 列表组件

### 4.1 RecyclerView
```xml
<androidx.recyclerview.widget.RecyclerView
    android:id="@+id/recyclerView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:clipToPadding="false"
    android:padding="8dp"/>
```

### 4.2 ListView
```xml
<ListView
    android:id="@+id/listView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:divider="@drawable/list_divider"
    android:dividerHeight="1dp"/>
```

## 5. 进度组件

### 5.1 ProgressBar
```xml
<ProgressBar
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:indeterminate="true"/>
```

### 5.2 ProgressBar（确定进度）
```xml
<ProgressBar
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    style="?android:attr/progressBarStyleHorizontal"
    android:progress="50"
    android:max="100"/>
```

### 5.3 CircularProgressIndicator
```xml
<com.google.android.material.progressindicator.CircularProgressIndicator
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:indeterminate="true"/>
```

## 6. 选择组件

### 6.1 CheckBox
```xml
<CheckBox
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="复选框"
    android:checked="false"/>
```

### 6.2 RadioButton
```xml
<RadioGroup
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <RadioButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="选项 1"/>

    <RadioButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="选项 2"/>

</RadioGroup>
```

### 6.3 Switch
```xml
<Switch
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="开关"
    android:checked="false"/>
```

## 7. 对话框组件

### 7.1 AlertDialog
```kotlin
AlertDialog.Builder(context)
    .setTitle("标题")
    .setMessage("消息内容")
    .setPositiveButton("确定") { dialog, _ ->
        // 处理确定按钮点击
        dialog.dismiss()
    }
    .setNegativeButton("取消") { dialog, _ ->
        // 处理取消按钮点击
        dialog.dismiss()
    }
    .show()
```

### 7.2 BottomSheetDialog
```kotlin
val bottomSheetDialog = BottomSheetDialog(context)
val view = layoutInflater.inflate(R.layout.bottom_sheet_layout, null)
bottomSheetDialog.setContentView(view)
bottomSheetDialog.show()
```

## 8. 输入组件

### 8.1 TextInputLayout
```xml
<com.google.android.material.textfield.TextInputLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="输入框"
    style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox">

    <com.google.android.material.textfield.TextInputEditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</com.google.android.material.textfield.TextInputLayout>
```

### 8.2 Spinner
```xml
<Spinner
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:entries="@array/spinner_items"/>
```

## 9. 卡片组件

### 9.1 CardView
```xml
<androidx.cardview.widget.CardView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="卡片标题"
            android:textSize="18sp"
            android:textStyle="bold"/>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="卡片内容"
            android:layout_marginTop="8dp"/>

    </LinearLayout>

</androidx.cardview.widget.CardView>
```

## 10. 实践练习

### 10.1 登录表单
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <com.google.android.material.textfield.TextInputLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="用户名"
        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox">

        <com.google.android.material.textfield.TextInputEditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="text"/>

    </com.google.android.material.textfield.TextInputLayout>

    <com.google.android.material.textfield.TextInputLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="密码"
        android:layout_marginTop="16dp"
        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox">

        <com.google.android.material.textfield.TextInputEditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textPassword"/>

    </com.google.android.material.textfield.TextInputLayout>

    <com.google.android.material.button.MaterialButton
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="登录"
        android:layout_marginTop="24dp"/>

</LinearLayout>
```

## 11. 下一步
完成常用 UI 组件学习后，您可以：
1. 学习 [ConstraintLayout 详解](./10_constraint_layout.md)
2. 了解 [Material Design 组件](./11_material_design.md)
3. 开始实践项目开发

## 12. 参考资料
- [Android UI 组件文档](https://developer.android.com/guide/topics/ui/controls)
- [Material Design 组件](https://material.io/components)
- [Android 开发者指南](https://developer.android.com/guide)