<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/rvbench/.github/main/profile/assets/rvbench-logo-dark.svg">
    <img alt="rvbench" src="https://raw.githubusercontent.com/rvbench/.github/main/profile/assets/rvbench-logo.svg" width="480">
  </picture>
</p>

<h3 align="center">RISC-V 软硬协同评测基础设施</h3>

<p align="center">合同定义实验 · worker 在真实板卡上执行 · 结果自带可复核的证据</p>

---

rvbench 要回答的不是「某段代码在板子上跑多快」，而是「这次测量的软件实现、硬件平台、输入、数值结论和样本，是否足以支撑一句可比较的话」。

软硬协同的结论只有在两端都能被单独指认时才成立：哪个 kernel 实现、哪版工具链、哪块板、哪组 CPU 亲和与运行时配置。rvbench 把这些写成显式的实验维度，而不是留在脚注、口头描述或一张截图里。

```text
观测 = 合同 + 代码身份 + 有效环境 + 验证证据 + 原始样本 + 显式状态
```

同一个名称不一定是同一个实验：shape、dtype、布局、输入生成规则、数值容差、工具链版本任一不同，延迟就不可直接比较。因此每个 case 固定自己的工作负载身份，结果显式记录代码与环境摘要，未知的检查保持未知。

## 第一个落地的场景：算子

算子是本基础设施第一个落地的典型场景，也是第一份公开测试集。

选它不是因为算子本身重要，而是因为它是软硬协同里边界最清楚的单元——一次调用、一组输入、一个数值参照。软件前端的选择与硬件后端的差异都能被约束在同一份工作负载身份内，测出来的差异才有可能归因。

| 维度 | 可变 | 固定并写入结果 |
|---|---|---|
| **软件前端** | `native-c` / `triton`、编译器与运行时版本、编译选项 | 导出入口与源码语义 |
| **硬件后端** | K1 / K3、CPU 亲和、AI thread 配置 | 板卡、instance、worker 身份 |
| **工作负载** | — | shape、dtype、layout、属性、generator + seed、容差 |

同一份请求在 K1 / K3 上都能选 `native-c` 或 `triton`，前后端两个维度由匹配机制按请求自由组合；排行榜按实现分别分组，不把不同软硬组合的延迟混进同一条曲线。

算子是起点而不是边界。同一套合同与证据模型可以承载更粗粒度的场景——算子库、计算图、端到端模型；每个新场景只增加自己的工作负载定义与参照，不改变观测结构。

## 仓库

| 仓库 | 内容 |
|---|---|
| [`rvbench`](https://github.com/rvbench/rvbench) | manager、CLI、运维工具、hello-world 最小示例 |
| [`rvbench-core-rust`](https://github.com/rvbench/rvbench-core-rust) | `rvbench-protocol`：OpenAPI 正本、生成类型、校验、摘要、存储 |
| `rvbench-spacemit-worker` | 板端执行服务：编译、验证、计时、能力探测 · *构建中* |
| `rvbench-official-suites` | 算子评测集，第一份公开测试集 · *构建中，后续开源* |

公开仓目前只内置 `hello-world` 一个最小用例，用于跑通提交、编译、执行与结果落盘；完整的算子评测集与基准数据会随 `rvbench-official-suites` 开源。

## 设计原则

| 原则 | 含义 |
|---|---|
| **合同优先** | JSON Schema 与 OpenAPI 是正本；wire 类型由合同生成，不手写平行 DTO |
| **后端与前端自由匹配** | 硬件后端（板卡、CPU 亲和、AI thread、工具链）与软件前端（`implementation` 与源文件）各自独立定义，按请求组合；换一边不动另一边，差异可单独归因 |
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

`func` / `correctness` / `performance` 三类任务共用同一套合同，同一份 spec 在三块板上语义一致。方向、结构与运行手册见 [`rvbench` README](https://github.com/rvbench/rvbench#readme)。

## 参与

- 缺陷与提案：在对应仓库开 Issue
- 合同变更：先改 `rvbench-core-rust` 的 OpenAPI，再生成类型与 conformance 测试，最后落到实现
- 新场景接入：新增工作负载定义与参照，复用既有观测结构，不另起一套测量口径
