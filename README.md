### 🇺🇸 English Version (`README.md`)

```markdown
# YuCheng (御程) - The AI-Native Systems Language with Hardware Awareness

> **From Silicon to Cloud: One Language to Rule Them All.**  
> A systems programming language that combines **enforced architectural contracts** (to prevent AI hallucinations) with **native hardware syntax** (for CPU/GPU heterogeneous computing).

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Backend](https://img.shields.io/badge/Backend-LLVM%2021-blue)](#)
[![Hardware](https://img.shields.io/badge/Feature-CPU%2FGPU%20Native-red)](#)
[![Written in](https://img.shields.io/badge/Written%20in-Rust-orange)](#)

## 🚀 The Problem: The Gap Between AI and Metal

In 2026, developers face a critical disconnect:
1.  **AI Hallucinations**: LLMs generate logic but ignore architectural boundaries and safety contracts.
2.  **Hardware Black Box**: Controlling modern heterogeneous hardware (P-Core/E-Core, SIMD, GPU) requires complex C/C++ intrinsics or CUDA, which AI struggles to write correctly.
3.  **Debugging Nightmares**: Tracing memory errors across CPU/GPU boundaries is nearly impossible.

**YuCheng** bridges this gap. It enforces rigor at the compiler level while exposing raw hardware power through simple, high-level syntax.

## 💡 Core Innovations

### 1. Native Hardware Syntax (CPU & GPU)
Forget complex intrinsics. YuCheng brings hardware control directly into the language grammar.

#### **A. Heterogeneous Scheduling (Big.LITTLE)**
Directly assign tasks to Performance Cores or Efficiency Cores.
```yu Cheng
// Run heavy AI inference on P-Cores
并行 core.p {
    模型.推理 (数据);
}

// Run background logging on E-Cores
并行 core.e {
    日志.写入 (事件);
}
```

#### **B. SIMD & Vectorization**
Explicitly utilize CPU vector instructions (AVX-512, NEON) without assembly.
```yu Cheng
// Auto-vectorize this loop using AVX-512
向量化 (策略 = "AVX512") 函数 处理大数据 (数组：[浮点数]) {
    对于 元素 in 数组 {
        元素 = 元素 * 2.0 + 1.0;
    }
}
```

#### **C. GPU Tensor Offloading**
Seamlessly offload matrix operations to GPU/NPU with native tensor types.
```yu Cheng
// Define tensors on GPU VRAM
张量<浮点数，设备="GPU"> 矩阵 A = 加载 ("data.bin");
张量<浮点数，设备="GPU"> 结果 = 矩阵乘法 (A, B); 
```

### 2. The "Four-File" Contract System (Anti-Hallucination)
To prevent AI from generating messy code, YuCheng enforces a strict separation of concerns at the file-system level.

```text
+-----------------------+       +-----------------------+
|   .yc  (Specification)| <---->|   .yci (Implementation)|
|  - Interfaces & Types |       |  - Business Logic      |
|  - Contracts          |       |  - Hardware Hints      |
+-----------+-----------+       +-----------+-----------+
            ^                               |
            | (Compiler Enforcement)        | (Runtime Flow)
            +-------------------------------+
[Rule]: If .yci violates .yc contracts or hardware constraints -> Compilation FAILS.
```
*   **AI Role**: The AI fills `.yci` but **cannot** break the rules defined in `.yc`.

### 3. Built-in Memory Transparency
Debug heterogeneous memory (Host vs. Device) with real-time visibility.
-   **Real-time JSON Snapshots**: The compiler injects probes to stream memory states during execution.
-   **Visual Debugging**: See pointer references, ownership changes, and data copies between CPU/GPU as a graph.
-   **AI Feedback Loop**: Feed memory snapshots back to the AI to auto-fix bugs.

### 4. AI-Native Transpilation
Describe your algorithm in Python or Natural English. Our built-in AI assistant transpiles it into optimized YuCheng code with correct **hardware annotations** (`core.p`, `向量化`, `设备="GPU"`).

## 🛠️ Architecture Overview

-   **Compiler**: Written in **Rust**, leveraging **LLVM 21** for native codegen (AOT).
-   **Scheduler**: Custom runtime scheduler (see [`SCHEDULER_DESIGN.md`](./SCHEDULER_DESIGN.md)) aware of CPU topology (Big.LITTLE).
-   **Backends**: Native (x86/ARM), WASM, and GPU Codegen (via LLVM SPIR-V).
-   **Toolchain**: Integrated AI Assistant, Memory Visualizer, and 13-Language Converter.

## 📚 Documentation

We believe in "Code as Documentation". Our architecture is fully open-sourced.

-   **[Architecture Design](./01-架构文档.md)** - Core philosophy and contract system.
-   **[Scheduler Design](./SCHEDULER_DESIGN.md)** - Deep dive into CPU/GPU scheduling logic.
-   **[Code Reference](./附录 A-代码级参考.md)** - Full syntax guide including hardware intrinsics.
-   **[Memory Transparency](./06-内存透视详解.md)** - How to use the debugger.
-   **[Project Overview](./00-项目总览.md)** - Roadmap and goals.

*(Note: Some docs are in Chinese. Use browser translation or our AI assistant for English summary.)*

## 🤝 Contributing

We are looking for contributors interested in:
-   **Compiler Engineering** (LLVM Backends for GPU/SIMD)
-   **Hardware Architecture** (ARM big.LITTLE, NVIDIA/AMD GPU)
-   **AI Integration** (Prompt Engineering, Auto-annotation)
-   **Standard Library** (Hardware-accelerated primitives)

---
**YuCheng**: Where High-Level Logic meets Low-Level Metal, enforced by Rigorous Architecture.
```

---
