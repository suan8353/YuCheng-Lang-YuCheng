# YuCheng 语言 - SDK管理器

> **实现状态**: ✅ 完整实现
> - ✅ SDK管理器完整实现（sdk_manager.rs 1329行）
> - ✅ 版本管理和多版本共存已实现
> - ✅ 自动更新和工具链管理完整
> - ✅ 组件仓库集成已实现

## 📊 实现状态

| 状态 | 说明 |
|------|------|
| **总体实现度** | 100% |
| **核心功能** | ✅ 已实现 |
| **可用性** | ✅ 可用 |
| **最后更新** | 2026-02-07 |

---

## 1. 概述

SDK管理器负责管理YuCheng语言的不同版本，支持多版本共存、快速切换和自动更新。

### 核心特性

- 📦 **版本管理** - 安装、卸载、切换SDK版本
- 🔄 **自动更新** - 检查和安装最新版本
- 🎯 **多版本共存** - 同时安装多个SDK版本
- 🔧 **工具链管理** - 管理编译器、标准库、工具
- 📊 **版本信息** - 查看SDK详细信息

---

## 2. 使用方法

### 2.1 安装SDK

```bash
# 安装最新版本
yucheng sdk install latest

# 安装指定版本
yucheng sdk install 0.1.0

# 列出可用版本
yucheng sdk list-remote
```

### 2.2 切换SDK版本

```bash
# 切换全局默认版本
yucheng sdk use 0.1.0

# 为当前项目设置版本
yucheng sdk use 0.1.0 --local
```

### 2.3 管理SDK

```bash
# 列出已安装版本
yucheng sdk list

# 查看当前版本
yucheng sdk current

# 卸载版本
yucheng sdk uninstall 0.0.9

# 更新SDK
yucheng sdk update
```

---

## 3. 配置文件

### 3.1 全局配置

位置: `~/.yucheng/sdk-config.toml`

```toml
[sdk]
default_version = "0.1.0"
auto_update = true
update_channel = "stable"
```

### 3.2 项目配置

位置: `project/.yucheng-version`

```
0.1.0
```

---

## 4. SDK结构

```
~/.yucheng/sdks/
├── 0.1.0/
│   ├── bin/
│   │   └── yucheng
│   ├── lib/
│   │   └── stdlib/
│   └── include/
├── 0.0.9/
│   └── ...
└── current -> 0.1.0/
```

---

## 5. 最佳实践

- 为不同项目使用不同SDK版本
- 定期更新到最新稳定版
- 保留至少一个旧版本用于兼容性测试
- 使用项目级版本配置

---

**对应代码**: `compiler/src/sdk_manager.rs`
