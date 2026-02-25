# 移动端 API

御程语言提供完整的移动端 API 模块，支持 Android 和 iOS 平台的系统功能调用，包括 GPS 定位、通知、振动、传感器、相机、生物识别等。

## 模块架构

```
mobile_api.rs
├── 平台检测
│   ├── MobilePlatform         # 平台类型
│   └── detect()               # 平台检测
├── 权限管理
│   ├── MobilePermission       # 权限类型
│   └── 权限字符串映射          # Android/iOS 权限
├── 数据结构
│   ├── GpsLocation            # GPS 位置
│   ├── DeviceInfo             # 设备信息
│   ├── SensorData             # 传感器数据
│   └── NetworkStatus          # 网络状态
├── 回调接口
│   ├── MobileApiCallback      # 回调 trait
│   └── MockMobileCallback     # 模拟实现
├── API 管理器
│   └── MobileApiManager       # 主管理器
└── 平台桥接
    ├── android::              # JNI 桥接
    └── ios::                  # C FFI 桥接
```

## 平台检测

### MobilePlatform

```rust
pub enum MobilePlatform {
    Android,    // Android 平台
    IOS,        // iOS 平台
    NotMobile,  // 非移动平台（桌面/服务器）
}

impl MobilePlatform {
    pub fn detect() -> Self;           // 检测当前平台
    pub fn name(&self) -> &'static str;         // 英文名称
    pub fn chinese_name(&self) -> &'static str; // 中文名称
}
```

### 平台检测示例

```rust
let platform = MobilePlatform::detect();
match platform {
    MobilePlatform::Android => println!("运行在安卓平台"),
    MobilePlatform::IOS => println!("运行在苹果平台"),
    MobilePlatform::NotMobile => println!("运行在桌面平台"),
}
```

## 权限管理

### MobilePermission

```rust
pub enum MobilePermission {
    Camera,              // 相机
    Microphone,          // 麦克风
    LocationFine,        // 精确位置
    LocationCoarse,      // 粗略位置
    StorageRead,         // 存储读取
    StorageWrite,        // 存储写入
    Contacts,            // 联系人
    Calendar,            // 日历
    Phone,               // 电话
    Sms,                 // 短信
    Notification,        // 通知
    Biometric,           // 生物识别
    Bluetooth,           // 蓝牙
    Nfc,                 // NFC
    Vibrate,             // 振动
    NetworkState,        // 网络状态
    BackgroundExecution, // 后台运行
}
```

### 权限字符串映射

| 权限 | Android | iOS (Info.plist) |
|------|---------|------------------|
| Camera | `android.permission.CAMERA` | `NSCameraUsageDescription` |
| Microphone | `android.permission.RECORD_AUDIO` | `NSMicrophoneUsageDescription` |
| LocationFine | `android.permission.ACCESS_FINE_LOCATION` | `NSLocationWhenInUseUsageDescription` |
| StorageRead | `android.permission.READ_EXTERNAL_STORAGE` | `NSPhotoLibraryUsageDescription` |
| StorageWrite | `android.permission.WRITE_EXTERNAL_STORAGE` | `NSPhotoLibraryAddUsageDescription` |
| Contacts | `android.permission.READ_CONTACTS` | `NSContactsUsageDescription` |
| Calendar | `android.permission.READ_CALENDAR` | `NSCalendarsUsageDescription` |
| Notification | `android.permission.POST_NOTIFICATIONS` | `NSUserNotificationUsageDescription` |
| Biometric | `android.permission.USE_BIOMETRIC` | `NSFaceIDUsageDescription` |
| Bluetooth | `android.permission.BLUETOOTH` | `NSBluetoothAlwaysUsageDescription` |
| Nfc | `android.permission.NFC` | `NFCReaderUsageDescription` |

### 权限解析

```rust
impl MobilePermission {
    pub fn from_str(s: &str) -> Option<Self>;
    pub fn android_permission(&self) -> &'static str;
    pub fn ios_usage_key(&self) -> &'static str;
}

// 支持中英文解析
MobilePermission::from_str("camera");  // Some(Camera)
MobilePermission::from_str("相机");    // Some(Camera)
MobilePermission::from_str("位置");    // Some(LocationFine)
```

## 数据结构

### GPS 位置

```rust
pub struct GpsLocation {
    pub latitude: f64,           // 纬度
    pub longitude: f64,          // 经度
    pub altitude: Option<f64>,   // 海拔（米）
    pub accuracy: Option<f64>,   // 精度（米）
    pub timestamp: i64,          // 时间戳
}

impl GpsLocation {
    pub fn to_value(&self) -> Value;  // 转换为 Value
}
```

### 设备信息

```rust
pub struct DeviceInfo {
    pub model: String,              // 设备型号
    pub manufacturer: String,       // 制造商
    pub os_version: String,         // 系统版本
    pub sdk_version: Option<i32>,   // SDK 版本（Android）
    pub screen_width: i32,          // 屏幕宽度
    pub screen_height: i32,         // 屏幕高度
    pub screen_density: f32,        // 屏幕密度
    pub battery_level: Option<i32>, // 电池电量（0-100）
    pub is_charging: Option<bool>,  // 是否正在充电
}

impl DeviceInfo {
    pub fn to_value(&self) -> Value;  // 转换为 Value
}
```

### 网络状态

```rust
pub enum NetworkStatus {
    None,      // 无网络
    Wifi,      // WiFi
    Mobile,    // 移动数据
    Ethernet,  // 以太网
    Unknown,   // 未知
}

impl NetworkStatus {
    pub fn to_value(&self) -> Value;  // 转换为 Value
}
```

### 传感器数据

```rust
pub enum SensorType {
    Accelerometer,  // 加速度计
    Gyroscope,      // 陀螺仪
    Magnetometer,   // 磁力计
    Barometer,      // 气压计
    Light,          // 光线传感器
    Proximity,      // 距离传感器
}

pub struct SensorData {
    pub x: f64,
    pub y: f64,
    pub z: f64,
    pub timestamp: i64,
}
```

## 回调接口

### MobileApiCallback

宿主应用需要实现此 trait：

```rust
pub trait MobileApiCallback: Send + Sync {
    // 权限管理
    fn request_permission(&self, permission: MobilePermission) -> bool;
    fn check_permission(&self, permission: MobilePermission) -> bool;
    
    // 位置服务
    fn get_location(&self) -> Option<GpsLocation>;
    
    // 通知
    fn show_notification(&self, title: &str, body: &str) -> bool;
    
    // 振动
    fn vibrate(&self, duration_ms: i64);
    
    // 设备信息
    fn get_device_info(&self) -> DeviceInfo;
    
    // 系统功能
    fn open_url(&self, url: &str) -> bool;
    fn share_text(&self, text: &str) -> bool;
    
    // 剪贴板
    fn copy_to_clipboard(&self, text: &str) -> bool;
    fn read_clipboard(&self) -> Option<String>;
    
    // Toast
    fn show_toast(&self, message: &str, long: bool);
    
    // 网络
    fn get_network_status(&self) -> NetworkStatus;
    
    // 相机
    fn take_photo(&self) -> Option<Vec<u8>>;
    fn pick_image(&self) -> Option<Vec<u8>>;
    
    // 二维码
    fn scan_qrcode(&self) -> Option<String>;
    
    // 生物识别
    fn authenticate_biometric(&self, reason: &str) -> bool;
}
```

## API 管理器

### MobileApiManager

```rust
pub struct MobileApiManager {
    platform: MobilePlatform,
    callback: Option<Box<dyn MobileApiCallback>>,
    granted_permissions: HashSet<MobilePermission>,
    mock_mode: bool,
}

impl MobileApiManager {
    pub fn new() -> Self;                    // 创建管理器
    pub fn mock() -> Self;                   // 创建模拟模式管理器
    pub fn set_callback(&mut self, callback: Box<dyn MobileApiCallback>);
    pub fn platform(&self) -> MobilePlatform;
    pub fn is_mobile(&self) -> bool;
    pub fn is_mock(&self) -> bool;
    
    // 调用 API
    pub fn call(&mut self, name: &str, args: Vec<Value>, span: &Span) -> Result<Value>;
}
```

## 支持的 API

### 权限管理

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| `请求权限` / `request_permission` | 权限名称 | 布尔 | 请求权限 |
| `检查权限` / `check_permission` | 权限名称 | 布尔 | 检查权限 |

### 位置服务

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| `获取位置` / `get_location` | 无 | Map/Null | 获取 GPS 位置 |

返回值结构：
```json
{
    "纬度": 39.9042,
    "经度": 116.4074,
    "海拔": 50.0,
    "精度": 10.0,
    "时间戳": 1234567890
}
```

### 通知

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| `显示通知` / `show_notification` | 标题, 内容 | 布尔 | 显示系统通知 |

### 振动

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| `振动` / `vibrate` | 时长(毫秒) | 空 | 设备振动 |

### 设备信息

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| `获取设备信息` / `get_device_info` | 无 | Map | 获取设备信息 |

返回值结构：
```json
{
    "型号": "Pixel 6",
    "制造商": "Google",
    "系统版本": "Android 13",
    "SDK版本": 33,
    "屏幕宽度": 1080,
    "屏幕高度": 2400,
    "屏幕密度": 2.625,
    "电池电量": 80,
    "正在充电": true
}
```

### 系统功能

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| `打开链接` / `open_url` | URL | 布尔 | 打开外部链接 |
| `分享文本` / `share_text` | 文本 | 布尔 | 分享文本 |

### 剪贴板

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| `复制到剪贴板` / `copy_to_clipboard` | 文本 | 布尔 | 复制到剪贴板 |
| `读取剪贴板` / `read_clipboard` | 无 | 文本/Null | 读取剪贴板 |

### Toast

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| `显示提示` / `show_toast` | 消息, [长时间] | 空 | 显示 Toast |

### 网络

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| `获取网络状态` / `get_network_status` | 无 | 文本 | 获取网络状态 |

返回值：`"无网络"` / `"WiFi"` / `"移动数据"` / `"以太网"` / `"未知"`

### 相机

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| `拍照` / `take_photo` | 无 | Base64/Null | 拍照并返回图片 |
| `选择图片` / `pick_image` | 无 | Base64/Null | 从相册选择图片 |

### 二维码

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| `扫描二维码` / `scan_qrcode` | 无 | 文本/Null | 扫描二维码 |

### 生物识别

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| `生物识别认证` / `authenticate_biometric` | 原因 | 布尔 | 生物识别认证 |

### 平台信息

| API 名称 | 参数 | 返回值 | 说明 |
|----------|------|--------|------|
| `获取平台` / `get_platform` | 无 | 文本 | 获取平台名称 |
| `获取平台中文名` / `get_platform_chinese` | 无 | 文本 | 获取平台中文名 |
| `是移动端` / `is_mobile` | 无 | 布尔 | 是否为移动平台 |

## 模拟模式

用于桌面测试的模拟实现：

```rust
pub struct MockMobileCallback {
    mock_location: GpsLocation,      // 模拟位置（北京）
    mock_device_info: DeviceInfo,    // 模拟设备信息
    clipboard: Mutex<String>,        // 模拟剪贴板
}

impl MockMobileCallback {
    pub fn new() -> Self;
}
```

### 模拟数据

- **位置**: 北京（39.9042, 116.4074）
- **设备**: Mock Device, YuCheng 制造
- **屏幕**: 1080x1920, 密度 2.0
- **电池**: 80%, 充电中
- **网络**: WiFi
- **权限**: 全部授权

## 使用示例

### 基本使用

```rust
let mut manager = MobileApiManager::new();
let span = Span::dummy();

// 获取平台
let platform = manager.call("获取平台", vec![], &span)?;

// 获取设备信息
let device_info = manager.call("获取设备信息", vec![], &span)?;

// 显示通知
manager.call(
    "显示通知",
    vec![
        Value::String("标题".to_string()),
        Value::String("内容".to_string()),
    ],
    &span,
)?;

// 振动
manager.call("振动", vec![Value::Integer(100)], &span)?;
```

### 模拟模式测试

```rust
let mut manager = MobileApiManager::mock();
let span = Span::dummy();

// 测试剪贴板
manager.call(
    "复制到剪贴板",
    vec![Value::String("测试文本".to_string())],
    &span,
)?;

let result = manager.call("读取剪贴板", vec![], &span)?;
// result = Value::String("测试文本")
```

### 权限检查

```rust
// 请求权限
let granted = manager.call(
    "请求权限",
    vec![Value::String("相机".to_string())],
    &span,
)?;

// 检查权限
let has_permission = manager.call(
    "检查权限",
    vec![Value::String("位置".to_string())],
    &span,
)?;
```

## 平台桥接

### Android (JNI)

```rust
#[cfg(target_os = "android")]
pub mod android {
    pub struct AndroidCallback {
        // JNI 环境通过 jni crate 注入
    }
    
    // JNI 导出函数示例
    // #[no_mangle]
    // pub extern "system" fn Java_dev_yucheng_MobileApi_nativeCall(
    //     env: JNIEnv,
    //     _class: JClass,
    //     name: JString,
    //     args: JObject,
    // ) -> jstring { ... }
}
```

### iOS (C FFI)

```rust
#[cfg(target_os = "ios")]
pub mod ios {
    pub struct IosCallback {
        // 回调函数指针
    }
    
    // C FFI 导出函数示例
    // #[no_mangle]
    // pub extern "C" fn yucheng_mobile_call(
    //     name: *const c_char,
    //     args: *const c_char,
    // ) -> *mut c_char { ... }
}
```

## 辅助函数

### Base64 编码

内置 Base64 编码实现，用于图片数据传输：

```rust
fn base64_encode(data: &[u8]) -> String;
```

### 参数检查

```rust
fn check_args(args: &[Value], expected: usize, func_name: &str, span: &Span) -> Result<()>;
fn get_string(value: &Value, param_name: &str, span: &Span) -> Result<String>;
fn get_integer(value: &Value, param_name: &str, span: &Span) -> Result<i64>;
fn get_bool(value: &Value, param_name: &str, span: &Span) -> Result<bool>;
```

## 最佳实践

1. **权限检查**: 调用需要权限的 API 前先检查权限
2. **错误处理**: 处理 API 调用可能返回的 Null 值
3. **平台适配**: 使用 `是移动端` 检查平台，提供降级方案
4. **模拟测试**: 开发阶段使用模拟模式进行测试
5. **资源释放**: 相机等资源使用后及时释放
