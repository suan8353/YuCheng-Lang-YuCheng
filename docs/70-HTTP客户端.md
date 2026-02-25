# HTTP 客户端

御程语言内置完整的 HTTP 客户端模块，提供纯 Rust 实现的网络请求功能，支持 GET/POST/PUT/DELETE 等方法、超时控制、重试机制、重定向跟随等特性。

## 模块架构

```
http_client.rs
├── HttpMethod          # HTTP 方法枚举
├── HttpRequest         # 请求构建器
├── HttpResponse        # 响应对象
├── HttpError           # 错误类型
├── ParsedUrl           # URL 解析器
├── HttpClient          # 客户端主类
└── 全局便捷函数         # get/post/put/delete
```

## 核心数据结构

### HTTP 方法

```rust
pub enum HttpMethod {
    Get,      // GET 请求
    Post,     // POST 请求
    Put,      // PUT 请求
    Delete,   // DELETE 请求
    Patch,    // PATCH 请求
    Head,     // HEAD 请求
    Options,  // OPTIONS 请求
}

impl HttpMethod {
    pub fn as_str(&self) -> &'static str;  // 转换为字符串
}
```

### HTTP 请求

```rust
pub struct HttpRequest {
    pub method: HttpMethod,                    // 请求方法
    pub url: String,                           // 请求 URL
    pub headers: HashMap<String, String>,      // 请求头
    pub body: Option<Vec<u8>>,                 // 请求体
    pub timeout: Duration,                     // 超时时间（默认 30 秒）
}
```

#### 请求构建器方法

| 方法 | 说明 | 示例 |
|------|------|------|
| `new(method, url)` | 创建请求 | `HttpRequest::new(HttpMethod::Get, "http://api.example.com")` |
| `get(url)` | 创建 GET 请求 | `HttpRequest::get("http://api.example.com")` |
| `post(url)` | 创建 POST 请求 | `HttpRequest::post("http://api.example.com")` |
| `put(url)` | 创建 PUT 请求 | `HttpRequest::put("http://api.example.com")` |
| `delete(url)` | 创建 DELETE 请求 | `HttpRequest::delete("http://api.example.com")` |
| `header(key, value)` | 添加请求头 | `.header("X-Custom", "value")` |
| `json_body(json)` | 设置 JSON 请求体 | `.json_body(r#"{"key": "value"}"#)` |
| `form_body(data)` | 设置表单请求体 | `.form_body("name=test&age=20")` |
| `raw_body(data)` | 设置原始请求体 | `.raw_body(vec![1, 2, 3])` |
| `timeout(duration)` | 设置超时 | `.timeout(Duration::from_secs(10))` |
| `auth_bearer(token)` | Bearer 认证 | `.auth_bearer("your-token")` |
| `auth_basic(user, pass)` | Basic 认证 | `.auth_basic("user", "pass")` |

### HTTP 响应

```rust
pub struct HttpResponse {
    pub status_code: u16,                      // 状态码
    pub status_text: String,                   // 状态文本
    pub headers: HashMap<String, String>,      // 响应头
    pub body: Vec<u8>,                         // 响应体
}
```

#### 响应方法

| 方法 | 返回类型 | 说明 |
|------|----------|------|
| `is_success()` | `bool` | 状态码 200-299 |
| `is_redirect()` | `bool` | 状态码 300-399 |
| `is_client_error()` | `bool` | 状态码 400-499 |
| `is_server_error()` | `bool` | 状态码 500+ |
| `text()` | `String` | 响应体转文本 |
| `json<T>()` | `Result<T, HttpError>` | 解析 JSON |
| `header(key)` | `Option<&String>` | 获取响应头 |

### HTTP 错误

```rust
pub enum HttpError {
    ConnectionError(String),   // 连接错误
    Timeout,                   // 请求超时
    DnsError(String),          // DNS 解析失败
    TlsError(String),          // SSL/TLS 错误
    ParseError(String),        // 解析错误
    TooManyRedirects,          // 重定向过多
    InvalidUrl(String),        // 无效 URL
    IoError(String),           // IO 错误
}
```

## HTTP 客户端

### 客户端配置

```rust
pub struct HttpClient {
    default_timeout: Duration,                 // 默认超时（30秒）
    max_redirects: u32,                        // 最大重定向次数（10）
    default_headers: HashMap<String, String>,  // 默认请求头
    follow_redirects: bool,                    // 是否跟随重定向
    retry_count: u32,                          // 重试次数
    retry_delay: Duration,                     // 重试延迟
}
```

### 客户端构建

```rust
let client = HttpClient::new()
    .with_timeout(Duration::from_secs(60))      // 设置超时
    .with_max_redirects(5)                       // 最大重定向
    .with_default_header("User-Agent", "MyApp") // 默认请求头
    .with_retry(3, Duration::from_millis(500))  // 重试配置
    .follow_redirects(true);                     // 跟随重定向
```

### 发送请求

```rust
// 使用客户端发送
let response = client.send(&request)?;

// 便捷方法
let response = client.get("http://api.example.com")?;
let response = client.post("http://api.example.com", r#"{"data": 1}"#)?;
let response = client.put("http://api.example.com", r#"{"data": 2}"#)?;
let response = client.delete("http://api.example.com")?;
let response = client.post_form("http://api.example.com", "key=value")?;
```

## URL 解析

```rust
pub struct ParsedUrl {
    pub scheme: String,        // 协议（http/https）
    pub host: String,          // 主机名
    pub port: u16,             // 端口（http:80, https:443）
    pub path: String,          // 路径
    pub query: Option<String>, // 查询参数
}

// 解析 URL
let url = ParsedUrl::parse("https://api.example.com:8443/v1/users?page=1")?;
// scheme: "https", host: "api.example.com", port: 8443, path: "/v1/users", query: Some("page=1")

// 获取完整路径
let full_path = url.full_path();  // "/v1/users?page=1"
```

## 全局便捷函数

```rust
// 无需创建客户端实例
let response = http_client::get("http://api.example.com")?;
let response = http_client::post("http://api.example.com", r#"{"key": "value"}"#)?;
let response = http_client::put("http://api.example.com", r#"{"key": "value"}"#)?;
let response = http_client::delete("http://api.example.com")?;
```

## HTTPS 支持

对于 HTTPS 请求，模块使用系统命令作为后备方案：

- **Windows**: 使用 PowerShell 的 `Invoke-WebRequest`
- **Unix/Linux/macOS**: 使用 `curl`

这是一个实用的设计决策，因为纯 Rust TLS 实现需要额外的依赖库。

## 认证支持

### Bearer Token 认证

```rust
let request = HttpRequest::get("https://api.example.com/protected")
    .auth_bearer("your-jwt-token");
// 添加 Header: Authorization: Bearer your-jwt-token
```

### Basic 认证

```rust
let request = HttpRequest::get("https://api.example.com/protected")
    .auth_basic("username", "password");
// 添加 Header: Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

内置 Base64 编码实现，无需外部依赖。

## 重试机制

```rust
let client = HttpClient::new()
    .with_retry(3, Duration::from_millis(500));  // 重试 3 次，间隔 500ms

// 只对以下错误类型重试：
// - ConnectionError（连接错误）
// - Timeout（超时）
```

## 重定向处理

```rust
let client = HttpClient::new()
    .with_max_redirects(10)    // 最多跟随 10 次重定向
    .follow_redirects(true);   // 启用重定向跟随

// 支持的重定向类型：
// - 绝对 URL: https://new-domain.com/path
// - 相对路径: /new-path
// - 相对路径（无斜杠）: new-path
```

## 使用示例

### 基本 GET 请求

```rust
use http_client::{HttpClient, HttpRequest};

let client = HttpClient::new();
let response = client.get("http://httpbin.org/get")?;

if response.is_success() {
    println!("响应: {}", response.text());
}
```

### POST JSON 数据

```rust
let request = HttpRequest::post("http://httpbin.org/post")
    .json_body(r#"{"name": "张三", "age": 25}"#)
    .header("X-Request-ID", "12345");

let response = HttpClient::new().send(&request)?;
```

### 带认证的请求

```rust
let request = HttpRequest::get("https://api.github.com/user")
    .auth_bearer("ghp_xxxxxxxxxxxx")
    .timeout(Duration::from_secs(10));

let response = HttpClient::new().send(&request)?;
```

### 表单提交

```rust
let response = HttpClient::new()
    .post_form("http://httpbin.org/post", "username=test&password=123")?;
```

### 错误处理

```rust
match client.get("http://invalid-url") {
    Ok(response) => {
        if response.is_success() {
            println!("成功: {}", response.text());
        } else if response.is_client_error() {
            println!("客户端错误: {}", response.status_code);
        } else if response.is_server_error() {
            println!("服务器错误: {}", response.status_code);
        }
    }
    Err(HttpError::Timeout) => println!("请求超时"),
    Err(HttpError::ConnectionError(msg)) => println!("连接失败: {}", msg),
    Err(HttpError::DnsError(msg)) => println!("DNS 解析失败: {}", msg),
    Err(e) => println!("其他错误: {}", e),
}
```

## 默认请求头

每个请求自动包含以下默认头：

```
User-Agent: YuCheng-HTTP/1.0
Accept: */*
Connection: close
```

可通过 `header()` 方法覆盖或添加自定义头。

## 性能考虑

1. **连接复用**: 当前实现每次请求创建新连接，适合低频请求场景
2. **超时控制**: 支持读写超时，防止请求阻塞
3. **重试策略**: 指数退避可通过自定义 retry_delay 实现
4. **内存效率**: 响应体存储为 `Vec<u8>`，大文件需注意内存占用

## 限制说明

1. HTTP/1.1 协议支持
2. HTTPS 依赖系统命令（PowerShell/curl）
3. 不支持 HTTP/2 和 WebSocket
4. 不支持流式响应
5. 不支持代理配置（可通过系统环境变量）
