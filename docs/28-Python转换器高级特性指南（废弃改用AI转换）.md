# Python转换器高级特性使用指南

## 📖 概述

本指南介绍YuCheng到Python转换器支持的高级特性，包括方法调用映射、错误处理、异步编程和复杂类型。

---

## 🔧 方法调用映射

### 函数风格映射

某些方法会被转换为Python的内置函数调用：

```yucheng
// YuCheng代码
var 文本 = "Hello"
var 长度 = 文本.长度()
var 类型 = 文本.类型()
```

转换为Python:
```python
文本 = "Hello"
长度 = len(文本)
类型 = type(文本)
```

**支持的函数风格方法**:
- `.长度()` → `len(...)`
- `.类型()` → `type(...)`

### 方法风格映射

大多数方法保持方法调用形式：

```yucheng
// YuCheng代码
var 文本 = "hello,world"
var 部分 = 文本.分割(",")
var 大写 = 文本.大写()

var 列表 = [1, 2, 3]
列表.添加(4)
```

转换为Python:
```python
文本 = "hello,world"
部分 = 文本.split(",")
大写 = 文本.upper()

列表 = [1, 2, 3]
列表.append(4)
```

**支持的方法风格映射**:

| YuCheng | Python | 用途 |
|---------|--------|------|
| `.分割()` | `.split()` | 字符串分割 |
| `.连接()` | `.join()` | 字符串连接 |
| `.替换()` | `.replace()` | 字符串替换 |
| `.大写()` | `.upper()` | 转大写 |
| `.小写()` | `.lower()` | 转小写 |
| `.去空格()` | `.strip()` | 去除空格 |
| `.添加()` | `.append()` | 列表添加 |
| `.弹出()` | `.pop()` | 列表弹出 |
| `.插入()` | `.insert()` | 列表插入 |
| `.删除()` | `.remove()` | 列表删除 |
| `.排序()` | `.sort()` | 列表排序 |
| `.反转()` | `.reverse()` | 列表反转 |
| `.开始于()` | `.startswith()` | 字符串开始 |
| `.结束于()` | `.endswith()` | 字符串结束 |
| `.查找()` | `.find()` | 字符串查找 |
| `.包含()` | `.count()` | 计数 |

---

## 🛡️ 错误处理

### 基本try-catch

```yucheng
// YuCheng代码
函数 安全除法(a: 整数, b: 整数) -> 整数 {
    尝试 {
        返回 a / b
    } 捕获 错误 {
        打印("除法错误")
        返回 0
    }
}
```

转换为Python:
```python
def 安全除法(a: int, b: int) -> int:
    try:
        return (a / b)
    except Exception as 错误:
        print("除法错误")
        return 0
```

### 完整try-catch-finally

```yucheng
// YuCheng代码
函数 读取文件(路径: 文本) -> 文本 {
    尝试 {
        var 内容 = 读文件(路径)
        返回 内容
    } 捕获 错误 {
        打印("读取失败: " + 错误)
        返回 ""
    } 最终 {
        打印("清理资源")
    }
}
```

转换为Python:
```python
def 读取文件(路径: str) -> str:
    try:
        内容 = open(路径)
        return 内容
    except Exception as 错误:
        print(("读取失败: " + 错误))
        return ""
    finally:
        print("清理资源")
```

### 抛出异常

```yucheng
// YuCheng代码
函数 验证年龄(年龄: 整数) {
    如果 年龄 < 0 {
        抛出 错误("年龄不能为负数")
    }
}
```

转换为Python:
```python
def 验证年龄(年龄: int):
    if (年龄 < 0):
        raise 错误("年龄不能为负数")
```

---

## ⚡ 异步编程

### 异步函数

```yucheng
// YuCheng代码
异步 函数 获取数据(url: 文本) -> 文本 {
    var 响应 = 等待 HTTP_GET(url)
    返回 响应
}
```

转换为Python:
```python
async def 获取数据(url: str) -> str:
    响应 = await HTTP_GET(url)
    return 响应
```

### await表达式

```yucheng
// YuCheng代码
异步 函数 处理多个请求() {
    var 结果1 = 等待 获取数据("url1")
    var 结果2 = 等待 获取数据("url2")
    打印(结果1 + 结果2)
}
```

转换为Python:
```python
async def 处理多个请求():
    结果1 = await 获取数据("url1")
    结果2 = await 获取数据("url2")
    print((结果1 + 结果2))
```

---

## 🎨 复杂类型

### 元组类型

```yucheng
// YuCheng代码
函数 获取坐标() -> 元组<整数, 整数> {
    返回 [10, 20]
}
```

转换为Python:
```python
def 获取坐标() -> Tuple[int, int]:
    return [10, 20]
```

### 联合类型

```yucheng
// YuCheng代码
函数 处理值(值: 联合<整数, 文本>) -> 文本 {
    返回 转文本(值)
}
```

转换为Python:
```python
def 处理值(值: Union[int, str]) -> str:
    return str(值)
```

### 可选类型

```yucheng
// YuCheng代码
函数 查找用户(id: 整数) -> 可选<文本> {
    如果 id > 0 {
        返回 "用户" + 转文本(id)
    }
    返回 空
}
```

转换为Python:
```python
def 查找用户(id: int) -> Optional[str]:
    if (id > 0):
        return ("用户" + str(id))
    return None
```

### 集合类型

```yucheng
// YuCheng代码
函数 去重(列表: 列表<整数>) -> 集合<整数> {
    var 结果: 集合<整数> = 集合()
    // ... 实现
    返回 结果
}
```

转换为Python:
```python
def 去重(列表: List[int]) -> Set[int]:
    结果: Set[int] = 集合()
    # ... 实现
    return 结果
```

### 嵌套泛型

```yucheng
// YuCheng代码
函数 复杂数据() -> 字典<文本, 列表<可选<整数>>> {
    返回 {
        "数据": [1, 2, 空, 3]
    }
}
```

转换为Python:
```python
def 复杂数据() -> Dict[str, List[Optional[int]]]:
    return {"数据": [1, 2, None, 3]}
```

---

## 📚 完整示例

### 示例1: 异步HTTP客户端

```yucheng
// YuCheng代码
异步 函数 获取用户信息(用户id: 整数) -> 可选<文本> {
    尝试 {
        var url = "https://api.example.com/users/" + 转文本(用户id)
        var 响应 = 等待 HTTP_GET(url)
        var 数据 = JSON解析(响应)
        返回 数据.名称
    } 捕获 错误 {
        打印("获取用户信息失败: " + 错误)
        返回 空
    }
}
```

转换为Python:
```python
async def 获取用户信息(用户id: int) -> Optional[str]:
    try:
        url = ("https://api.example.com/users/" + str(用户id))
        响应 = await HTTP_GET(url)
        数据 = JSON解析(响应)
        return 数据.名称
    except Exception as 错误:
        print(("获取用户信息失败: " + 错误))
        return None
```

### 示例2: 文本处理工具

```yucheng
// YuCheng代码
函数 处理文本(文本: 文本) -> 列表<文本> {
    var 清理后 = 文本.去空格()
    var 小写 = 清理后.小写()
    var 单词 = 小写.分割(" ")
    
    var 结果: 列表<文本> = []
    对于 单词 在 单词 {
        如果 单词.长度() > 3 {
            结果.添加(单词)
        }
    }
    
    返回 结果
}
```

转换为Python:
```python
def 处理文本(文本: str) -> List[str]:
    清理后 = 文本.strip()
    小写 = 清理后.lower()
    单词 = 小写.split(" ")
    
    结果: List[str] = []
    for 单词 in 单词:
        if (len(单词) > 3):
            结果.append(单词)
    
    return 结果
```

---

## ⚠️ 注意事项

### 当前限制

1. **错误传播运算符 `?`**
   - YuCheng: `var 结果 = 可能失败()?`
   - Python: 生成注释 `# TODO: 错误传播`
   - 原因: Python没有直接等价物

2. **列表推导式**
   - 当前: 使用for循环
   - 未来: 将优化为列表推导式

3. **装饰器**
   - 当前: 注解未映射
   - 未来: 将支持装饰器转换

### 最佳实践

1. **使用类型注解**
   ```yucheng
   // 推荐: 明确类型
   函数 处理(值: 整数) -> 文本 { ... }
   
   // 不推荐: 缺少类型
   函数 处理(值) { ... }
   ```

2. **合理使用异步**
   ```yucheng
   // 推荐: I/O操作使用异步
   异步 函数 读取文件(路径: 文本) { ... }
   
   // 不推荐: 计算密集型使用异步
   异步 函数 计算斐波那契(n: 整数) { ... }
   ```

3. **错误处理**
   ```yucheng
   // 推荐: 具体的错误处理
   尝试 {
       ...
   } 捕获 错误: 文件错误 {
       // 处理文件错误
   }
   
   // 可接受: 通用错误处理
   尝试 {
       ...
   } 捕获 错误 {
       // 处理所有错误
   }
   ```

---

## 🔍 调试技巧

### 查看生成的Python代码

```bash
# 转换单个文件
yucheng convert --to python input.ycs -o output.py

# 查看生成的代码
cat output.py
```

### 测试生成的代码

```bash
# 运行生成的Python代码
python output.py

# 使用mypy检查类型
mypy output.py
```

### 常见问题

**Q: 为什么方法调用没有被映射？**
A: 检查方法名是否在支持列表中。如果不在，将保持原样。

**Q: 异步函数如何运行？**
A: 需要使用asyncio:
```python
import asyncio
asyncio.run(你的异步函数())
```

**Q: 类型注解不正确怎么办？**
A: 确保YuCheng代码中有正确的类型声明。

---

## 📖 参考资料

- [YuCheng语言规范](./03-语法详解.md)
- [Python类型注解](https://docs.python.org/3/library/typing.html)
- [Python异步编程](https://docs.python.org/3/library/asyncio.html)
- [转换器实现报告](../.project-reports/implementation-status/28-PYTHON-ADVANCED-FEATURES-COMPLETION.md)

---

**文档版本**: 1.0  
**最后更新**: 2026-02-08
