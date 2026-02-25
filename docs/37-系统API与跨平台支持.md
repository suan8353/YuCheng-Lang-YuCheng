# 系统 API 与跨平台支持

> **实现状态**: ⚠️ 部分实现
> - ✅ 语法支持完整（`外部函数` 关键字已定义）
> - ✅ AST 节点完整（`ExternalFunctionDecl` 已实现）
> - ✅ 互操作框架基础已实现（`lang_interop.rs`）
> - ⚠️ 系统 API 实际绑定部分实现（通过多语言互操作实现）
> - ⚠️ 解释器集成部分实现（需要进一步路由外部函数调用）
> 
> **代码位置**: 
> - `compiler/src/ast.rs` (ExternalFunctionDecl)
> - `compiler/src/heterogeneous.rs` (多语言互操作)

## 一、当前状态分析

### 1.1 现有能力

| 功能 | 状态 | 说明 |
|------|------|------|
| 语法支持 | ✅ | `外部函数` 关键字已定义 |
| AST 节点 | ✅ | `ExternalFunctionDecl` 已实现 |
| 互操作框架 | ✅ | `lang_interop.rs` 已有基础 |
| 实际绑定 | ⚠️ | 通过多语言互操作实现，系统API绑定待完善 |
| 解释器集成 | ⚠️ | 部分实现，需要进一步路由外部函数调用 |

### 1.2 语法示例

```yucheng
// 声明外部函数
外部函数 读取文件(路径: 文本) -> 文本 from "system";
外部函数 写入文件(路径: 文本, 内容: 文本) from "system";
外部函数 获取环境变量(键: 文本) -> 文本 from "system";
```

---

## 二、系统 API 设计

### 2.1 统一抽象原则

> **"一次声明，多端实现"**

用户用同一套 `外部函数` 声明，YuCheng 在不同平台自动绑定对应原生 API。

### 2.2 标准系统 API 列表

#### 文件系统

```yucheng
外部函数 读取文件(路径: 文本) -> 文本 from "system";
外部函数 写入文件(路径: 文本, 内容: 文本) from "system";
外部函数 追加文件(路径: 文本, 内容: 文本) from "system";
外部函数 删除文件(路径: 文本) -> 布尔 from "system";
外部函数 文件存在(路径: 文本) -> 布尔 from "system";
外部函数 列出目录(路径: 文本) -> 文本[] from "system";
外部函数 创建目录(路径: 文本) -> 布尔 from "system";
外部函数 删除目录(路径: 文本) -> 布尔 from "system";
外部函数 获取文件大小(路径: 文本) -> 整数 from "system";
外部函数 获取修改时间(路径: 文本) -> 整数 from "system";
```

#### 环境与进程

```yucheng
外部函数 获取环境变量(键: 文本) -> 文本 from "system";
外部函数 设置环境变量(键: 文本, 值: 文本) from "system";
外部函数 获取当前目录() -> 文本 from "system";
外部函数 设置当前目录(路径: 文本) -> 布尔 from "system";
外部函数 执行命令(命令: 文本) -> 整数 from "system";
外部函数 执行命令获取输出(命令: 文本) -> 文本 from "system";
外部函数 退出(状态码: 整数) from "system";
外部函数 获取进程ID() -> 整数 from "system";
```

#### 时间与日期

```yucheng
外部函数 当前时间戳() -> 整数 from "system";
外部函数 当前日期时间() -> 文本 from "system";
外部函数 休眠(毫秒: 整数) from "system";
```

#### 网络（基础）

```yucheng
外部函数 HTTP获取(网址: 文本) -> 文本 from "system";
外部函数 HTTP发送(网址: 文本, 方法: 文本, 数据: 文本) -> 文本 from "system";
```

#### 系统信息

```yucheng
外部函数 获取操作系统() -> 文本 from "system";
外部函数 获取CPU架构() -> 文本 from "system";
外部函数 获取主机名() -> 文本 from "system";
外部函数 获取用户名() -> 文本 from "system";
外部函数 获取主目录() -> 文本 from "system";
```

---

## 三、跨平台实现策略

### 3.1 桌面端（Windows/macOS/Linux）

| 类别 | Windows | macOS | Linux | YuCheng 函数 |
|------|---------|-------|-------|-------------|
| 文件 I/O | CreateFile | open | open | `读取文件` |
| 进程 | CreateProcess | fork | fork | `执行命令` |
| 通知 | Toast API | NSUserNotification | libnotify | `显示通知` |
| 剪贴板 | OpenClipboard | NSPasteboard | xclip | `设置剪贴板` |

### 3.2 移动端支持策略

#### Android（嵌入式引擎）

```kotlin
// MainActivity.kt
val engine = YuchengEngine()
engine.eval("""
    外部函数 获取GPS() from "android";
    显示通知("位置", 获取GPS())
""")
```

#### iOS（嵌入式引擎）

```swift
// ViewController.swift
let engine = YuchengEngine()
engine.evaluate("""
    外部函数 请求权限(类型: 文本) from "ios";
    请求权限("定位")
""")
```

### 3.3 Web 端（Wasm）

- 编译为 WebAssembly
- 通过 Web API 访问设备功能
- 支持 PWA 安装

---

## 四、安全与权限模型

### 4.1 权限声明

在 `.ycl` 配置文件中声明所需权限：

```yaml
permissions:
  filesystem:
    read: ["./", "~/Documents"]
    write: ["./output/"]
  network:
    allow: ["api.example.com"]
  process:
    allow_exec: false
```

### 4.2 沙箱模式

- Web IDE 中禁用危险 API
- 移动端遵循平台权限模型
- 桌面端可配置沙箱级别

---

## 五、实现路线图

### Phase 1：核心文件 API（已完成基础）

- [x] 文件读写
- [x] 目录操作
- [x] 文件信息

### Phase 2：环境与进程（已完成）

- [x] 环境变量
- [x] 命令执行
- [x] 进程管理

### Phase 3：网络 API（基础已完成）

- [x] HTTP 客户端（基础）
- [ ] WebSocket
- [ ] TCP/UDP

### Phase 4：平台特定 API

- [ ] Windows 特有
- [ ] macOS 特有
- [ ] Linux 特有
- [ ] Android 特有
- [ ] iOS 特有

---

## 六、使用示例

### 6.1 文件操作

```yucheng
// 读取配置文件
变量 配置 = 读取文件("config.json")
打印(配置)

// 写入日志
写入文件("app.log", "程序启动\n")
追加文件("app.log", "操作完成\n")

// 列出目录
变量 文件列表 = 列出目录("./")
循环 文件 在 文件列表 {
    打印(文件)
}
```

### 6.2 系统信息

```yucheng
打印("操作系统: " + 获取操作系统())
打印("用户名: " + 获取用户名())
打印("主目录: " + 获取主目录())
打印("当前目录: " + 获取当前目录())
```

### 6.3 命令执行

```yucheng
// 执行命令
变量 结果 = 执行命令获取输出("dir")
打印(结果)

// 带返回码
变量 状态码 = 执行命令("echo hello")
如果 状态码 != 0 {
    打印("命令执行失败")
}
```


---

## 七、实现状态

### 7.1 已实现的系统 API

| API 名称 | 中文名 | 状态 | 说明 |
|---------|-------|------|------|
| `read_file` | `读取文件` | ✅ | 读取文件内容为文本 |
| `write_file` | `写入文件` | ✅ | 写入文本到文件 |
| `append_file` | `追加文件` | ✅ | 追加内容到文件末尾 |
| `delete_file` | `删除文件` | ✅ | 删除指定文件 |
| `file_exists` | `文件存在` | ✅ | 检查文件是否存在 |
| `list_dir` | `列出目录` | ✅ | 列出目录内容 |
| `create_dir` | `创建目录` | ✅ | 创建目录（递归） |
| `remove_dir` | `删除目录` | ✅ | 删除目录（递归） |
| `file_size` | `获取文件大小` | ✅ | 获取文件字节数 |
| `is_dir` | `是目录` | ✅ | 检查是否为目录 |
| `is_file` | `是文件` | ✅ | 检查是否为文件 |
| `get_env` | `获取环境变量` | ✅ | 获取环境变量值 |
| `set_env` | `设置环境变量` | ✅ | 设置环境变量 |
| `get_cwd` | `获取当前目录` | ✅ | 获取当前工作目录 |
| `exec` | `执行命令` | ✅ | 执行系统命令 |
| `exec_output` | `执行命令获取输出` | ✅ | 执行命令并获取输出 |
| `get_pid` | `获取进程ID` | ✅ | 获取当前进程 ID |
| `get_args` | `获取命令行参数` | ✅ | 获取命令行参数列表 |
| `timestamp` | `当前时间戳` | ✅ | 获取 Unix 时间戳 |
| `datetime` | `当前日期时间` | ✅ | 获取 ISO 8601 格式时间 |
| `sleep` | `休眠` | ✅ | 暂停执行指定毫秒 |
| `get_os` | `获取操作系统` | ✅ | 获取操作系统名称 |
| `get_arch` | `获取CPU架构` | ✅ | 获取 CPU 架构 |
| `get_hostname` | `获取主机名` | ✅ | 获取主机名 |
| `get_username` | `获取用户名` | ✅ | 获取当前用户名 |
| `get_home` | `获取主目录` | ✅ | 获取用户主目录 |
| `get_temp_dir` | `获取临时目录` | ✅ | 获取系统临时目录 |
| `http_get` | `HTTP获取` | ✅ | 发送 HTTP GET 请求 |

### 7.2 模块结构

```
compiler/src/
├── system_api.rs      # 系统 API 管理器（带权限控制）
├── stdlib.rs          # 标准库（注册系统 API 函数）
└── lib.rs             # 导出 SystemApiManager
```

### 7.3 使用方式

系统 API 已集成到标准库中，可直接在 YuCheng 代码中调用：

```yucheng
// 文件操作
变量 内容 = 读取文件("config.txt")
写入文件("output.txt", "Hello, World!")

// 系统信息
打印("操作系统: " + 获取操作系统())
打印("用户: " + 获取用户名())

// 命令执行
变量 结果 = 执行命令获取输出("dir")
打印(结果)

// 时间
打印("当前时间: " + 当前日期时间())
休眠(1000)  // 暂停 1 秒
```

### 7.4 权限模型

`SystemApiManager` 支持权限控制：

```rust
// 默认权限（允许当前目录读写）
let manager = SystemApiManager::new();

// 沙箱模式（严格限制）
let sandbox = SystemApiManager::with_permissions(SystemPermissions::sandbox());

// 自定义权限
let custom = SystemApiManager::with_permissions(SystemPermissions {
    read_paths: vec!["./".to_string()],
    write_paths: vec!["./output/".to_string()],
    network_allow: vec!["api.example.com".to_string()],
    allow_exec: false,
    sandbox_mode: true,
});
```

---

## 八、移动端 API 支持

### 8.1 已实现的移动端 API

| API 名称 | 中文名 | 状态 | 说明 |
|---------|-------|------|------|
| `get_platform` | `获取平台` | ✅ | 获取当前平台（android/ios/desktop） |
| `is_mobile` | `是移动端` | ✅ | 检查是否为移动平台 |
| `request_permission` | `请求权限` | ✅ | 请求移动端权限 |
| `check_permission` | `检查权限` | ✅ | 检查权限状态 |
| `get_location` | `获取位置` | ✅ | 获取 GPS 位置信息 |
| `show_notification` | `显示通知` | ✅ | 显示系统通知 |
| `show_toast` | `显示提示` | ✅ | 显示 Toast 提示 |
| `vibrate` | `振动` | ✅ | 设备振动 |
| `get_device_info` | `获取设备信息` | ✅ | 获取设备详细信息 |
| `get_network_status` | `获取网络状态` | ✅ | 获取网络连接状态 |
| `open_url` | `打开链接` | ✅ | 打开 URL |
| `share_text` | `分享文本` | ✅ | 分享文本内容 |
| `copy_to_clipboard` | `复制到剪贴板` | ✅ | 复制文本到剪贴板 |
| `read_clipboard` | `读取剪贴板` | ✅ | 读取剪贴板内容 |
| `take_photo` | `拍照` | ✅ | 调用相机拍照 |
| `pick_image` | `选择图片` | ✅ | 从相册选择图片 |
| `scan_qrcode` | `扫描二维码` | ✅ | 扫描二维码 |
| `authenticate_biometric` | `生物识别认证` | ✅ | 指纹/面容认证 |

### 8.2 移动端权限类型

| 权限 | Android | iOS |
|------|---------|-----|
| 相机 | `CAMERA` | `NSCameraUsageDescription` |
| 麦克风 | `RECORD_AUDIO` | `NSMicrophoneUsageDescription` |
| 位置 | `ACCESS_FINE_LOCATION` | `NSLocationWhenInUseUsageDescription` |
| 存储 | `READ_EXTERNAL_STORAGE` | `NSPhotoLibraryUsageDescription` |
| 联系人 | `READ_CONTACTS` | `NSContactsUsageDescription` |
| 通知 | `POST_NOTIFICATIONS` | `NSUserNotificationUsageDescription` |
| 生物识别 | `USE_BIOMETRIC` | `NSFaceIDUsageDescription` |
| 蓝牙 | `BLUETOOTH` | `NSBluetoothAlwaysUsageDescription` |

### 8.3 模块结构

```
compiler/src/
├── mobile_api.rs      # 移动端 API 管理器
│   ├── MobilePlatform        # 平台枚举
│   ├── MobilePermission      # 权限枚举
│   ├── MobileApiCallback     # 回调 trait（宿主实现）
│   ├── MobileApiManager      # API 管理器
│   ├── MockMobileCallback    # 模拟回调（桌面测试）
│   ├── GpsLocation           # GPS 位置数据
│   ├── DeviceInfo            # 设备信息
│   └── NetworkStatus         # 网络状态
├── stdlib.rs          # 标准库（注册移动端 API 函数）
└── lib.rs             # 导出移动端 API
```

### 8.4 使用示例

```yucheng
// 平台检测
打印("当前平台: " + 获取平台())
如果 是移动端() {
    打印("运行在移动设备上")
}

// 权限请求
如果 请求权限("位置") {
    变量 位置 = 获取位置()
    打印("纬度: " + 位置.纬度)
    打印("经度: " + 位置.经度)
}

// 通知与提示
显示通知("标题", "这是通知内容")
显示提示("操作成功")

// 设备功能
振动(200)  // 振动 200 毫秒
变量 设备 = 获取设备信息()
打印("设备型号: " + 设备.型号)

// 剪贴板
复制到剪贴板("要复制的文本")
变量 内容 = 读取剪贴板()

// 相机与二维码
变量 照片 = 拍照()  // 返回 Base64 编码
变量 二维码 = 扫描二维码()

// 生物识别
如果 生物识别认证("请验证身份") {
    打印("认证成功")
}
```

### 8.5 Android 集成

将 YuCheng 编译为 Android 库后，在 Kotlin 中使用：

```kotlin
// build.gradle
dependencies {
    implementation("dev.yucheng:yucheng-engine:1.0.0")
}

// MainActivity.kt
class MainActivity : AppCompatActivity() {
    private val engine = YuchengEngine()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // 设置移动端回调
        engine.setMobileCallback(object : MobileApiCallback {
            override fun requestPermission(permission: String): Boolean {
                return ActivityCompat.checkSelfPermission(
                    this@MainActivity, 
                    permission
                ) == PackageManager.PERMISSION_GRANTED
            }
            
            override fun getLocation(): GpsLocation? {
                // 使用 FusedLocationProviderClient
                return locationProvider.lastLocation?.let {
                    GpsLocation(it.latitude, it.longitude)
                }
            }
            
            override fun showNotification(title: String, body: String): Boolean {
                NotificationManagerCompat.from(this@MainActivity)
                    .notify(1, NotificationCompat.Builder(this@MainActivity, "channel")
                        .setContentTitle(title)
                        .setContentText(body)
                        .build())
                return true
            }
            
            // ... 其他回调实现
        })
        
        // 执行 YuCheng 代码
        engine.eval("""
            显示通知("欢迎", "YuCheng 移动端已启动")
        """)
    }
}
```

### 8.6 iOS 集成

将 YuCheng 编译为 iOS 库后，在 Swift 中使用：

```swift
// Podfile
pod 'YuchengEngine', '~> 1.0'

// ViewController.swift
import YuchengEngine

class ViewController: UIViewController {
    let engine = YuchengEngine()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 设置移动端回调
        engine.setMobileCallback(MobileCallbackImpl())
        
        // 执行 YuCheng 代码
        engine.eval("""
            如果 请求权限("位置") {
                变量 位置 = 获取位置()
                显示提示("位置: " + 位置.纬度 + ", " + 位置.经度)
            }
        """)
    }
}

class MobileCallbackImpl: MobileApiCallback {
    func requestPermission(_ permission: String) -> Bool {
        switch permission {
        case "位置", "location":
            return CLLocationManager.authorizationStatus() == .authorizedWhenInUse
        case "相机", "camera":
            return AVCaptureDevice.authorizationStatus(for: .video) == .authorized
        default:
            return false
        }
    }
    
    func getLocation() -> GpsLocation? {
        guard let location = locationManager.location else { return nil }
        return GpsLocation(
            latitude: location.coordinate.latitude,
            longitude: location.coordinate.longitude
        )
    }
    
    func showNotification(title: String, body: String) -> Bool {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        let request = UNNotificationRequest(identifier: UUID().uuidString, content: content, trigger: nil)
        UNUserNotificationCenter.current().add(request)
        return true
    }
    
    // ... 其他回调实现
}
```

### 8.7 桌面端模拟模式

在桌面环境开发时，移动端 API 使用模拟实现：

```rust
// 创建模拟模式管理器
let mut manager = MobileApiManager::mock();

// 所有 API 调用返回模拟数据
// - 权限请求总是返回 true
// - GPS 返回北京坐标 (39.9042, 116.4074)
// - 设备信息返回 "Mock Device"
// - 通知/Toast 打印到控制台
```

这允许在桌面环境完整测试移动端代码逻辑。


---

## 九、硬件控制 API 支持

### 9.1 已实现的硬件控制 API

| API 名称 | 中文名 | 状态 | 说明 |
|---------|-------|------|------|
| `get_cpu_info` | `获取CPU信息` | ✅ | 获取CPU详细信息（品牌、型号、核心数等） |
| `get_cpu_usage` | `获取CPU使用率` | ✅ | 获取CPU当前使用率（百分比） |
| `get_cpu_temperature` | `获取CPU温度` | ✅ | 获取CPU温度（摄氏度） |
| `get_cpu_frequency` | `获取CPU频率` | ✅ | 获取CPU当前频率（MHz） |
| `set_cpu_frequency` | `设置CPU频率` | ✅ | 设置CPU频率（需要管理员权限） |
| `set_cpu_affinity` | `设置CPU亲和性` | ✅ | 设置进程CPU亲和性（需要管理员权限） |
| `get_cpu_cores` | `获取CPU核心数` | ✅ | 获取物理核心数 |
| `get_cpu_threads` | `获取CPU线程数` | ✅ | 获取逻辑线程数 |
| `get_gpu_info` | `获取GPU信息` | ✅ | 获取GPU详细信息（名称、显存等） |
| `get_gpu_usage` | `获取GPU使用率` | ✅ | 获取GPU当前使用率（百分比） |
| `get_gpu_temperature` | `获取GPU温度` | ✅ | 获取GPU温度（摄氏度） |
| `get_gpu_memory` | `获取GPU内存` | ✅ | 获取GPU显存信息（总量、已用、空闲） |
| `set_gpu_frequency` | `设置GPU频率` | ✅ | 设置GPU频率（需要管理员权限） |
| `set_gpu_power_limit` | `设置GPU功耗限制` | ✅ | 设置GPU功耗限制（需要管理员权限） |
| `get_gpu_list` | `获取GPU列表` | ✅ | 获取所有GPU列表 |
| `get_memory_info` | `获取内存信息` | ✅ | 获取内存详细信息（总量、已用、空闲） |
| `get_memory_usage` | `获取内存使用率` | ✅ | 获取内存使用率（百分比） |
| `clear_memory` | `清理内存` | ✅ | 清理内存缓存（需要管理员权限） |

### 9.2 权限级别

硬件控制系统支持三种权限级别：

| 权限级别 | 说明 | 可用操作 |
|---------|------|---------|
| User（用户级） | 默认权限，只能查询信息 | 获取CPU/GPU/内存信息、使用率、温度等 |
| Admin（管理员级） | 可以控制硬件 | 设置CPU/GPU频率、功耗限制、清理内存等 |
| System（系统级） | 完全控制 | 所有操作 |

### 9.3 模块结构

```
compiler/src/
├── hardware_control.rs    # 硬件控制管理器
│   ├── HardwareControlManager     # 硬件控制管理器
│   ├── PermissionLevel            # 权限级别枚举
│   ├── CpuController              # CPU控制器
│   ├── CpuInfo                    # CPU信息结构
│   ├── GpuController              # GPU控制器
│   ├── GpuInfo                    # GPU信息结构
│   └── MemoryController           # 内存控制器
├── stdlib.rs              # 标准库（注册硬件控制API函数）
└── lib.rs                 # 导出硬件控制API
```

### 9.4 使用示例

```yucheng
// CPU信息查询
变量 cpu信息 = 获取CPU信息()
打印("CPU品牌: " + cpu信息.品牌)
打印("CPU核心数: " + cpu信息.核心数)
打印("CPU线程数: " + cpu信息.线程数)

// CPU使用率监控
变量 cpu使用率 = 获取CPU使用率()
打印("CPU使用率: " + cpu使用率 + "%")

// CPU温度监控（Linux支持）
尝试 {
    变量 cpu温度 = 获取CPU温度()
    打印("CPU温度: " + cpu温度 + "°C")
} 捕获 错误 {
    打印("当前平台不支持CPU温度读取")
}

// GPU信息查询
变量 gpu信息 = 获取GPU信息()
打印("GPU名称: " + gpu信息.名称)
打印("GPU显存: " + gpu信息.显存 + "MB")

// GPU使用率监控（需要NVIDIA GPU和驱动）
尝试 {
    变量 gpu使用率 = 获取GPU使用率()
    打印("GPU使用率: " + gpu使用率 + "%")
} 捕获 错误 {
    打印("无法获取GPU使用率")
}

// GPU显存信息
尝试 {
    变量 gpu内存 = 获取GPU内存()
    打印("显存总量: " + gpu内存.总量 + "MB")
    打印("显存已用: " + gpu内存.已用 + "MB")
    打印("显存空闲: " + gpu内存.空闲 + "MB")
} 捕获 错误 {
    打印("无法获取GPU内存信息")
}

// 内存信息查询
变量 内存信息 = 获取内存信息()
打印("内存总量: " + 内存信息.总量 + "MB")
打印("内存已用: " + 内存信息.已用 + "MB")
打印("内存空闲: " + 内存信息.空闲 + "MB")

// 内存使用率
变量 内存使用率 = 获取内存使用率()
打印("内存使用率: " + 内存使用率 + "%")

// 设置CPU频率（需要管理员权限）
尝试 {
    设置CPU频率(2400)  // 设置为2400MHz
    打印("CPU频率设置成功")
} 捕获 错误 {
    打印("设置CPU频率失败: " + 错误)
}

// 清理内存（需要管理员权限）
尝试 {
    清理内存()
    打印("内存清理成功")
} 捕获 错误 {
    打印("清理内存失败: " + 错误)
}
```

### 9.5 平台支持

| 功能 | Windows | macOS | Linux | 说明 |
|------|---------|-------|-------|------|
| CPU信息 | ✅ | ✅ | ✅ | 使用sysinfo库 |
| CPU使用率 | ✅ | ✅ | ✅ | 使用sysinfo库 |
| CPU温度 | ⚠️ | ⚠️ | ✅ | Linux通过/sys/class/thermal |
| CPU频率 | ✅ | ✅ | ✅ | 使用sysinfo库 |
| 设置CPU频率 | ⚠️ | ❌ | ⚠️ | 需要系统级权限 |
| GPU信息 | ✅ | ⚠️ | ✅ | NVIDIA需要nvml-wrapper |
| GPU使用率 | ✅ | ⚠️ | ✅ | NVIDIA需要nvml-wrapper |
| GPU温度 | ✅ | ⚠️ | ✅ | NVIDIA需要nvml-wrapper |
| GPU显存 | ✅ | ⚠️ | ✅ | NVIDIA需要nvml-wrapper |
| 设置GPU频率 | ⚠️ | ❌ | ⚠️ | 需要管理员权限和驱动支持 |
| 内存信息 | ✅ | ✅ | ✅ | 使用sysinfo库 |
| 内存使用率 | ✅ | ✅ | ✅ | 使用sysinfo库 |
| 清理内存 | ⚠️ | ⚠️ | ⚠️ | 需要系统级权限 |

**图例**:
- ✅ 完全支持
- ⚠️ 部分支持或需要特殊权限
- ❌ 不支持

### 9.6 依赖配置

硬件控制功能使用可选依赖，需要在编译时启用相应的feature：

```toml
# Cargo.toml
[dependencies]
num_cpus = "1.16"  # CPU核心数检测（默认启用）

[dependencies.sysinfo]
version = "0.30"
optional = true

[dependencies.nvml-wrapper]
version = "0.10"
optional = true

[features]
# 系统信息获取（硬件控制）
sysinfo = ["dep:sysinfo"]
# NVIDIA GPU控制
nvml = ["dep:nvml-wrapper"]
```

编译时启用硬件控制功能：

```bash
# 启用系统信息获取（CPU、内存）
cargo build --features sysinfo

# 启用NVIDIA GPU控制
cargo build --features nvml

# 启用所有硬件控制功能
cargo build --features sysinfo,nvml
```

### 9.7 安全注意事项

⚠️ **警告**: 硬件控制API涉及底层系统操作，不当使用可能导致：
- 系统不稳定
- 硬件损坏
- 数据丢失

**安全建议**:
1. 默认使用用户级权限（只读查询）
2. 设置频率和功耗限制前，确保了解硬件规格
3. 在生产环境中谨慎使用管理员级操作
4. 监控硬件温度，避免过热
5. 定期备份重要数据

### 9.8 桌面端模拟模式

在没有硬件访问权限或特定硬件的环境中，硬件控制API会返回模拟数据或错误：

```rust
// 创建用户级权限管理器（默认）
let manager = HardwareControlManager::new(PermissionLevel::User);

// 查询操作总是可用（返回实际数据或模拟数据）
// - CPU信息返回实际检测到的信息
// - GPU信息在没有GPU时返回"Unknown GPU"
// - 内存信息返回实际系统内存

// 控制操作需要管理员权限
// - 设置频率、功耗限制等操作会返回权限错误
```

这允许在开发环境安全测试硬件控制代码逻辑。
