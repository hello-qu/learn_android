# Android Studio 使用指南

## 1. Android Studio 界面概览

### 1.1 主要界面区域
- **工具栏（Toolbar）**：包含常用操作按钮
- **导航栏（Navigation Bar）**：项目文件导航
- **编辑区（Editor）**：代码编辑区域
- **工具窗口（Tool Windows）**：各种功能面板
- **状态栏（Status Bar）**：显示项目状态信息

### 1.2 常用工具窗口
- **Project**：项目文件结构
- **Logcat**：日志查看器
- **Build**：构建输出
- **Run**：运行配置
- **Debug**：调试工具
- **Terminal**：终端
- **Device Manager**：设备管理器

## 2. 项目结构

### 2.1 标准项目结构
```
app/
├── manifests/        # 应用配置文件
├── java/            # Kotlin/Java 源代码
├── res/             # 资源文件
│   ├── layout/      # 布局文件
│   ├── values/      # 值资源
│   ├── drawable/    # 图片资源
│   └── mipmap/      # 应用图标
└── build.gradle     # 模块级构建配置
```

### 2.2 重要配置文件
- **build.gradle (Project)**：项目级构建配置
- **build.gradle (Module)**：模块级构建配置
- **AndroidManifest.xml**：应用清单文件
- **gradle.properties**：Gradle 属性配置

## 3. 常用功能

### 3.1 代码编辑
```kotlin
// 1. 代码补全
// 输入时按 Ctrl + Space (Windows) 或 Command + Space (Mac)

// 2. 快速修复
// 按 Alt + Enter (Windows) 或 Option + Enter (Mac)

// 3. 重构
// 按 Shift + F6 重命名
// 按 Ctrl + Alt + L (Windows) 或 Command + Option + L (Mac) 格式化代码
```

### 3.2 布局编辑器
1. **设计视图**：可视化编辑界面
2. **代码视图**：直接编辑 XML
3. **预览**：实时预览布局效果
4. **属性面板**：编辑组件属性

### 3.3 调试技巧
1. **断点设置**
   - 点击行号设置断点
   - 右键断点设置条件

2. **调试控制**
   - F8：单步执行
   - F7：步入
   - Shift + F8：步出
   - F9：继续执行

3. **变量查看**
   - 在调试窗口查看变量值
   - 使用表达式求值

## 4. 实用快捷键

### 4.1 编辑操作
- **Ctrl + C / Command + C**：复制
- **Ctrl + V / Command + V**：粘贴
- **Ctrl + X / Command + X**：剪切
- **Ctrl + Z / Command + Z**：撤销
- **Ctrl + Y / Command + Y**：重做

### 4.2 导航操作
- **Ctrl + N / Command + O**：查找类
- **Ctrl + Shift + N / Command + Shift + O**：查找文件
- **Ctrl + B / Command + B**：跳转到声明
- **Alt + F7 / Option + F7**：查找使用

### 4.3 重构操作
- **Shift + F6**：重命名
- **Ctrl + Alt + L / Command + Option + L**：格式化代码
- **Ctrl + Alt + O / Command + Option + O**：优化导入

## 5. 性能优化工具

### 5.1 性能分析器
1. **CPU Profiler**
   - 分析 CPU 使用情况
   - 查看方法调用栈

2. **Memory Profiler**
   - 监控内存使用
   - 检测内存泄漏

3. **Network Profiler**
   - 监控网络请求
   - 分析网络性能

### 5.2 布局检查器
- 检查视图层次结构
- 分析布局性能
- 调试布局问题

## 6. 版本控制集成

### 6.1 Git 操作
1. **提交更改**
   - Ctrl + K / Command + K

2. **更新项目**
   - Ctrl + T / Command + T

3. **查看历史**
   - Alt + 9 / Command + 9

### 6.2 分支管理
- 创建分支
- 切换分支
- 合并分支

## 7. 实用技巧

### 7.1 代码模板
```kotlin
// 输入 "sout" 自动展开为：
System.out.println()

// 输入 "psvm" 自动展开为：
public static void main(String[] args) {
}
```

### 7.2 实时模板
1. **创建新模板**
   - File > Settings > Editor > Live Templates

2. **使用模板**
   - 输入模板缩写
   - 按 Tab 展开

### 7.3 代码生成
- **Alt + Insert / Command + N**：生成代码
- 生成 getter/setter
- 生成构造函数
- 生成重写方法

## 8. 常见问题解决

### 8.1 构建问题
1. **Gradle 同步失败**
   - 检查网络连接
   - 更新 Gradle 版本
   - 清理项目缓存

2. **编译错误**
   - 查看 Build 窗口
   - 检查依赖配置
   - 更新 SDK 版本

### 8.2 性能问题
1. **IDE 运行缓慢**
   - 增加内存分配
   - 禁用不必要的插件
   - 清理缓存

2. **模拟器性能**
   - 使用硬件加速
   - 调整模拟器设置
   - 考虑使用真机调试

## 9. 下一步
完成 Android Studio 学习后，您可以：
1. 开始学习 [Android 应用架构](./04_android_architecture.md)
2. 了解 [Activity 生命周期](./05_activity_lifecycle.md)
3. 尝试创建您的第一个 Android 项目

## 10. 参考资料
- [Android Studio 官方文档](https://developer.android.com/studio)
- [Android Studio 快捷键指南](https://developer.android.com/studio/intro/keyboard-shortcuts)
- [Android Studio 性能优化指南](https://developer.android.com/studio/profile) 