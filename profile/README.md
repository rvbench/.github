<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/rvbench/.github/main/profile/assets/rvbench-logo-dark.svg">
    <img alt="rvbench" src="https://raw.githubusercontent.com/rvbench/.github/main/profile/assets/rvbench-logo.svg" width="480">
  </picture>
</p>

<h3 align="center">潮水涨落，刻度不动</h3>

<p align="center">RISC-V 软硬协同评测基础设施 · 每个实验可定制 · 每个结果可复现 · 每条证据可核对</p>

---

RISC-V 生态高度多样化，也常被称作碎片化：扩展集合、微架构实现、工具链、板卡各家一套，同一段代码换一种组合就可能换一个结论。对这样的生态，一套可信赖、可复现、可扩展的评测系统本身就是基础设施——可信赖指每个数都带着可核对的证据，可复现指同一份合同重跑回到同一处，可扩展指新扩展、新板卡、新场景接入时只加定义、不动测量口径。

rvbench 要回答的是：准确、可复现的软硬协同测试结果。可复现排在可比较之前——同一份合同换一天、换一块板重跑，结论要落回同一处；跑不住的结果不进入比较。落到每个 case 上，问题就是：这次测量的软件实现、硬件平台、输入、数值结论和样本，是否足以支撑一句可比较的话，以及这句话在重跑之后是否还成立。

软硬协同的结论只有在两端都能被单独指认时才成立：哪个 kernel 实现、哪版工具链、哪块板、哪组 CPU 亲和与运行时配置。rvbench 把这些写成显式的实验维度，随结果一起落盘；脚注、口头描述和截图都算不得证据。

```text
观测 = 合同 + 代码身份 + 有效环境 + 验证证据 + 原始样本 + 显式状态
```

同一个名称不一定是同一个实验：shape、dtype、布局、输入生成规则、数值容差、工具链版本任一不同，延迟就不可直接比较。因此每个 case 固定自己的工作负载身份，结果显式记录代码与环境摘要，未知的检查保持未知。

> [!TIP]
> **一起把刻度补上**
>
> - **硬件 / SDK worker**：手上有一块新板子、一套 SDK，或者一个模拟器？都可以接进来。香山、如意正在建，玄铁也在筹备。
> - **应用 / 软件 suite**：你每天真正在跑的负载，比合成用例更贴近现场。整理成评测集，交给同一套合同来判。
>
> 想参与，就在本仓或 [rvbench](https://github.com/rvbench/rvbench) 开个 Issue，说说平台和 SDK 版本；有一份能跑起来的最小负载就够开始。

## 快速开始

```sh
cargo install --git https://github.com/rvbench/rvbench-cli.git --locked    # 装 rvbench 与 rb-cli
git clone https://github.com/rvbench/rvbench.git && cd rvbench

rvbench validate spec examples/hello-world/spec.json   # 离线校验，无需板卡与网络
rvbench digest   examples/hello-world/spec.json        # 工作负载身份摘要

rb-cli init demo        # 生成 worker + manager + suite 的独立运行环境
```

`rb-cli init` 产出的目录自带 Makefile：`make build` → `make manager` / `make worker` → `make validate` → `make submit`，一条 `hello-world` 作业从提交跑到结果落盘。worker 仓暂未公开，公开环境下用 `rb-cli init demo --manager --suite` 只取控制面与用例。

`func` / `correctness` / `performance` 三类任务共用同一套合同，同一份 spec 在三块板上语义一致。方向、结构与运行手册见 [`rvbench` README](https://github.com/rvbench/rvbench#readme)。

## 首批上线设施

| 仓库 | 内容 |
|---|---|
| [`rvbench`](https://github.com/rvbench/rvbench) | wire 合同正本、运维工具、hello-world 最小示例 |
| [`rvbench-cli`](https://github.com/rvbench/rvbench-cli) | 控制面客户端与离线校验库：`rvbench` / `rb-cli` |
| [`rvbench-official-manager`](https://github.com/rvbench/rvbench-official-manager) | 控制面服务：提交、队列、准入、账本与观测汇总 |
| [`rvbench-core-rust`](https://github.com/rvbench/rvbench-core-rust) | `rvbench-protocol`：wire 合同副本、生成类型、校验、摘要、存储 |
| `rvbench-spacemit-worker` | 板端执行服务：编译、验证、计时、能力探测 · *构建中* |
| `rvbench-xiangshan-worker` | 香山（XiangShan）板端执行服务 · *构建中* |
| `rvbench-ruyi-worker` | 如意（Ruyi）板端执行服务 · *构建中* |
| `rvbench-xuantie-worker` | 玄铁（XuanTie）板端执行服务 · *筹备中* |
| `rvbench-official-suites` | 算子评测集，第一份公开测试集 · *测试中，即将开源* |
| `rvbench-security-suite` | 安全评测集：常量时间与侧信道暴露的判定用例 · *构建中* |

worker 按后端拆仓，各自独立部署，共用同一套合同与执行语义；接入新板卡只新增一个 worker 仓，不动控制面与 CLI。

公开仓目前只内置 `hello-world` 一个最小用例，用于跑通提交、编译、执行与结果落盘；完整的算子评测集与基准数据会随 `rvbench-official-suites` 开源。

## rvbench-cli

`rvbench-cli` 是外部接触 rvbench 的唯一入口：控制面客户端 + 离线校验库，Rust 原生、无 Python 包装。装一次得到两个命令——`rvbench` 与别名 `rb-cli`，子命令、参数与退出码完全一致。

```sh
cargo install --git https://github.com/rvbench/rvbench-cli.git --locked
rvbench --help
```

| 能力 | 命令 | 说明 |
|---|---|---|
| 初始化 | `rb-cli init` | 拉 worker / manager / suite，生成配置、可提交的 `JobRequest` 与 Makefile |
| 控制面 | `rvbench control` | `submit` / `get` / `workers` / `capabilities` / `drain`；指向 manager，或加 `--direct` 直连 worker |
| 离线校验 | `rvbench validate` | `spec` / `hidden` / `results` / `baseline-results`；schema 编译期嵌入，不访问网络、不需要板卡 |
| 身份摘要 | `rvbench digest` | canonical JSON 的 SHA-256，与 manager / worker 同一实现 |

`validate spec --require-ready` 在仍有 readiness blocker 时返回 2：合同合法与完成板上验证是两件事。校验器、摘要与 I/O 同时以 lib（`rvbench_cli`）导出，运维工具与 CLI 共用同一套实现，不存在第二套校验口径。

## 设计原则

| 原则 | 含义 |
|---|---|
| **合同优先** | JSON Schema 与 OpenAPI 是正本；wire 类型由合同生成，不手写平行 DTO |
| **后端与前端自由匹配** | 硬件后端（板卡、CPU 亲和、AI thread、工具链）与软件前端（`implementation` 与源文件）各自独立定义，按请求组合；换一边不动另一边，差异可单独归因 |
| **身份不可隐式改变** | 一个 case 固定 shape、dtype、layout、属性与输入生成器；未知的输出检查记为 `not_passed`，不记为通过 |
| **执行、验证、计时是三件事** | `func` 只证明能编译能调用，`correctness` 只覆盖声明范围，`performance` 只对通过数值预检的样本负责 |
| **证据随结果走** | 原始样本、warmup、重复次数、工件 SHA-256、worker 与板卡身份一并落盘 |

任务 `done` 是服务终态，不等于科学评测通过。合同校验通过、哈希一致、作业状态 `done`，任何一项单独都不能当作完整验证。

## 更多资源

| 目标 | 入口 | 状态 |
|---|---|---|
| 提交第一个算子 | [spec.json v2 格式与首份实例](https://github.com/rvbench/rvbench/blob/main/docs/specs/2026-10-08-spec-json%E6%A0%BC%E5%BC%8F.md) — 算子声明、用例、输入生成与数值合同 | 可用 |
| 构建第一个 worker | [`examples/hello-world/worker`](https://github.com/rvbench/rvbench/tree/main/examples/hello-world/worker) — 板端执行服务的最小实现骨架 | 准备中 |
| 构建第一个 suite | [`examples/hello-world/suite`](https://github.com/rvbench/rvbench/tree/main/examples/hello-world/suite) — 从 spec 到可提交评测集的完整目录 | 准备中 |

## 参与

- 缺陷与提案：在对应仓库开 Issue
- 合同变更：先改 `rvbench` 的 `contracts/api/openapi.json`，同步协议仓与 CLI 副本，再生成类型与 conformance 测试，最后落到实现
- 新场景接入：新增工作负载定义与参照，复用既有观测结构，不另起一套测量口径
