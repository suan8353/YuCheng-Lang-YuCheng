# 系统 API 详解

## 概述

系统 API 模块 (`system_api.rs`) 提供跨平台的系统级功能访问，包括文件系统、进程管理、环境变量、系统信息等。

## 架构设计

```rust
/// 系统 API 管理器
pub struct SystemApi {
    /// 文件系统操作
    pub fs: FileSystemApi,
    /// 进程管理
    pub process: ProcessApi,
    /// 环境变量
    pub env: EnvironmentApi,
    /// 系统信息
    pub info: SystemInfoApi,
    /// 网络操作
    pub network: NetworkApi,
}
```

## 文件系统 API

### 数据结构

```rust
/// 文件信息
pub struct FileInfo {
    pub path: PathBuf,
    pub size: u64,
    pub is_dir: bool,
    pub is_file: bool,
    pub is_symlink: bool,
    pub created: Option<SystemTime>,
    pub modified: Option<SystemTime>,
    pub accessed: Option<SystemTime>,
    pub permissions: FilePermissions,
}

/// 文件权限
pub struct FilePermissions {
    pub readonly: bool,
    pub mode: u32,  // Unix 权限模式
}
```

### 文件操作

```rust
impl FileSystemApi {
    /// 读取文件内容
    pub fn read_file(&self, path: &str) -> Result<String, SystemError> {
        let content = std::fs::read_to_string(path)?;
        Ok(content)
    }
    
    /// 读取二进制文件
    pub fn read_binary(&self, path: &str) -> Result<Vec<u8>, SystemError> {
        let data = std::fs::read(path)?;
        Ok(data)
    }
    
    /// 写入文件
    pub fn write_file(&self, path: &str, content: &str) -> Result<(), SystemError> {
        std::fs::write(path, content)?;
        Ok(())
    }
    
    /// 追加内容
    pub fn append_file(&self, path: &str, content: &str) -> Result<(), SystemError> {
        use std::io::Write;
        let mut file = std::fs::OpenOptions::new()
            .append(true)
            .create(true)
            .open(path)?;
        file.write_all(content.as_bytes())?;
        Ok(())
    }
    
    /// 删除文件
    pub fn delete_file(&self, path: &str) -> Result<(), SystemError> {
        std::fs::remove_file(path)?;
        Ok(())
    }
    
    /// 复制文件
    pub fn copy_file(&self, src: &str, dst: &str) -> Result<u64, SystemError> {
        let bytes = std::fs::copy(src, dst)?;
        Ok(bytes)
    }
    
    /// 移动/重命名文件
    pub fn move_file(&self, src: &str, dst: &str) -> Result<(), SystemError> {
        std::fs::rename(src, dst)?;
        Ok(())
    }
}
```

### 目录操作

```rust
impl FileSystemApi {
    /// 创建目录
    pub fn create_dir(&self, path: &str) -> Result<(), SystemError> {
        std::fs::create_dir(path)?;
        Ok(())
    }
    
    /// 递归创建目录
    pub fn create_dir_all(&self, path: &str) -> Result<(), SystemError> {
        std::fs::create_dir_all(path)?;
        Ok(())
    }
    
    /// 删除空目录
    pub fn remove_dir(&self, path: &str) -> Result<(), SystemError> {
        std::fs::remove_dir(path)?;
        Ok(())
    }
    
    /// 递归删除目录
    pub fn remove_dir_all(&self, path: &str) -> Result<(), SystemError> {
        std::fs::remove_dir_all(path)?;
        Ok(())
    }
    
    /// 列出目录内容
    pub fn list_dir(&self, path: &str) -> Result<Vec<FileInfo>, SystemError> {
        let mut entries = Vec::new();
        for entry in std::fs::read_dir(path)? {
            let entry = entry?;
            let metadata = entry.metadata()?;
            entries.push(FileInfo {
                path: entry.path(),
                size: metadata.len(),
                is_dir: metadata.is_dir(),
                is_file: metadata.is_file(),
                is_symlink: metadata.file_type().is_symlink(),
                created: metadata.created().ok(),
                modified: metadata.modified().ok(),
                accessed: metadata.accessed().ok(),
                permissions: FilePermissions {
                    readonly: metadata.permissions().readonly(),
                    mode: 0,
                },
            });
        }
        Ok(entries)
    }
}
```

### 路径操作

```rust
impl FileSystemApi {
    /// 检查路径是否存在
    pub fn exists(&self, path: &str) -> bool {
        std::path::Path::new(path).exists()
    }
    
    /// 检查是否为文件
    pub fn is_file(&self, path: &str) -> bool {
        std::path::Path::new(path).is_file()
    }
    
    /// 检查是否为目录
    pub fn is_dir(&self, path: &str) -> bool {
        std::path::Path::new(path).is_dir()
    }
    
    /// 获取绝对路径
    pub fn absolute_path(&self, path: &str) -> Result<String, SystemError> {
        let abs = std::fs::canonicalize(path)?;
        Ok(abs.to_string_lossy().to_string())
    }
    
    /// 获取文件名
    pub fn file_name(&self, path: &str) -> Option<String> {
        std::path::Path::new(path)
            .file_name()
            .map(|s| s.to_string_lossy().to_string())
    }
    
    /// 获取扩展名
    pub fn extension(&self, path: &str) -> Option<String> {
        std::path::Path::new(path)
            .extension()
            .map(|s| s.to_string_lossy().to_string())
    }
    
    /// 获取父目录
    pub fn parent(&self, path: &str) -> Option<String> {
        std::path::Path::new(path)
            .parent()
            .map(|p| p.to_string_lossy().to_string())
    }
    
    /// 连接路径
    pub fn join(&self, base: &str, path: &str) -> String {
        std::path::Path::new(base)
            .join(path)
            .to_string_lossy()
            .to_string()
    }
}
```

## 进程 API

### 数据结构

```rust
/// 进程信息
pub struct ProcessInfo {
    pub pid: u32,
    pub name: String,
    pub status: ProcessStatus,
    pub cpu_usage: f32,
    pub memory_usage: u64,
}

/// 进程状态
pub enum ProcessStatus {
    Running,
    Sleeping,
    Stopped,
    Zombie,
    Unknown,
}

/// 命令执行结果
pub struct CommandResult {
    pub exit_code: i32,
    pub stdout: String,
    pub stderr: String,
    pub success: bool,
}
```

### 进程操作

```rust
impl ProcessApi {
    /// 获取当前进程 ID
    pub fn current_pid(&self) -> u32 {
        std::process::id()
    }
    
    /// 执行命令
    pub fn exec(&self, command: &str) -> Result<CommandResult, SystemError> {
        let output = if cfg!(windows) {
            std::process::Command::new("cmd")
                .args(["/C", command])
                .output()?
        } else {
            std::process::Command::new("sh")
                .args(["-c", command])
                .output()?
        };
        
        Ok(CommandResult {
            exit_code: output.status.code().unwrap_or(-1),
            stdout: String::from_utf8_lossy(&output.stdout).to_string(),
            stderr: String::from_utf8_lossy(&output.stderr).to_string(),
            success: output.status.success(),
        })
    }
    
    /// 执行命令并获取输出
    pub fn exec_output(&self, command: &str) -> Result<String, SystemError> {
        let result = self.exec(command)?;
        if result.success {
            Ok(result.stdout)
        } else {
            Err(SystemError::CommandFailed(result.stderr))
        }
    }
    
    /// 启动后台进程
    pub fn spawn(&self, command: &str, args: &[&str]) -> Result<u32, SystemError> {
        let child = std::process::Command::new(command)
            .args(args)
            .spawn()?;
        Ok(child.id())
    }
    
    /// 获取命令行参数
    pub fn args(&self) -> Vec<String> {
        std::env::args().collect()
    }
}
```

## 环境变量 API

```rust
impl EnvironmentApi {
    /// 获取环境变量
    pub fn get(&self, name: &str) -> Option<String> {
        std::env::var(name).ok()
    }
    
    /// 设置环境变量
    pub fn set(&self, name: &str, value: &str) {
        std::env::set_var(name, value);
    }
    
    /// 删除环境变量
    pub fn remove(&self, name: &str) {
        std::env::remove_var(name);
    }
    
    /// 获取所有环境变量
    pub fn all(&self) -> HashMap<String, String> {
        std::env::vars().collect()
    }
    
    /// 获取当前工作目录
    pub fn current_dir(&self) -> Result<String, SystemError> {
        let path = std::env::current_dir()?;
        Ok(path.to_string_lossy().to_string())
    }
    
    /// 设置当前工作目录
    pub fn set_current_dir(&self, path: &str) -> Result<(), SystemError> {
        std::env::set_current_dir(path)?;
        Ok(())
    }
    
    /// 获取用户主目录
    pub fn home_dir(&self) -> Option<String> {
        dirs::home_dir().map(|p| p.to_string_lossy().to_string())
    }
    
    /// 获取临时目录
    pub fn temp_dir(&self) -> String {
        std::env::temp_dir().to_string_lossy().to_string()
    }
}
```

## 系统信息 API

```rust
/// 系统信息
pub struct SystemInfo {
    pub os_name: String,
    pub os_version: String,
    pub arch: String,
    pub hostname: String,
    pub username: String,
    pub cpu_count: usize,
    pub total_memory: u64,
    pub available_memory: u64,
}

impl SystemInfoApi {
    /// 获取操作系统名称
    pub fn os_name(&self) -> &'static str {
        if cfg!(windows) {
            "windows"
        } else if cfg!(target_os = "macos") {
            "macos"
        } else if cfg!(target_os = "linux") {
            "linux"
        } else {
            "unknown"
        }
    }
    
    /// 获取 CPU 架构
    pub fn arch(&self) -> &'static str {
        if cfg!(target_arch = "x86_64") {
            "x86_64"
        } else if cfg!(target_arch = "aarch64") {
            "aarch64"
        } else if cfg!(target_arch = "x86") {
            "x86"
        } else {
            "unknown"
        }
    }
    
    /// 获取主机名
    pub fn hostname(&self) -> String {
        hostname::get()
            .map(|h| h.to_string_lossy().to_string())
            .unwrap_or_else(|_| "unknown".to_string())
    }
    
    /// 获取当前用户名
    pub fn username(&self) -> String {
        whoami::username()
    }
    
    /// 获取 CPU 核心数
    pub fn cpu_count(&self) -> usize {
        num_cpus::get()
    }
    
    /// 获取完整系统信息
    pub fn full_info(&self) -> SystemInfo {
        SystemInfo {
            os_name: self.os_name().to_string(),
            os_version: "".to_string(), // 需要平台特定实现
            arch: self.arch().to_string(),
            hostname: self.hostname(),
            username: self.username(),
            cpu_count: self.cpu_count(),
            total_memory: 0,
            available_memory: 0,
        }
    }
}
```

## 时间 API

```rust
impl TimeApi {
    /// 获取当前时间戳（毫秒）
    pub fn timestamp_ms(&self) -> u64 {
        use std::time::{SystemTime, UNIX_EPOCH};
        SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_millis() as u64
    }
    
    /// 获取当前时间戳（秒）
    pub fn timestamp(&self) -> u64 {
        use std::time::{SystemTime, UNIX_EPOCH};
        SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_secs()
    }
    
    /// 获取当前日期时间字符串
    pub fn datetime(&self) -> String {
        chrono::Local::now().format("%Y-%m-%d %H:%M:%S").to_string()
    }
    
    /// 获取 ISO 8601 格式时间
    pub fn datetime_iso(&self) -> String {
        chrono::Utc::now().to_rfc3339()
    }
    
    /// 休眠
    pub fn sleep(&self, millis: u64) {
        std::thread::sleep(std::time::Duration::from_millis(millis));
    }
}
```

## 错误类型

```rust
/// 系统 API 错误
#[derive(Debug)]
pub enum SystemError {
    /// IO 错误
    IoError(std::io::Error),
    /// 文件未找到
    FileNotFound(String),
    /// 权限不足
    PermissionDenied(String),
    /// 命令执行失败
    CommandFailed(String),
    /// 环境变量未设置
    EnvNotSet(String),
    /// 其他错误
    Other(String),
}

impl From<std::io::Error> for SystemError {
    fn from(err: std::io::Error) -> Self {
        match err.kind() {
            std::io::ErrorKind::NotFound => {
                SystemError::FileNotFound(err.to_string())
            }
            std::io::ErrorKind::PermissionDenied => {
                SystemError::PermissionDenied(err.to_string())
            }
            _ => SystemError::IoError(err),
        }
    }
}
```

## 使用示例

### 文件操作

```yucheng
副作用 func 处理文件() {
    // 读取文件
    var 内容 = 读取文件("data.txt")
    匹配 内容 {
        成功(文本) => 打印(文本),
        失败(错误) => 打印("读取失败:", 错误),
    }
    
    // 写入文件
    写入文件("output.txt", "Hello, World!")
    
    // 列出目录
    var 文件列表 = 列出目录("./")
    循环 文件 在 文件列表 {
        打印(文件)
    }
}
```

### 执行命令

```yucheng
副作用 func 运行命令() {
    // 执行命令并获取输出
    var 输出 = 执行命令获取输出("echo Hello")
    打印(输出)
    
    // 获取系统信息
    var 系统 = 获取操作系统()
    var 架构 = 获取CPU架构()
    打印("系统:", 系统, "架构:", 架构)
}
```

### 环境变量

```yucheng
副作用 func 环境操作() {
    // 获取环境变量
    var 路径 = 获取环境变量("PATH")
    如果 路径 != 空 {
        打印("PATH:", 路径)
    }
    
    // 设置环境变量
    设置环境变量("MY_VAR", "my_value")
    
    // 获取目录
    var 主目录 = 获取主目录()
    var 临时目录 = 获取临时目录()
    打印("主目录:", 主目录)
    打印("临时目录:", 临时目录)
}
```
