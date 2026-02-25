# 附录A：代码级参考

> 本文档记录 YuCheng 编译器源代码层面的所有关键字、AST 节点和契约语法
> 版本：v0.4.0
> 更新日期：2026-02-23
> 代码位置：`compiler/src/keywords.rs`, `compiler/src/ast.rs`, `compiler/src/token.rs`

---

## 目录

1. [关键字定义](#1-关键字定义)
2. [AST 节点类型](#2-ast-节点类型)
3. [Token 类型](#3-token-类型)
4. [契约系统](#4-契约系统)
5. [类型系统](#5-类型系统)

---

## 1. 关键字定义

文件位置：`compiler/src/keywords.rs`

### 1.1 函数分类关键字

| 中文 | 英文 | Token类型 | 代码位置 |
|------|------|-----------|----------|
| 纯函数 | pure | `TokenKind::Pure` | keywords.rs:L35-36 |
| 查询 | query | `TokenKind::Query` | keywords.rs:L38-39 |
| 副作用 | effect | `TokenKind::Effect` | keywords.rs:L41-42 |
| 异步 | async | `TokenKind::Async` | keywords.rs:L44-45 |
| 函数/func | func | `TokenKind::Func` | keywords.rs:L49-51 |

**语法示例**：
```yucheng
纯函数 func 加法(a: 整数, b: 整数) -> 整数 {
    a + b
}

查询 func 获取用户(id: 整数) -> 用户 {
    // ...
}

副作用 func 保存用户(用户: 用户) {
    // ...
}
```

### 1.2 变量声明关键字

| 中文 | 英文 | Token类型 | 代码位置 |
|------|------|-----------|----------|
| 变量 | var | `TokenKind::Var` | keywords.rs:L54-56 |
| 常量 | const | `TokenKind::Const` | keywords.rs:L58-60 |

### 1.3 控制流关键字

| 中文 | 英文 | Token类型 | 代码位置 |
|------|------|-----------|----------|
| 如果 | if | `TokenKind::If` | keywords.rs:L63-64 |
| 否则 | else | `TokenKind::Else` | keywords.rs:L66-67 |
| 否则如果 | elif | `TokenKind::Elif` | keywords.rs:L69-70 |
| 匹配 | match | `TokenKind::Match` | keywords.rs:L72-73 |
| 当 | while | `TokenKind::While` | keywords.rs:L75-76 |
| 循环/对于 | for | `TokenKind::For` | keywords.rs:L78-80 |
| 在/中的 | in | `TokenKind::In` | keywords.rs:L82-84 |
| 从 | from | `TokenKind::From` | keywords.rs:L86-87 |
| 到 | to | `TokenKind::To` | keywords.rs:L89-90 |
| 次 | times | `TokenKind::Times` | keywords.rs:L92-93 |
| 中断 | break | `TokenKind::Break` | keywords.rs:L95-96 |
| 继续 | continue | `TokenKind::Continue` | keywords.rs:L98-99 |
| 返回 | return | `TokenKind::Return` | keywords.rs:L101-102 |

### 1.4 类型关键字

| 中文 | 英文 | Token类型 | 代码位置 |
|------|------|-----------|----------|
| 类型 | type | `TokenKind::Type` | keywords.rs:L105-106 |
| 枚举 | enum | `TokenKind::Enum` | keywords.rs:L108-109 |
| 整数 | int | `TokenKind::Int` | keywords.rs:L111-112 |
| 小数 | float | `TokenKind::Float` | keywords.rs:L114-115 |
| 文本 | string | `TokenKind::String` | keywords.rs:L117-118 |
| 布尔 | bool | `TokenKind::Bool` | keywords.rs:L120-121 |
| 字节 | byte | `TokenKind::Byte` | keywords.rs:L123-124 |
| 空 | null | `TokenKind::Null` | keywords.rs:L126-127 |

### 1.5 布尔值关键字

| 中文 | 英文 | Token类型 | 代码位置 |
|------|------|-----------|----------|
| 真 | true | `TokenKind::True` | keywords.rs:L130-131 |
| 假 | false | `TokenKind::False` | keywords.rs:L133-134 |

### 1.6 契约关键字 (Design by Contract)

| 中文 | 英文 | Token类型 | 代码位置 |
|------|------|-----------|----------|
| 要求 | requires | `TokenKind::Requires` | keywords.rs:L137-138 |
| 保证 | ensures | `TokenKind::Ensures` | keywords.rs:L140-141 |
| 不变量 | invariant | `TokenKind::Invariant` | keywords.rs:L143-144 |
| 断言 | assert | `TokenKind::Assert` | keywords.rs:L146-147 |
| 断言_关键 | assert_critical | `TokenKind::AssertCritical` | keywords.rs:L148-149 |
| 断言_调试 | assert_debug | `TokenKind::AssertDebug` | keywords.rs:L150-151 |
| 断言_始终 | assert_always | `TokenKind::AssertAlways` | keywords.rs:L152-153 |

### 1.7 内存管理关键字

| 中文 | 英文 | Token类型 | 代码位置 |
|------|------|-----------|----------|
| 申请 | alloc | `TokenKind::Alloc` | keywords.rs:L158-159 |
| 释放 | free | `TokenKind::Free` | keywords.rs:L161-162 |
| 持有 | retain | `TokenKind::Retain` | keywords.rs:L164-165 |
| 指针 | pointer | `TokenKind::Pointer` | keywords.rs:L167-168 |
| 手动指针 | manual_ptr | `TokenKind::ManualPtr` | keywords.rs:L170-171 |
| 智能指针 | smart_ptr | `TokenKind::SmartPtr` | keywords.rs:L173-174 |
| 分配 | allocate | `TokenKind::Allocate` | keywords.rs:L176-177 |
| 延迟 | defer | `TokenKind::Defer` | keywords.rs:L197-199 |
| 析构 | destructor | `TokenKind::Destructor` | keywords.rs:L201-202 |

### 1.8 错误处理关键字

| 中文 | 英文 | Token类型 | 代码位置 |
|------|------|-----------|----------|
| 尝试 | try | `TokenKind::Try` | keywords.rs:L205-206 |
| 捕获 | catch | `TokenKind::Catch` | keywords.rs:L208-209 |
| 最终 | finally | `TokenKind::Finally` | keywords.rs:L211-212 |
| 抛出 | throw | `TokenKind::Throw` | keywords.rs:L361-362 |
| 可抛出 | throws | `TokenKind::Throws` | keywords.rs:L364-365 |

### 1.9 并发关键字

| 中文 | 英文 | Token类型 | 代码位置 |
|------|------|-----------|----------|
| 等待 | await | `TokenKind::Await` | keywords.rs:L217-218 |
| 通道 | channel | `TokenKind::Channel` | keywords.rs:L220-221 |
| 并行 | parallel | `TokenKind::Parallel` | keywords.rs:L223-224 |

### 1.10 调度控制关键字

| 中文 | 英文 | Token类型 | 代码位置 |
|------|------|-----------|----------|
| 核心 | core | `TokenKind::Core` | keywords.rs:L227-228 |
| 性能核心 | pcore | `TokenKind::PCore` | keywords.rs:L230-231 |
| 能效核心 | ecore | `TokenKind::ECore` | keywords.rs:L233-234 |
| 亲和性 | affinity | `TokenKind::Affinity` | keywords.rs:L236-237 |
| 优先级 | priority | `TokenKind::Priority` | keywords.rs:L239-240 |
| 任务 | task | `TokenKind::Task` | keywords.rs:L242-243 |
| 线程 | thread | `TokenKind::Thread` | keywords.rs:L245-246 |
| 创建任务 | spawn | `TokenKind::Spawn` | keywords.rs:L251-252 |
| GPU | gpu | `TokenKind::Gpu` | keywords.rs:L257-258 |
| CPU | cpu | `TokenKind::Cpu` | keywords.rs:L260-261 |
| NUMA | numa | `TokenKind::Numa` | keywords.rs:L263-264 |

### 1.11 模块与可见性关键字

| 中文 | 英文 | Token类型 | 代码位置 |
|------|------|-----------|----------|
| 导入 | import | `TokenKind::Import` | keywords.rs:L285-286 |
| 导出 | export | `TokenKind::Export` | keywords.rs:L291-292 |
| 模块 | module | `TokenKind::Module` | keywords.rs:L294-295 |
| 外部 | external | `TokenKind::External` | keywords.rs:L297-298 |
| 公开 | public | `TokenKind::Public` | keywords.rs:L303-304 |
| 私有 | private | `TokenKind::Private` | keywords.rs:L306-307 |

### 1.12 逻辑运算关键字

| 中文 | 英文 | Token类型 | 代码位置 |
|------|------|-----------|----------|
| 且 | and | `TokenKind::And` | keywords.rs:L310-311 |
| 或 | or | `TokenKind::Or` | keywords.rs:L313-314 |
| 非 | not | `TokenKind::Not` | keywords.rs:L316-317 |

### 1.13 面向对象关键字

| 中文 | 英文 | Token类型 | 代码位置 |
|------|------|-----------|----------|
| 继承 | extends | `TokenKind::Extends` | keywords.rs:L320-321 |
| 新建 | new | `TokenKind::New` | keywords.rs:L337-338 |
| 自己 | self | `TokenKind::Self` | keywords.rs:L340-341 |
| 接口 | interface | `TokenKind::Interface` | keywords.rs:L343-344 |
| 实现 | impl | `TokenKind::Impl` | keywords.rs:L346-347 |
| 类 | class | `TokenKind::Class` | keywords.rs:L349-350 |

### 1.14 日志系统关键字

| 中文 | 英文 | Token类型 | 代码位置 |
|------|------|-----------|----------|
| 日志 | log | `TokenKind::Log` | keywords.rs:L368-369 |
| 调试 | debug | `TokenKind::Debug` | keywords.rs:L371-372 |
| 信息 | info | `TokenKind::Info` | keywords.rs:L374-375 |
| 警告 | warn | `TokenKind::Warn` | keywords.rs:L377-378 |
| 错误日志 | error_log | `TokenKind::Error` | keywords.rs:L380-381 |
| 追踪 | trace | `TokenKind::Trace` | keywords.rs:L383-384 |

### 1.15 测试关键字

| 中文 | 英文 | Token类型 | 代码位置 |
|------|------|-----------|----------|
| 测试 | test | `TokenKind::Test` | keywords.rs:L323-324 |
| 边界测试 | boundary_test | `TokenKind::BoundaryTest` | keywords.rs:L387-388 |
| 链路测试 | chain_test | `TokenKind::ChainTest` | keywords.rs:L390-391 |
| 属性测试 | property_test | `TokenKind::PropertyTest` | keywords.rs:L393-394 |
| 纠错测试 | correction_test | `TokenKind::CorrectionTest` | keywords.rs:L396-397 |
| 降级测试 | degradation_test | `TokenKind::DegradationTest` | keywords.rs:L399-400 |

### 1.16 运算符 Token

文件位置：`compiler/src/token.rs`

| Token | 符号 | 代码位置 |
|-------|------|----------|
| Plus | + | token.rs:L333 |
| Minus | - | token.rs:L335 |
| Star | * | token.rs:L337 |
| Power | ** | token.rs:L339 |
| Slash | / | token.rs:L341 |
| Percent | % | token.rs:L343 |
| Equal | == | token.rs:L357 |
| NotEqual | <> | token.rs:L359 |
| Less | < | token.rs:L361 |
| LessEqual | <= | token.rs:L363 |
| Greater | > | token.rs:L365 |
| GreaterEqual | >= | token.rs:L367 |
| Arrow | => | token.rs:L369 |
| ThinArrow | -> | token.rs:L371 |
| LeftArrow | <- | token.rs:L373 |
| Question | ? | token.rs:L375 |
| Dot | . | token.rs:L379 |
| DotDot | .. | token.rs:L381 |
| DotDotDot | ... | token.rs:L383 |
| Pipe | |> | token.rs:L385 |
| Bar | | | token.rs:L387 |
| NullCoalesce | ?? | token.rs:L389 |
| SafeAccess | ?. | token.rs:L391 |

---

## 2. AST 节点类型

文件位置：`compiler/src/ast.rs`

### 2.1 顶层节点

```rust
// ast.rs:L142-146
pub struct Program {
    pub items: Vec<Item>,
    pub span: Span,
}
```

### 2.2 Item 变体 (顶层项)

| 变体 | 结构体 | 代码位置 |
|------|--------|----------|
| Function | `Item::Function(Function)` | ast.rs:L152 |
| Method | `Item::Method(MethodDef)` | ast.rs:L154 |
| TypeDef | `Item::TypeDef(TypeDef)` | ast.rs:L156 |
| InterfaceDef | `Item::Interface(InterfaceDef)` | ast.rs:L158 |
| ImplBlock | `Item::Impl(ImplBlock)` | ast.rs:L160 |
| Variable | `Item::Variable(VariableDecl)` | ast.rs:L168 |
| Module | `Item::Module(Module)` | ast.rs:L170 |
| Import | `Item::Import(Import)` | ast.rs:L186 |
| Export | `Item::Export(Export)` | ast.rs:L188 |

### 2.3 函数相关 AST

```rust
// ast.rs:L209-219
pub enum FunctionKind {
    Pure,       // 纯函数
    Query,     // 查询函数
    Effect,    // 副作用函数
    Async,     // 异步函数
}

// ast.rs:L412-434
pub struct Function {
    pub name: String,
    pub kind: FunctionKind,
    pub params: Vec<Parameter>,
    pub return_type: Option<TypeExpr>,
    pub body: Block,
    pub contracts: Vec<Contract>,
    pub generics: Vec<GenericParam>,
    // ...
}
```

### 2.4 语句 (Stmt) 变体

文件位置：`ast.rs:L723-763`

| 变体 | 说明 |
|------|------|
| `Stmt::VarDecl` | 变量声明 |
| `Stmt::Destructure` | 解构赋值 |
| `Stmt::Expr` | 表达式语句 |
| `Stmt::Return` | 返回语句 |
| `Stmt::If` | if 语句 |
| `Stmt::While` | while 循环 |
| `Stmt::For` | for 循环 |
| `Stmt::Match` | match 语句 |
| `Stmt::Break` | break |
| `Stmt::Continue` | continue |
| `Stmt::Assert` | 断言 |
| `Stmt::Block` | 代码块 |
| `Stmt::Unsafe` | 不安全块 |
| `Stmt::TryCatch` | try-catch |
| `Stmt::Defer` | defer |
| `Stmt::Throw` | throw |

### 2.5 表达式 (Expr) 变体

文件位置：`ast.rs:L880-950`

| 变体 | 说明 | 语法示例 |
|------|------|----------|
| `Expr::Literal` | 字面量 | `123`, `"hello"`, `真` |
| `Expr::Variable` | 变量引用 | `x` |
| `Expr::Binary` | 二元运算 | `a + b` |
| `Expr::Unary` | 一元运算 | `-x`, `非 flag` |
| `Expr::Call` | 函数调用 | `foo(1, 2)` |
| `Expr::Member` | 成员访问 | `obj.field` |
| `Expr::SafeMember` | 安全成员 | `obj?.field` |
| `Expr::Index` | 索引访问 | `arr[0]` |
| `Expr::Lambda` | Lambda | `|x| x + 1` |
| `Expr::Pipe` | 管道 | `x |> foo |> bar` |
| `Expr::Match` | 匹配表达式 | `匹配 x { ... }` |
| `Expr::Select` | 选择表达式 | `选择 { ... }` |
| `Expr::Await` | 等待 | `等待 future` |

---

## 3. Token 类型

文件位置：`compiler/src/token.rs`

### 3.1 字面量 Token

| Token | 说明 | 示例 |
|-------|------|------|
| `TokenKind::Integer` | 整数字面量 | `123` |
| `TokenKind::Float` | 浮点数字面量 | `3.14` |
| `TokenKind::String` | 字符串字面量 | `"hello"` |
| `TokenKind::Identifier` | 标识符 | `my_var` |

### 3.2 特殊标记 Token

| Token | 说明 | 语法 |
|-------|------|------|
| `TokenKind::SectionSpec` | 规格区域 | `#规格` |
| `TokenKind::SectionImpl` | 实现区域 | `#实现` |
| `TokenKind::SectionView` | 视图区域 | `#视图` |
| `TokenKind::SectionTest` | 测试区域 | `#测试` |

---

## 4. 契约系统

文件位置：`compiler/src/ast.rs` 和 `compiler/src/parser.rs`

### 4.1 契约种类 (ContractKind)

```rust
// ast.rs:L454-461
pub enum ContractKind {
    Requires,  // 前置条件 (要求)
    Ensures,   // 后置条件 (保证)
    Invariant, // 不变量
}
```

### 4.2 契约等级 (ContractLevel)

```rust
// ast.rs:L468-483
pub enum ContractLevel {
    Checkable,  // 可验证 (编译期检查)
    Runtime,    // 运行时验证
    Assume,     // 假设 (文档级)
    Unsafe,     // 不安全
}
```

### 4.3 契约关键字注解格式

文件位置：`compiler/src/parser.rs:L812-830`

| 注解 | 契约等级 | 说明 |
|------|----------|------|
| `@checkable` / `@可验证` | `Checkable` | 编译期检查 |
| `@runtime` / `@运行时` | `Runtime` | 运行时检查 |
| `@assume` / `@假设` | `Assume` | 仅文档 |
| `@unsafe` / `@不安全` | `Unsafe` | 不安全 |

### 4.4 契约语法示例

**前置条件 (requires)**：
```yucheng
纯函数 func 除法(a: 整数, b: 整数) -> 整数
    要求 b != 0  // 前置条件
{
    a / b
}
```

**后置条件 (ensures)**：
```yucheng
纯函数 func 绝对值(n: 整数) -> 整数
    保证 结果 >= 0  // 后置条件
{
    如果 n < 0 则 返回 -n 否则 返回 n
}
```

**不变量 (invariant)**：
```yucheng
类 计数器 {
    变量 计数: 整数 = 0
    不变量 计数 >= 0  // 类不变量

    副作用 func 递增() {
        计数 = 计数 + 1
    }
}
```

**带契约等级**：
```yucheng
纯函数 func 除法(a: 整数, b: 整数) -> 整数
    @runtime
    要求 b != 0
    保证 结果 == a / b
{
    a / b
}
```

### 4.5 契约相关特殊变量

| 变量 | 适用范围 | 说明 |
|------|----------|------|
| `参数名` | requires | 函数参数 |
| `结果` | ensures | 函数返回值 |
| `是否异常` | ensures | 是否有异常 |
| `self.字段` | invariant | 实例字段 |

---

## 5. 类型系统

文件位置：`compiler/src/types.rs` 和 `compiler/src/ast.rs`

### 5.1 AST 类型表达式 (TypeExpr)

```rust
// ast.rs:L680-700
pub enum TypeExpr {
    Simple(String, Span),                    // 简单类型名
    Generic(String, Vec<TypeExpr>, Span),   // 泛型类型
    Optional(Box<TypeExpr>, Span),           // 可选类型 T?
    Function(Vec<TypeExpr>, Box<TypeExpr>, Span), // 函数类型 func(T) -> R
    GuardType { base_type, constraint, span }, // 守卫类型
    SelfType(Span),                          // self 类型
}
```

### 5.2 类型种类 (TypeKind)

文件位置：`compiler/src/types.rs:L39-73`

| 类型 | 说明 | 示例 |
|------|------|------|
| `TypeKind::Primitive` | 基础类型 | `整数`, `布尔`, `文本` |
| `TypeKind::Struct` | 结构体 | `用户 { 名字, 年龄 }` |
| `TypeKind::Enum` | 枚举 | `状态 { 成功, 失败 }` |
| `TypeKind::Interface` | 接口 | |
| `TypeKind::Generic` | 泛型实例 | `列表<整数>` |
| `TypeKind::Guarded` | 守卫类型 | `整数 { n > 0 }` |
| `TypeKind::Union` | 联合类型 | |
| `TypeKind::Function` | 函数类型 | `func(整数) -> 文本` |
| `TypeKind::Optional` | 可选类型 | `整数?` |
| `TypeKind::List` | 列表类型 | `[1, 2, 3]` |
| `TypeKind::Map` | 映射类型 | `{k: v}` |
| `TypeKind::Tuple` | 元组类型 | `(1, "a")` |
| `TypeKind::TypeVariable` | 类型参数 | `<T>` |
| `TypeKind::TraitObject` | trait 对象 | 运行时多态 |

---

## 6. 与成熟编程语言的特性差距

### 6.1 核心语法对比（已完整）

经过代码验证，YuCheng 在以下核心语法方面**已经非常成熟**，与主流语言对齐：

| 类别 | YuCheng 现状 | 对比 |
|------|-------------|------|
| **Token 类型** | 100+ Token | ✅ 完整 |
| **表达式类型** | 25+ Expr 变体 | ✅ 完整 |
| **运算符** | 40+ (含幂运算、位运算、约等于) | ✅ 完整 |
| **类型系统** | 15+ TypeKind | ✅ 完整 |
| **? 运算符** | ✅ 已实现 (Expr::TryExpr) | ✅ 对齐 Rust |
| **原子操作** | ✅ 标准库函数 (stdlib_atomic_*) | ✅ 功能完整 |
| **select 表达式** | ✅ 已实现 (条件选择) | ⚠️ 非Go风格通道多路复用 |

### 6.2 真实差距（YuCheng 确实缺少的）

| 特性 | 说明 | 影响 |
|------|------|------|
| **宏系统** | 无 `macro_rules!`、过程宏 | 无法实现 DSL |
| **反射机制** | 无运行时类型自省 | 动态能力受限 |
| **生成器** | 无 `yield` 关键字 | 无惰性流 |
| **包管理** | CLI 未实现 | 包生态不完善 |

### 6.3 与主流语言对比

| 特性 | Rust | Go | Python | Java | YuCheng |
|------|------|-----|--------|------|---------|
| 宏系统 | ✅ | ❌ | ❌ | ❌ | ❌ |
| 反射 | ❌ | ✅ | ✅ | ✅ | ❌ |
| Yield/Generator | ✅ | ❌ | ✅ | ❌ | ❌ |
| ? 运算符 | ✅ | ❌ | ✅ | ✅ | ✅ |
| 原子操作 | ✅ | ✅ | ✅ | ✅ | ✅ |
| 泛型 | ✅ | ❌ | ✅ | ✅ | ✅ |
| 管道符 \|> | ❌ | ❌ | ❌ | ❌ | ✅ 独有 |
| 守卫类型 | ❌ | ❌ | ❌ | ✅ | ✅ 独有 |
| 中英双语 | ❌ | ❌ | ❌ | ❌ | ✅ 独有 |

### 6.4 总结

**YuCheng 核心语法已经非常成熟**，与 Rust/Go/Python/Java 相比：
- ✅ Token/表达式/运算符/类型系统：**完整对齐**
- ✅ ? 运算符、原子操作：**已实现**
- ❌ 宏系统、反射、生成器：**真实差距**

---

## 附录：代码位置索引

| 文件 | 说明 | 行数 |
|------|------|------|
| `compiler/src/keywords.rs` | 关键字定义 | 500+ |
| `compiler/src/token.rs` | Token 类型定义 | 500+ |
| `compiler/src/ast.rs` | AST 节点定义 | 1500+ |
| `compiler/src/types.rs` | 类型系统 | 1600+ |
| `compiler/src/parser.rs` | 语法解析器 | 5500+ |
| `compiler/src/semantic.rs` | 语义分析器 | 6500+ |
| `compiler/src/interpreter.rs` | 解释器 | 3200+ |

---

最后更新：2026-02-23*
