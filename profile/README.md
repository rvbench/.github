<p align="center"><b>English</b> · <a href="https://github.com/rvbench/.github/blob/main/profile/README.zh-CN.md">简体中文</a></p>

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/rvbench/.github/main/profile/assets/rvbench-logo-dark.svg">
    <img alt="rvbench" src="https://raw.githubusercontent.com/rvbench/.github/main/profile/assets/rvbench-logo.svg" width="480">
  </picture>
</p>

<h3 align="center">Tides change. The gauge doesn't.</h3>

<p align="center">RISC-V software–hardware co-evaluation infrastructure · every experiment configurable · every result reproducible · every piece of evidence checkable</p>

---

The RISC-V ecosystem is deeply diverse: extensions, microarchitectures, toolchains and boards each come in many flavors, and the same piece of code can land on a different conclusion under a different combination. In an ecosystem like this, a trustworthy, reproducible and extensible evaluation system is infrastructure in itself — new hardware and new software gain the full set of existing capabilities the moment they are plugged in.

What rvbench aims to provide is accurate, reproducible results for software–hardware co-evaluation. A co-design conclusion holds only when both ends can be identified separately: which kernel implementation, which toolchain version, which board, which CPU affinity and runtime configuration. rvbench writes these as explicit experiment dimensions and stores them alongside the result, so any result can be reproduced directly, at any time.

```text
observation = contract + code identity + valid environment + verification evidence + raw samples + explicit state
```

One name does not always mean one experiment: change any of shape, dtype, layout, input generation rule, numerical tolerance or toolchain version, and the latencies stop being directly comparable. So every case pins its own workload identity, results record code and environment digests explicitly, and checks that are unknown stay unknown.

Open-source stacks can be evaluated; closed-source boards and toolchains plug in just the same — whether anything gets open-sourced stays in the user's hands, and results are equally reproducible.

> [!IMPORTANT]
> **Help fill in the marks**
>
> - **Hardware / SDK workers**: got a new board, an SDK, or a simulator? Bring it in and immediately see which layer your real workloads bottleneck on. XiangShan and Ruyi are under construction; XuanTie is in preparation.
> - **Application / software suites**: turn the workloads you actually run every day into an evaluation suite, and it adapts to a diverse set of hardware backends at once, with no per-backend porting.
>
> To join in, open an issue in this repo or in [rvbench](https://github.com/rvbench/rvbench) with your platform and SDK version — one minimal workload that runs is enough to start.

## Quick start

```sh
cargo install --git https://github.com/rvbench/rvbench-cli.git --locked    # installs rvbench and rb-cli
git clone https://github.com/rvbench/rvbench.git && cd rvbench

rvbench validate spec examples/hello-world/spec.json   # offline validation, no board, no network
rvbench digest   examples/hello-world/spec.json        # workload identity digest

rb-cli init demo        # generate a standalone worker + manager + suite environment
```

The directory produced by `rb-cli init` ships with a Makefile: `make build` → `make manager` / `make worker` → `make validate` → `make submit`, taking one `hello-world` job from submission to a result on disk. Worker repos are not public yet; in a public environment use `rb-cli init demo --manager --suite` to fetch only the control plane and the suite.

The three task kinds — `func` / `correctness` / `performance` — share one contract, so the same spec means the same thing on all three boards. For direction, structure and the operations manual, see the [`rvbench` README](https://github.com/rvbench/rvbench#readme).

## First facilities

| Repository | Contents |
|---|---|
| [`rvbench`](https://github.com/rvbench/rvbench) | canonical wire contracts, operations tooling, minimal hello-world example |
| [`rvbench-cli`](https://github.com/rvbench/rvbench-cli) | control plane client and offline validation library: `rvbench` / `rb-cli` |
| [`rvbench-official-manager`](https://github.com/rvbench/rvbench-official-manager) | control plane service: submission, queue, admission, ledger, observability rollups |
| [`rvbench-core-rust`](https://github.com/rvbench/rvbench-core-rust) | `rvbench-protocol`: wire contract copy, generated types, validation, digests, storage |
| `rvbench-spacemit-worker` | board-side execution service: compile, verify, time, probe capabilities · *under construction* |
| `rvbench-xiangshan-worker` | board-side execution service for XiangShan · *under construction* |
| `rvbench-ruyi-worker` | board-side execution service for Ruyi · *under construction* |
| `rvbench-xuantie-worker` | board-side execution service for XuanTie · *in preparation* |
| `rvbench-official-suites` | operator suites, the first public test set · *in testing, open-sourcing soon* |
| `rvbench-security-suite` | security suite · *under construction* |

Workers split by backend, deploy independently, and share one contract and one execution semantics; adding a new board means adding a worker repo, with no change to the control plane or the CLI.

The public repos ship one minimal `hello-world` case, enough to exercise submission, compilation, execution and result persistence; the full operator suite and baseline data will be open-sourced with `rvbench-official-suites`.

## rvbench-cli

`rvbench-cli` is the only way in from outside: control plane client plus offline validation library, native Rust, no Python wrapper. Install once and you get two commands — `rvbench` and its alias `rb-cli` — with identical subcommands, arguments and exit codes.

```sh
cargo install --git https://github.com/rvbench/rvbench-cli.git --locked
rvbench --help
```

| Capability | Command | Notes |
|---|---|---|
| Scaffolding | `rb-cli init` | pulls worker / manager / suite and generates config, a submittable `JobRequest` and a Makefile |
| Control plane | `rvbench control` | `submit` / `get` / `workers` / `capabilities` / `drain`; points at a manager, or add `--direct` to talk to a worker directly |
| Offline validation | `rvbench validate` | `spec` / `hidden` / `results` / `baseline-results`; schemas embedded at compile time, no network, no board |
| Identity digest | `rvbench digest` | SHA-256 of canonical JSON, same implementation as manager and worker |

`validate spec --require-ready` returns 2 while a readiness blocker remains: a legal contract and a validated board run are two different things. The validator, digests and I/O are also exported as a lib (`rvbench_cli`), shared by the operations tooling and the CLI — there is no second validation implementation.

## Design principles

| Principle | Meaning |
|---|---|
| **Contract first** | JSON Schema and OpenAPI are canonical; wire types are generated from the contract, never hand-written parallel DTOs |
| **Backend and frontend match freely** | Hardware backend (board, CPU affinity, AI thread, toolchain) and software frontend (`implementation` and source files) are defined independently and combined per request; change one side without touching the other, and the difference can be attributed |
| **Identity cannot change implicitly** | A case pins shape, dtype, layout, attributes and input generator; an unknown output check is recorded as `not_passed`, never as a pass |
| **Execution, validation and timing are three things** | `func` only proves it compiles and runs, `correctness` only covers the declared scope, `performance` only answers for samples that passed the numerical precheck |
| **Evidence travels with the result** | Raw samples, warmup, repeat count, artifact SHA-256, worker and board identity are stored together with the result |

A task `done` is a service terminal state, not scientific validation. Contract validation, hash agreement and a `done` job status — none of these alone counts as full verification.

## More resources

| Goal | Entry point | Status |
|---|---|---|
| Submit your first operator | [spec.json v2 format and first instance](https://github.com/rvbench/rvbench/blob/main/docs/specs/2026-10-08-spec-json%E6%A0%BC%E5%BC%8F.md) — operator declaration, cases, input generation and numerical contract | available |
| Build your first worker | [`examples/hello-world/worker`](https://github.com/rvbench/rvbench/tree/main/examples/hello-world/worker) — the smallest skeleton of a board-side execution service | in preparation |
| Build your first suite | [`examples/hello-world/suite`](https://github.com/rvbench/rvbench/tree/main/examples/hello-world/suite) — the full directory layout from spec to a submittable suite | in preparation |

## Contributing

- Bugs and proposals: open an issue in the relevant repository
- Contract changes: start from `contracts/api/openapi.json` in `rvbench`, sync the protocol repo and the CLI copy, regenerate types and conformance tests, then land the implementation
- New scenarios: add a workload definition and its reference, reuse the existing observation structure instead of inventing a second measurement language
