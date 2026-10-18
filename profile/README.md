<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/rvbench/.github/main/profile/assets/rvbench-logo-dark.svg">
    <img alt="rvbench" src="https://raw.githubusercontent.com/rvbench/.github/main/profile/assets/rvbench-logo.svg" width="480">
  </picture>
</p>

<h3 align="center">RISC-V 算子评测基础设施</h3>

<p align="center">合同定义实验 · worker 在真实板卡上执行 · 结果自带可复核的证据</p>

---

rvbench 要回答的不是「这个算子有多快」，而是「这次测量的代码、环境、输入、数值结论和样本，是否足以支撑一句可比较的话」。

同一个算子名称不一定是同一个实验：shape、dtype、布局、输入生成规则、数值容差、工具链版本任一不同，延迟就不可直接比较。因此每个 case 固定自己的工作负载身份，结果显式记录代码与环境摘要，未知的检查保持未知。

```text
观测 = 合同 + 代码身份 + 有效环境 + 验证证据 + 原始样本 + 显式状态
```

## 仓库

| 仓库 | 内容 |
|---|---|
| [`rvbench`](https://github.com/rvbench/rvbench) | manager、CLI、运维工具、hello-world 示例评测集 |
| [`rvbench-core-rust`](https://github.com/rvbench/rvbench-core-rust) | `rvbench-protocol`：OpenAPI 正本、生成类型、校验、摘要、存储 |
| `rvbench-spacemit-worker` | 板端执行服务：编译、验证、计时、能力探测 · *构建中* |
| `rvbench-official-suites` | 完整官方评测集 · *构建中，后续开源* |

公开仓目前只内置 `hello-world` 一个最小用例，用于跑通提交、编译、执行与结果落盘；完整的算子评测集与基准数据会随 `rvbench-official-suites` 开源。

## 设计原则

| 原则 | 含义 |
|---|---|
| **合同优先** | JSON Schema 与 OpenAPI 是正本；wire 类型由合同生成，不手写平行 DTO |
| **身份不可隐式改变** | 一个 case 固定 shape、dtype、layout、属性与输入生成器；未知的输出检查记为 `not_passed`，不记为通过 |
| **执行、验证、计时是三件事** | `func` 只证明能编译能调用，`correctness` 只覆盖声明范围，`performance` 只对通过数值预检的样本负责 |
| **证据随结果走** | 原始样本、warmup、重复次数、工件 SHA-256、worker 与板卡身份一并落盘 |

任务 `done` 是服务终态，不等于科学评测通过。合同校验通过、哈希一致、作业状态 `done`，任何一项单独都不能当作完整验证。

## 快速开始

```sh
git clone https://github.com/rvbench/rvbench.git
cd rvbench
cargo build --workspace --locked
cargo run -p rvbench-cli -- validate spec examples/hello-world/spec.json
```

实现与硬件解耦：同一份请求在 K1 / K3 上都能选 `native-c` 或 `triton`，支持 `func` / `correctness` / `performance` 三类任务。方向、结构与运行手册见 [`rvbench` README](https://github.com/rvbench/rvbench#readme)。

## 参与

- 缺陷与提案：在对应仓库开 Issue
- 合同变更：先改 `rvbench-core-rust` 的 OpenAPI，再生成类型与 conformance 测试，最后落到实现
