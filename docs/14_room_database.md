# Room 数据库使用指南

## 1. Room 概述

### 1.1 什么是 Room？
Room 是 Android 官方提供的：
- SQLite 抽象层
- 编译时 SQL 验证
- 便捷的数据库访问
- 支持协程和 Flow

### 1.2 主要组件
- **Entity**: 数据表结构
- **DAO**: 数据访问对象
- **Database**: 数据库类
- **TypeConverter**: 类型转换器

## 2. 基本使用

### 2.1 添加依赖
```gradle
dependencies {
    def room_version = "2.6.1"
    
    implementation "androidx.room:room-runtime:$room_version"
    kapt "androidx.room:room-compiler:$room_version"
    implementation "androidx.room:room-ktx:$room_version"
}
```

### 2.2 创建实体类
```kotlin
@Entity(tableName = "users")
data class User(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    
    @ColumnInfo(name = "user_name")
    val name: String,
    
    @ColumnInfo(name = "user_age")
    val age: Int,
    
    @ColumnInfo(name = "user_email")
    val email: String
)
```

### 2.3 创建 DAO
```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAllUsers(): Flow<List<User>>
    
    @Query("SELECT * FROM users WHERE id = :userId")
    suspend fun getUserById(userId: Int): User?
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUser(user: User)
    
    @Update
    suspend fun updateUser(user: User)
    
    @Delete
    suspend fun deleteUser(user: User)
    
    @Query("DELETE FROM users")
    suspend fun deleteAllUsers()
}
```

### 2.4 创建数据库类
```kotlin
@Database(entities = [User::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
    
    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null
        
        fun getDatabase(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "app_database"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

## 3. 高级特性

### 3.1 关系映射
```kotlin
// 一对多关系
@Entity
data class User(
    @PrimaryKey val id: Int,
    val name: String
)

@Entity
data class Book(
    @PrimaryKey val id: Int,
    val title: String,
    val userId: Int
)

data class UserWithBooks(
    @Embedded val user: User,
    @Relation(
        parentColumn = "id",
        entityColumn = "userId"
    )
    val books: List<Book>
)

@Dao
interface UserDao {
    @Transaction
    @Query("SELECT * FROM users")
    fun getUsersWithBooks(): Flow<List<UserWithBooks>>
}
```

### 3.2 类型转换器
```kotlin
class Converters {
    @TypeConverter
    fun fromTimestamp(value: Long?): Date? {
        return value?.let { Date(it) }
    }
    
    @TypeConverter
    fun dateToTimestamp(date: Date?): Long? {
        return date?.time
    }
}

@Database(entities = [User::class], version = 1)
@TypeConverters(Converters::class)
abstract class AppDatabase : RoomDatabase()
```

### 3.3 数据库迁移
```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("""
            ALTER TABLE users 
            ADD COLUMN phone_number TEXT
        """)
    }
}

Room.databaseBuilder(context, AppDatabase::class.java, "app_database")
    .addMigrations(MIGRATION_1_2)
    .build()
```

## 4. 使用示例

### 4.1 Repository 模式
```kotlin
class UserRepository(private val userDao: UserDao) {
    val allUsers: Flow<List<User>> = userDao.getAllUsers()
    
    suspend fun insert(user: User) {
        userDao.insertUser(user)
    }
    
    suspend fun update(user: User) {
        userDao.updateUser(user)
    }
    
    suspend fun delete(user: User) {
        userDao.deleteUser(user)
    }
}
```

### 4.2 ViewModel 实现
```kotlin
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    val allUsers: Flow<List<User>> = repository.allUsers
    
    fun insert(user: User) = viewModelScope.launch {
        repository.insert(user)
    }
    
    fun update(user: User) = viewModelScope.launch {
        repository.update(user)
    }
    
    fun delete(user: User) = viewModelScope.launch {
        repository.delete(user)
    }
}
```

### 4.3 Activity 中使用
```kotlin
class MainActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels {
        object : ViewModelProvider.Factory {
            override fun <T : ViewModel> create(modelClass: Class<T>): T {
                val database = AppDatabase.getDatabase(applicationContext)
                val repository = UserRepository(database.userDao())
                return UserViewModel(repository) as T
            }
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // 观察数据变化
        lifecycleScope.launch {
            viewModel.allUsers.collect { users ->
                // 更新 UI
                updateUI(users)
            }
        }
    }
}
```

## 5. 最佳实践

### 5.1 性能优化
- 使用协程进行异步操作
- 合理使用索引
- 避免在主线程访问数据库
- 使用 Flow 进行响应式编程

### 5.2 代码组织
- 使用 Repository 模式
- 实现依赖注入
- 遵循单一职责原则
- 使用事务保证数据一致性

## 6. 实践练习

### 6.1 待办事项应用
```kotlin
@Entity(tableName = "todos")
data class Todo(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val title: String,
    val description: String,
    val isCompleted: Boolean,
    val createdAt: Date
)

@Dao
interface TodoDao {
    @Query("SELECT * FROM todos ORDER BY createdAt DESC")
    fun getAllTodos(): Flow<List<Todo>>
    
    @Insert
    suspend fun insert(todo: Todo)
    
    @Update
    suspend fun update(todo: Todo)
    
    @Delete
    suspend fun delete(todo: Todo)
}
```

## 7. 注意事项

### 7.1 数据库设计
- 合理设计表结构
- 使用适当的数据类型
- 建立必要的关系
- 考虑数据完整性

### 7.2 错误处理
- 处理数据库异常
- 实现回滚机制
- 记录错误日志
- 提供用户反馈

## 8. 下一步
完成 Room 数据库学习后，您可以：
1. 学习 [文件存储](./docs/15_file_storage.md)
2. 了解 [网络请求基础](./docs/16_network_basics.md)
3. 开始实践项目开发

## 9. 参考资料
- [Room 官方文档](https://developer.android.com/training/data-storage/room)
- [Android 数据存储指南](https://developer.android.com/guide/topics/data)
- [Android 开发者指南](https://developer.android.com/guide) 