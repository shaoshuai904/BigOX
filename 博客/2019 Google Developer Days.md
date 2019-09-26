## 2019 Google Developer Days


本文为2019 Google Developer Days大会内容总结。详情可参阅[官网](https://events.google.cn/intl/zh-CN/developerdays2019/).

![思维导图](https://github.com/shaoshuai904/BigOX/blob/master/博客/res/2019_google_dev_days.png)


---

### Android


![思维导图](https://github.com/shaoshuai904/BigOX/blob/master/博客/res/2019_android.png)


> Android 10新特性

![android 10](https://s3.ifanr.com/wp-content/uploads/2019/09/HeroArticlePage_1500x850.max-1000x1000.png!720)

- 系统级暗黑模式

详细信息请查看官网 [https://developer.android.com/guide/topics/ui/look-and-feel/darktheme](https://developer.android.com/guide/topics/ui/look-and-feel/darktheme)

![全局暗黑](https://s3.ifanr.com/wp-content/uploads/2019/09/concact-2019-09-04-173004.png!720)

- 通知中的智能回复

详细信息请查看官网 [https://developer.android.com/reference/android/app/Notification.Action.Builder#setAllowGeneratedReplies(boolean)](https://developer.android.com/reference/android/app/Notification.Action.Builder#setAllowGeneratedReplies(boolean))

![notification](https://github.com/shaoshuai904/BigOX/blob/master/博客/res/android_10_notification.png)

![notification](https://s3.ifanr.com/wp-content/uploads/2019/09/Android-105.gif)

- 隐私和位置
	- 在后台访问设备位置需要获得许可
	- 保护外部存储中的用户数据
	- 限制从后台启动的活动
	- MAC地址随机化
	- [对不可重置设备标识符的访问限制](https://developer.android.com/training/articles/user-data-ids)

详细信息请查看官网 [https://developer.android.com/about/versions/10/privacy](https://developer.android.com/about/versions/10/privacy)

![权限更新](https://s3.ifanr.com/wp-content/uploads/2019/09/concact-2019-09-04-172506.png!720)

- 手势导航

详细信息请查看官网 [https://developer.android.com/guide/navigation/gesturenav](https://developer.android.com/guide/navigation/gesturenav)

![notification](https://s3.ifanr.com/wp-content/uploads/2019/09/23-1440x1346.jpg!720)

- 支持折叠屏

详细信息请查看官网 [https://developer.android.com/guide/topics/ui/foldables](https://developer.android.com/guide/topics/ui/foldables)

![折叠屏](https://developer.android.com/images/guide/topics/ui/foldables/fold-out-app-continuity.gif)

- 支持5G

详细信息请查看官网 [https://developer.android.com/reference/android/net/ConnectivityManager](https://developer.android.com/reference/android/net/ConnectivityManager)

- 专注模式
- SAW* 被废止
- TLS 1.3 默认开启
- 生物识别登陆
- ShareSheet API
- Bubbles
- ...

> Kotlin First

- 新 Jetpack API 和功能首先在Kotlin中提供
- 新项目用Kotlin编写

> Android Jetpack

- Camera X
	- 兼容至L (90%设备)
	- 设备一致性
	- 使用方便
- AndroidX Test
- 新包
	- Jetpack Compose 下一代UI工具包
	- Jetpack Security Library 加密包，支持6.0+
	- Jetpack Benchmark Library 测试代码性能
- 架构组件
	- Data Binding
	- View Binding 
	- WorkManager
	- Navigation
	- SavedState for ViewModel
	- Room
	- ViewPager2

> ConstraintLayout 和 MotionLayout 增强

- ```ConstraintLayout```是功能更强大的```RelativeLayout```，新项目默认布局
- class ```MotionLayout``` **extends** ```ConstraintLayout```

> App Bundle实现自定义交付，并共享测试版本。

- 10%-60%的app瘦身
- 定制化分发
	- 应用内更新
	- 按条件分发
	- 按需分发
	- 游戏资源分发
- Demo：[Plaid](https://github.com/android/plaid)

> 借助Android Studio工具，基准化分析，提升应用性能

- Profiler
- Build -> Analyze APK...

> 无障碍

- 有10亿人存在各种不同程度的障碍，无障碍化app势在必行

> Android Studio 3.5

- Project Marble 磐石计划
- 离线组件
- 增加内存堆
- Lint代码检查
- 构建速度
- Apply Changes

---

### Flutter

- 1.9版本发布
- 混合开发，势在必行
- Flutter如何呈现组件
- 快速构建Flutter混合应用

---

### TensorFlow

- TensorFlow 2.0
- TensorFlow Lite - 面向移动和物联网设备
- TensorFlow.js : JavaScript平台的机器学习
- Swift for TensorFlow : 无边界机器学习
- 机器学习在去中心化数据上的应用
- SmileAR : 爱奇艺移动端AR解决方案
- ElasticDL : 弹性分布式深度学习
- 在Arm架构上运行TensorFlow Lite
- 使用tf.distribute拓展TensorFlow
- 借助TensorFlow Extended打造机器学习服务










