# 简化实现与技术债务清单

本文档汇总编译器后端中所有标注为「简化实现」「暂未实现」「TODO」等的代码位置，便于后续补全或重构。

---

## 一、核心后端（影响正确性与性能）

### interpreter.rs
| 位置 | 说明 |
|------|------|
| 60, 64 | Box/Rc 类型：简化实现，使用 Box<Value>、Rc<RefCell<Value>> |
| 2722 | 尾调用优化：简化实现，仅支持递归调用自己（不支持相互尾调用） |
| 3651 | 缓存：简化实现，完整实现需 Rc<RefCell<>> 共享缓存 |

### bytecode.rs
| 位置 | 说明 |
|------|------|
| 71 | PostconditionFail：预留，暂未使用 |
| 1562 | match 语句编译：简化实现 |
| 1602 | List 模式：简化实现，对 List 值总是匹配 |
| 2628 | 结构体：简化实现，创建空结构体 |
| 2728 | JIT 缓存：简化实现，存储优化后的指令序列 |
| 2881 | 寄存器分配：简化实现，LoadLocal/StoreLocal 时使用寄存器 |
| 3031 | 循环模式检测：简化实现 |
| 3107 | 常量创建：简化实现，直接使用结果值 |
| 4355 | 异常处理：简化实现，直接返回值 |
| 4459 | finally 块：简化实现，直接跳转到 finally_ip |

### llvm_backend.rs
| 位置 | 说明 |
|------|------|
| 320 | 循环优化：简化实现，实际应在编译循环时应用 |
| 803 | 循环限制：简化实现，固定为 10 |
| 896 | 构造器模式：简化实现 |
| 971 | 某优化：简化实现 |
| 1181 | 字段访问：简化实现，假设字段索引为 0 |
| 1436 | 优化：简化实现，实际由 LLVM 处理 |

### wasm_backend.rs
| 位置 | 说明 |
|------|------|
| 934 | 错误值：简化实现，设为 0 |
| 1009 | 字符串：简化实现，假设已预分配 |
| 1266 | 闭包捕获：简化实现，遍历 Variable 节点 |
| 1328 | 闭包标识：简化实现，返回环境指针 |

### compiler.rs
| 位置 | 说明 |
|------|------|
| 697 | LLVM IR 生成器：整节标注为简化实现 |
| 1077 | 类型定义：简化实现，暂不生成 |
| 1750 | for 循环：简化实现，转换为 while |
| 1833-1834 | 约束表达式：简化实现，需更复杂的表达式重写 |
| 1945 | IR 输出：简化实现，返回字节表示 |
| 1956 | WebAssembly 生成器：整节标注为简化实现 |
| 2948 | 函数签名：简化实现 |

---

## 二、语义与优化

### semantic.rs
| 位置 | 说明 |
|------|------|
| 314 | type_cache 清理：简化实现，全量清理 |
| 4140 | is_some 检查：TODO，可发出编译期警告 |
| 4585-4586 | 泛型匹配：简化实现，只匹配返回类型；TODO 参数类型精确匹配 |

### optimizer.rs
| 位置 | 说明 |
|------|------|
| 988, 993, 998 | 多处：简化实现，直接返回 |

---

## 三、审计与安全

### audit_engine.rs
| 位置 | 说明 |
|------|------|
| 613 | MEM001 悬垂指针：简化版 |
| 630 | MEM003 越界访问：简化版 |
| 3680 | 函数复杂度：简化版，检查嵌套深度 |

### audit_engine_advanced.rs
| 位置 | 说明 |
|------|------|
| 182 | 污点分析：简化实现，假设所有参数流向污点汇 |
| 189, 507 | 多处：简化实现 |

---

## 四、调度与硬件

### scheduler.rs
| 位置 | 说明 |
|------|------|
| 631 | Windows 线程亲和性：简化实现 |

---

## 五、AI 与工具链

### ai_conversation.rs
| 位置 | 说明 |
|------|------|
| 245 | 重构：简化实现，只支持改名 |
| 463, 494 | 自然语言解析：简化实现 |

### ai_code_generator.rs
| 位置 | 说明 |
|------|------|
| 791 | 某逻辑：简化实现 |

### ai_assistant.rs
| 位置 | 说明 |
|------|------|
| 418, 826 | 代码生成：TODO 占位 |
| 1440, 1447 | 自然语言转 Spec：简化版 |

### ai_arch_advisor.rs
| 位置 | 说明 |
|------|------|
| 286 | 架构检查：简化版，检查 UI->Service->Database |
| 485 | TODO：AI 增强建议 |

---

## 六、LSP 与 IDE

### lsp.rs
| 位置 | 说明 |
|------|------|
| 1168 | 符号引用查找：简化版，返回所有匹配位置 |

### lsp_enhancements.rs / lsp_enhancements_ast.rs
| 位置 | 说明 |
|------|------|
| 637 | 函数名提取：简化版 |
| 619 | 符号表查询：简化实现 |

---

## 七、标准库与运行时

### stdlib.rs
| 位置 | 说明 |
|------|------|
| 6294-6304 | 锁定/解锁内存页、刷新缓存：暂未实现 |
| 9103, 9122, 9212 | Lambda/函数调用：非解释器场景为简化实现 |
| 13248 | 某状态：简化实现，返回成功 |
| 13430 | 全局调度器配置：简化实现 |
| 13492 | 闭包执行：简化实现 |

---

## 八、其他模块

### refactoring.rs
| 位置 | 说明 |
|------|------|
| 670 | 参数对应：简化实现，假设按顺序 |
| 720-722 | 参数名提取：简化实现 |

### plugin_system.rs
| 位置 | 说明 |
|------|------|
| 422-423 | ZIP 解压：简化实现 |
| 508 | 压缩数据：简化实现，跳过 |

### project_checker.rs
| 位置 | 说明 |
|------|------|
| 520, 540 | 实现文件检查：简化实现，TODO 后续完善 |
| 738 | 验证：因简化实现，只验证能运行 |

### result_stdlib.rs
| 位置 | 说明 |
|------|------|
| 250, 403, 456 | 多处：简化实现 |

### contract_generator.rs / contract_parser.rs
| 位置 | 说明 |
|------|------|
| 多处 | TODO 占位、简化版格式化 |

### layers/application.rs, layers/service.rs
| 位置 | 说明 |
|------|------|
| 254, 392 | 简化实现、返回空列表 |

---

## 统计摘要（精确统计，排除第三方/备份/废弃文件）

### 简化实现 / 简化版

| 目录 | 数量 | 说明 |
|------|------|------|
| compiler/src | 68 | 不含 bin/syntax_debt_scan.rs 的 3 处关键词定义 |
| compiler/tests | 11 | 不含 technical_debt_validation.rs 的 3 处测试断言 |
| stdlib/*.yci | 24 | 协程(7)、JSON(5)、断言(5)、数学(3)、守卫(2)、调试/容器(各1) |
| ide | 6 | lib.rs(4)、UiDesigner.vue(1)、e2e(1) |
| examples | 1 | 13-algorithms-simple.ycs |
| **合计** | **110** | |

### TODO / FIXME（compiler/src 活跃代码）

| 文件 | 数量 |
|------|------|
| lib.rs | 1 |
| semantic.rs | 2 |
| remote_debug.rs | 2 |
| ai_arch_advisor.rs | 1 |
| ai_assistant.rs | 2 |
| ai_pipeline.rs | 1 |
| ai_integration.rs | 2 |
| project_checker.rs | 2 |
| ui_designer.rs | 1 |
| ai_contract_gen.rs | 1 |
| contract_generator.rs | 4 |
| **合计** | **17** |

### 暂未实现 / 暂未使用

| 位置 | 说明 |
|------|------|
| bytecode.rs:71 | PostconditionFail 预留，暂未使用 |
| stdlib.rs:6294-6304 | 锁定/解锁内存页、刷新缓存（3 个函数） |
| pointer_memory_checks.rs | Arena/Bump/Slab 分配器占位测试 |
| **合计** | **4 处** |

### 预留

| 位置 | 说明 |
|------|------|
| bytecode.rs | 4 处（槽位预留等） |
| types.rs | 1 处（内置类型 ID 预留） |
| gpu_scheduler.rs | 2 处（显存预留） |

---

## 总计

| 类别 | 精确数量 |
|------|----------|
| 简化实现/简化版 | **110** |
| TODO/FIXME | **17** |
| 暂未实现 | **4** |
| 预留 | **7** |

---

## 建议优先级

1. **高**：interpreter 尾调用（支持相互尾调用）、bytecode 寄存器分配、LLVM 字段索引
2. **中**：语义泛型匹配、审计引擎检测、WASM 闭包
3. **低**：AI 辅助、LSP 增强、插件 ZIP 等


---

## 2026-02-22 代码对齐增量（与 compiler/src 同步）

以下项目已在代码中确认实现，不应再按“未实现”认定：

1. Contract `invariant` 关键字与语义链路已打通
- 词法/关键字：`compiler/src/token.rs`、`compiler/src/keywords.rs`、`compiler/src/lexer.rs`
- 语法：`compiler/src/parser.rs`（支持 `invariant`、`不变量`、`@invariant`、`@不变量`）
- 语义：`compiler/src/semantic.rs`（恒假不变量报错）
- 分析统计：`compiler/src/analyzer.rs`（`invariants` 计数）

2. Contract 解析失败不再回退为 `true`
- 已由测试覆盖：`compiler/src/parser.rs` 中 `test_contract_parse_failure_should_not_fallback_to_true`

3. 接口方法签名兼容性检查已实现（含参数类型兼容）
- 位置：`compiler/src/semantic.rs`
- 关键函数：`is_param_type_compatible`、`are_method_signatures_compatible`

4. 对应回归测试已存在
- `test_invariant_keyword_tokenization`
- `test_parse_invariant_contract_keyword`
- `test_invariant_always_false_should_fail_semantic`
- `test_invariant_contract_counting`

说明：本次为“增量对齐”，未重跑全量技术债扫描；文档内历史汇总总数（110/17/4/7）先保留，待下一次全量扫描后统一改写。

---

## 2026-02-22 全量重扫统计（最新口径）

统计口径（本节覆盖旧统计口径）：
- 扫描范围：`compiler/src`（排除 `*.backup` 与 `bin/syntax_debt_scan.rs`）、`compiler/tests`、`stdlib/*.yci`、`ide`、`examples`
- `简化实现`：匹配 `简化实现|简化版`
- `TODO/FIXME`：匹配 `TODO|FIXME`（仅 `compiler/src` 活跃代码）
- `暂未实现`：匹配 `暂未实现|暂未使用|未实现`（仅 `compiler/src` 活跃代码）

### 最新计数

| 类别 | 数量 |
|------|------|
| 简化实现（compiler/src） | **60** |
| 简化实现（compiler/tests） | **16** |
| 简化实现（stdlib/*.yci） | **24** |
| 简化实现（ide + examples） | **7** |
| 简化实现总计 | **107** |
| TODO/FIXME（compiler/src） | **31** |
| 暂未实现相关（compiler/src） | **12** |

### 与旧统计对比

| 类别 | 旧值 | 新值 | 变化 |
|------|------|------|------|
| 简化实现总计 | 110 | 107 | -3 |
| TODO/FIXME | 17 | 31 | +14 |
| 暂未实现 | 4 | 12 | +8 |

注：差异主要来自统计口径更新与源码新增注释/占位标记，不代表功能回退。
