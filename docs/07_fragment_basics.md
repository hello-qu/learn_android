# Fragment 基础

## 1. Fragment 概述

### 1.1 什么是 Fragment？
Fragment 是 Activity 中的模块化部分，具有自己的生命周期和用户界面。Fragment 可以：
- 组合成完整的用户界面
- 在不同 Activity 中重用
- 动态添加、移除、替换
- 响应配置更改

### 1.2 Fragment 的优势
- 模块化设计
- 灵活的 UI 组合
- 更好的适配不同屏幕
- 更流畅的用户体验

## 2. 创建 Fragment

### 2.1 基本 Fragment
```kotlin
class MyFragment : Fragment() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // 初始化 Fragment
    }

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // 创建 Fragment 的视图
        return inflater.inflate(R.layout.fragment_my, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        // 初始化视图
    }
}
```

### 2.2 Fragment 布局
```xml
<!-- fragment_my.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello Fragment" />

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Click Me" />

</LinearLayout>
```

## 3. Fragment 生命周期

### 3.1 生命周期方法
```kotlin
class MyFragment : Fragment() {
    override fun onAttach(context: Context) {
        super.onAttach(context)
        // Fragment 附加到 Activity
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Fragment 创建
    }

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // 创建 Fragment 视图
        return inflater.inflate(R.layout.fragment_my, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        // 视图创建完成
    }

    override fun onStart() {
        super.onStart()
        // Fragment 变为可见
    }

    override fun onResume() {
        super.onResume()
        // Fragment 可交互
    }

    override fun onPause() {
        super.onPause()
        // Fragment 暂停
    }

    override fun onStop() {
        super.onStop()
        // Fragment 停止
    }

    override fun onDestroyView() {
        super.onDestroyView()
        // 视图被销毁
    }

    override fun onDestroy() {
        super.onDestroy()
        // Fragment 被销毁
    }

    override fun onDetach() {
        super.onDetach()
        // Fragment 从 Activity 分离
    }
}
```

## 4. Fragment 管理

### 4.1 在 Activity 中添加 Fragment
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 使用 FragmentManager 添加 Fragment
        if (savedInstanceState == null) {
            supportFragmentManager.beginTransaction()
                .add(R.id.fragmentContainer, MyFragment())
                .commit()
        }
    }
}
```

### 4.2 Fragment 事务
```kotlin
// 替换 Fragment
supportFragmentManager.beginTransaction()
    .replace(R.id.fragmentContainer, NewFragment())
    .addToBackStack(null)  // 添加到返回栈
    .commit()

// 添加 Fragment
supportFragmentManager.beginTransaction()
    .add(R.id.fragmentContainer, NewFragment())
    .addToBackStack(null)
    .commit()

// 移除 Fragment
supportFragmentManager.beginTransaction()
    .remove(fragment)
    .commit()
```

## 5. Fragment 通信

### 5.1 使用接口
```kotlin
// 定义接口
interface OnFragmentInteractionListener {
    fun onFragmentInteraction(data: String)
}

// 在 Fragment 中使用
class MyFragment : Fragment() {
    private var listener: OnFragmentInteractionListener? = null

    override fun onAttach(context: Context) {
        super.onAttach(context)
        if (context is OnFragmentInteractionListener) {
            listener = context
        }
    }

    private fun sendDataToActivity(data: String) {
        listener?.onFragmentInteraction(data)
    }
}

// 在 Activity 中实现
class MainActivity : AppCompatActivity(), OnFragmentInteractionListener {
    override fun onFragmentInteraction(data: String) {
        // 处理来自 Fragment 的数据
    }
}
```

### 5.2 使用 ViewModel
```kotlin
class SharedViewModel : ViewModel() {
    private val _data = MutableLiveData<String>()
    val data: LiveData<String> = _data

    fun updateData(newData: String) {
        _data.value = newData
    }
}

// 在 Fragment 中使用
class MyFragment : Fragment() {
    private val viewModel: SharedViewModel by activityViewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        viewModel.data.observe(viewLifecycleOwner) { data ->
            // 处理数据更新
        }
    }
}
```

## 6. Fragment 状态保存

### 6.1 保存状态
```kotlin
class MyFragment : Fragment() {
    private var counter = 0

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putInt("counter", counter)
    }

    override fun onViewStateRestored(savedInstanceState: Bundle?) {
        super.onViewStateRestored(savedInstanceState)
        counter = savedInstanceState?.getInt("counter") ?: 0
    }
}
```

## 7. 最佳实践

### 7.1 使用 Fragment Factory
```kotlin
class MyFragmentFactory : FragmentFactory() {
    override fun instantiate(classLoader: ClassLoader, className: String): Fragment {
        return when (className) {
            MyFragment::class.java.name -> MyFragment()
            else -> super.instantiate(classLoader, className)
        }
    }
}

// 在 Activity 中使用
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        supportFragmentManager.fragmentFactory = MyFragmentFactory()
        super.onCreate(savedInstanceState)
    }
}
```

### 7.2 使用 Navigation 组件
```kotlin
// 在 navigation graph 中定义
<navigation>
    <fragment
        android:id="@+id/myFragment"
        android:name="com.example.MyFragment"
        android:label="My Fragment">
        <action
            android:id="@+id/action_to_second"
            app:destination="@id/secondFragment" />
    </fragment>
</navigation>

// 在 Fragment 中使用
findNavController().navigate(R.id.action_to_second)
```

## 8. 实践练习

### 8.1 简单的列表-详情界面
```kotlin
// 列表 Fragment
class ListFragment : Fragment() {
    private val viewModel: ListViewModel by viewModels()
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        // 设置列表点击事件
        listView.setOnItemClickListener { _, _, position, _ ->
            val item = viewModel.items[position]
            // 导航到详情页面
            findNavController().navigate(
                ListFragmentDirections.actionListToDetail(item.id)
            )
        }
    }
}

// 详情 Fragment
class DetailFragment : Fragment() {
    private val args: DetailFragmentArgs by navArgs()
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        // 加载详情数据
        val itemId = args.itemId
        // 显示详情
    }
}
```

## 9. 下一步
完成 Fragment 学习后，您可以：
1. 学习 [XML 布局基础](./08_xml_layout_basics.md)
2. 了解 [常用 UI 组件](./09_common_ui_components.md)
3. 开始实践项目开发

## 10. 参考资料
- [Android Fragment 指南](https://developer.android.com/guide/fragments)
- [Android Fragment 生命周期](https://developer.android.com/guide/fragments/lifecycle)
- [Android Navigation 组件](https://developer.android.com/guide/navigation) 