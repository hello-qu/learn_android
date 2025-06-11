# Android 文件存储指南

## 1. 文件存储概述

### 1.1 存储位置
- **内部存储**：应用私有目录
- **外部存储**：共享存储空间
- **缓存目录**：临时文件存储

### 1.2 权限要求
```xml
<!-- 外部存储权限 -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

## 2. 内部存储

### 2.1 基本操作
```kotlin
class FileStorageManager(private val context: Context) {
    // 写入文件
    fun writeToInternalStorage(fileName: String, content: String) {
        context.openFileOutput(fileName, Context.MODE_PRIVATE).use { output ->
            output.write(content.toByteArray())
        }
    }
    
    // 读取文件
    fun readFromInternalStorage(fileName: String): String {
        return context.openFileInput(fileName).bufferedReader().use { it.readText() }
    }
    
    // 删除文件
    fun deleteFile(fileName: String): Boolean {
        return context.deleteFile(fileName)
    }
    
    // 获取文件列表
    fun getFileList(): Array<String> {
        return context.fileList()
    }
}
```

### 2.2 缓存文件
```kotlin
class CacheManager(private val context: Context) {
    // 写入缓存
    fun writeToCache(fileName: String, content: String) {
        File(context.cacheDir, fileName).writeText(content)
    }
    
    // 读取缓存
    fun readFromCache(fileName: String): String {
        return File(context.cacheDir, fileName).readText()
    }
    
    // 清除缓存
    fun clearCache() {
        context.cacheDir.deleteRecursively()
    }
}
```

## 3. 外部存储

### 3.1 检查存储状态
```kotlin
class ExternalStorageManager(private val context: Context) {
    fun isExternalStorageWritable(): Boolean {
        return Environment.getExternalStorageState() == Environment.MEDIA_MOUNTED
    }
    
    fun isExternalStorageReadable(): Boolean {
        return Environment.getExternalStorageState() in 
            setOf(Environment.MEDIA_MOUNTED, Environment.MEDIA_MOUNTED_READ_ONLY)
    }
}
```

### 3.2 基本操作
```kotlin
class ExternalStorageManager(private val context: Context) {
    // 获取外部存储目录
    private fun getExternalStorageDir(fileName: String): File {
        return File(context.getExternalFilesDir(null), fileName)
    }
    
    // 写入文件
    fun writeToExternalStorage(fileName: String, content: String) {
        getExternalStorageDir(fileName).writeText(content)
    }
    
    // 读取文件
    fun readFromExternalStorage(fileName: String): String {
        return getExternalStorageDir(fileName).readText()
    }
    
    // 删除文件
    fun deleteFile(fileName: String): Boolean {
        return getExternalStorageDir(fileName).delete()
    }
}
```

## 4. 媒体文件

### 4.1 图片存储
```kotlin
class ImageStorageManager(private val context: Context) {
    // 保存图片到相册
    fun saveImageToGallery(bitmap: Bitmap, fileName: String): Uri? {
        val contentValues = ContentValues().apply {
            put(MediaStore.Images.Media.DISPLAY_NAME, fileName)
            put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg")
            put(MediaStore.Images.Media.RELATIVE_PATH, Environment.DIRECTORY_PICTURES)
        }
        
        val resolver = context.contentResolver
        val uri = resolver.insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, contentValues)
        
        uri?.let {
            resolver.openOutputStream(it)?.use { stream ->
                bitmap.compress(Bitmap.CompressFormat.JPEG, 100, stream)
            }
        }
        
        return uri
    }
    
    // 从相册加载图片
    fun loadImageFromGallery(uri: Uri): Bitmap? {
        return context.contentResolver.openInputStream(uri)?.use { stream ->
            BitmapFactory.decodeStream(stream)
        }
    }
}
```

### 4.2 视频存储
```kotlin
class VideoStorageManager(private val context: Context) {
    // 保存视频
    fun saveVideo(uri: Uri, fileName: String): Uri? {
        val contentValues = ContentValues().apply {
            put(MediaStore.Video.Media.DISPLAY_NAME, fileName)
            put(MediaStore.Video.Media.MIME_TYPE, "video/mp4")
            put(MediaStore.Video.Media.RELATIVE_PATH, Environment.DIRECTORY_MOVIES)
        }
        
        val resolver = context.contentResolver
        return resolver.insert(MediaStore.Video.Media.EXTERNAL_CONTENT_URI, contentValues)
    }
}
```

## 5. 文件操作工具类

### 5.1 文件工具类
```kotlin
object FileUtils {
    // 获取文件大小
    fun getFileSize(file: File): String {
        val size = file.length()
        return when {
            size < 1024 -> "$size B"
            size < 1024 * 1024 -> "${size / 1024} KB"
            else -> "${size / (1024 * 1024)} MB"
        }
    }
    
    // 获取文件扩展名
    fun getFileExtension(file: File): String {
        return file.name.substringAfterLast('.', "")
    }
    
    // 检查文件类型
    fun isImageFile(file: File): Boolean {
        return file.extension.lowercase() in listOf("jpg", "jpeg", "png", "gif")
    }
}
```

### 5.2 目录管理
```kotlin
class DirectoryManager(private val context: Context) {
    // 创建目录
    fun createDirectory(dirName: String): File {
        val dir = File(context.filesDir, dirName)
        if (!dir.exists()) {
            dir.mkdirs()
        }
        return dir
    }
    
    // 列出目录内容
    fun listDirectory(dirName: String): List<File> {
        return File(context.filesDir, dirName).listFiles()?.toList() ?: emptyList()
    }
}
```

## 6. 实践练习

### 6.1 文件管理器
```kotlin
class FileManager(private val context: Context) {
    private val internalStorage = FileStorageManager(context)
    private val externalStorage = ExternalStorageManager(context)
    private val imageStorage = ImageStorageManager(context)
    
    // 保存用户数据
    fun saveUserData(userData: String) {
        internalStorage.writeToInternalStorage("user_data.txt", userData)
    }
    
    // 保存图片
    fun saveImage(bitmap: Bitmap, fileName: String): Uri? {
        return imageStorage.saveImageToGallery(bitmap, fileName)
    }
    
    // 导出数据
    fun exportData(content: String, fileName: String) {
        externalStorage.writeToExternalStorage(fileName, content)
    }
}
```

## 7. 最佳实践

### 7.1 性能优化
- 使用缓冲流进行文件操作
- 大文件操作使用协程
- 及时释放资源
- 定期清理缓存

### 7.2 安全考虑
- 敏感数据加密存储
- 检查存储权限
- 验证文件类型
- 处理存储空间不足

## 8. 注意事项

### 8.1 存储限制
- 内部存储空间有限
- 外部存储需要权限
- 考虑 Android 10+ 的分区存储
- 注意文件大小限制

### 8.2 错误处理
- 处理文件不存在
- 处理权限拒绝
- 处理存储空间不足
- 处理文件损坏

## 9. 下一步
完成文件存储学习后，您可以：
1. 学习 [网络请求基础](./docs/16_network_basics.md)
2. 了解 [图片加载](./docs/17_image_loading.md)
3. 开始实践项目开发

## 10. 参考资料
- [Android 存储指南](https://developer.android.com/guide/topics/data)
- [Android 文件存储最佳实践](https://developer.android.com/training/data-storage)
- [Android 开发者指南](https://developer.android.com/guide) 