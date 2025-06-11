# RecyclerView 使用指南

## 1. RecyclerView 概述

### 1.1 什么是 RecyclerView？
RecyclerView 是 Android 中用于显示列表数据的组件，它：
- 高效处理大量数据
- 支持列表项的回收和复用
- 提供灵活的布局管理
- 支持动画效果

### 1.2 主要组件
- **RecyclerView**: 列表容器
- **Adapter**: 数据适配器
- **ViewHolder**: 视图持有者
- **LayoutManager**: 布局管理器

## 2. 基本使用

### 2.1 布局文件
```xml
<!-- activity_main.xml -->
<androidx.recyclerview.widget.RecyclerView
    android:id="@+id/recyclerView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager" />

<!-- item_list.xml -->
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="16dp"
    android:orientation="vertical">
    
    <TextView
        android:id="@+id/titleText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="16sp"
        android:textStyle="bold" />
        
    <TextView
        android:id="@+id/descriptionText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp" />
</LinearLayout>
```

### 2.2 数据模型
```kotlin
data class Item(
    val id: Int,
    val title: String,
    val description: String
)
```

### 2.3 适配器实现
```kotlin
class ItemAdapter(
    private val items: List<Item>,
    private val onItemClick: (Item) -> Unit
) : RecyclerView.Adapter<ItemAdapter.ViewHolder>() {

    class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val titleText: TextView = view.findViewById(R.id.titleText)
        val descriptionText: TextView = view.findViewById(R.id.descriptionText)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_list, parent, false)
        return ViewHolder(view)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val item = items[position]
        holder.titleText.text = item.title
        holder.descriptionText.text = item.description
        
        holder.itemView.setOnClickListener {
            onItemClick(item)
        }
    }

    override fun getItemCount() = items.size
}
```

### 2.4 Activity 中使用
```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var recyclerView: RecyclerView
    private lateinit var adapter: ItemAdapter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        recyclerView = findViewById(R.id.recyclerView)
        
        // 准备数据
        val items = listOf(
            Item(1, "标题1", "描述1"),
            Item(2, "标题2", "描述2"),
            Item(3, "标题3", "描述3")
        )

        // 设置适配器
        adapter = ItemAdapter(items) { item ->
            // 处理点击事件
            Toast.makeText(this, "点击了: ${item.title}", Toast.LENGTH_SHORT).show()
        }
        
        recyclerView.adapter = adapter
    }
}
```

## 3. 布局管理器

### 3.1 线性布局
```kotlin
// 垂直列表
recyclerView.layoutManager = LinearLayoutManager(this)

// 水平列表
recyclerView.layoutManager = LinearLayoutManager(
    this,
    LinearLayoutManager.HORIZONTAL,
    false
)
```

### 3.2 网格布局
```kotlin
// 2列网格
recyclerView.layoutManager = GridLayoutManager(this, 2)

// 3列网格，第一个项目占2列
val layoutManager = GridLayoutManager(this, 3)
layoutManager.spanSizeLookup = object : GridLayoutManager.SpanSizeLookup() {
    override fun getSpanSize(position: Int): Int {
        return if (position == 0) 2 else 1
    }
}
recyclerView.layoutManager = layoutManager
```

### 3.3 瀑布流布局
```kotlin
recyclerView.layoutManager = StaggeredGridLayoutManager(
    2, // 列数
    StaggeredGridLayoutManager.VERTICAL
)
```

## 4. 高级特性

### 4.1 添加分割线
```kotlin
recyclerView.addItemDecoration(
    DividerItemDecoration(
        this,
        DividerItemDecoration.VERTICAL
    )
)
```

### 4.2 自定义分割线
```kotlin
class CustomDividerItemDecoration(
    private val color: Int,
    private val height: Int
) : RecyclerView.ItemDecoration() {

    override fun onDraw(c: Canvas, parent: RecyclerView, state: RecyclerView.State) {
        val left = parent.paddingLeft
        val right = parent.width - parent.paddingRight

        for (i in 0 until parent.childCount - 1) {
            val child = parent.getChildAt(i)
            val params = child.layoutParams as RecyclerView.LayoutParams
            val top = child.bottom + params.bottomMargin
            val bottom = top + height

            c.drawRect(left.toFloat(), top.toFloat(), right.toFloat(), bottom.toFloat(),
                Paint().apply {
                    this.color = color
                })
        }
    }
}
```

### 4.3 添加动画
```kotlin
// 设置默认动画
recyclerView.itemAnimator = DefaultItemAnimator()

// 自定义动画
class CustomItemAnimator : DefaultItemAnimator() {
    override fun animateAdd(holder: RecyclerView.ViewHolder): Boolean {
        holder.itemView.alpha = 0f
        holder.itemView.animate()
            .alpha(1f)
            .setDuration(300)
            .start()
        return true
    }
}
```

## 5. 性能优化

### 5.1 设置固定大小
```kotlin
recyclerView.setHasFixedSize(true)
```

### 5.2 预加载
```kotlin
recyclerView.setItemViewCacheSize(20)
```

### 5.3 使用 DiffUtil
```kotlin
class ItemDiffCallback : DiffUtil.ItemCallback<Item>() {
    override fun areItemsTheSame(oldItem: Item, newItem: Item): Boolean {
        return oldItem.id == newItem.id
    }

    override fun areContentsTheSame(oldItem: Item, newItem: Item): Boolean {
        return oldItem == newItem
    }
}

// 在适配器中使用
class ItemAdapter : ListAdapter<Item, ItemAdapter.ViewHolder>(ItemDiffCallback())
```

## 6. 实践练习

### 6.1 实现一个待办事项列表
```kotlin
data class TodoItem(
    val id: Int,
    val title: String,
    val isCompleted: Boolean
)

class TodoAdapter(
    private val onItemClick: (TodoItem) -> Unit,
    private val onCheckClick: (TodoItem) -> Unit
) : RecyclerView.Adapter<TodoAdapter.ViewHolder>() {

    private var items: List<TodoItem> = emptyList()

    fun updateItems(newItems: List<TodoItem>) {
        val diffCallback = object : DiffUtil.Callback() {
            override fun getOldListSize() = items.size
            override fun getNewListSize() = newItems.size
            override fun areItemsTheSame(oldPos: Int, newPos: Int) =
                items[oldPos].id == newItems[newPos].id
            override fun areContentsTheSame(oldPos: Int, newPos: Int) =
                items[oldPos] == newItems[newPos]
        }
        
        val diffResult = DiffUtil.calculateDiff(diffCallback)
        items = newItems
        diffResult.dispatchUpdatesTo(this)
    }

    // ViewHolder 和 其他方法实现...
}
```

## 7. 最佳实践

### 7.1 代码组织
- 将适配器逻辑分离到单独的类
- 使用 ViewBinding 代替 findViewById
- 实现 DiffUtil 进行高效更新
- 使用 ViewModel 管理数据

### 7.2 性能考虑
- 避免在 onBindViewHolder 中创建对象
- 使用 setHasFixedSize 优化性能
- 合理使用缓存机制
- 实现高效的 DiffUtil

## 8. 下一步
完成 RecyclerView 学习后，您可以：
1. 学习 [自定义 View](./docs/12_custom_view.md)
2. 了解 [SharedPreferences](./docs/13_shared_preferences.md)
3. 开始实践项目开发

## 9. 参考资料
- [RecyclerView 官方文档](https://developer.android.com/guide/topics/ui/layout/recyclerview)
- [Android 开发者指南](https://developer.android.com/guide)
- [Material Design 列表指南](https://material.io/components/lists) 