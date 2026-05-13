# claude_multi_agent

这是一个面向 Claude Code / Copilot Agent 工作流的多 Agent 自主检测与修复编排仓库。当前仓库的核心内容位于 `.claude/agents/`，用于定义流水线角色、阶段顺序、运行状态和落盘产物。

## 仓库用途

- 以 `orchestrator` 作为唯一调度入口，串行驱动多个专项 Agent 执行检测、归集、修复与终验。
- 通过文件系统落盘状态、心跳、问题单和最终报告，不依赖外部消息队列。
- 适合把“发现问题 -> 汇总 -> 修复 -> 回归 -> 验收”流程固化为可重复执行的 Agent pipeline。

## 目录说明

```text
.
├── .claude/
│   ├── agents/                  # Agent 定义、流水线配置、运行产物
│   ├── rules.md                 # 仓库级约束，调度时应注入给子 Agent
│   └── skills/                  # 可复用技能说明
└── README.md                    # 本文件
```

`.claude/agents/` 下的关键文件：

- `README.md`：Agent 体系详细说明
- `workflow.json`：阶段顺序、状态机、重试与退出条件
- `state.json`：当前流水线状态
- `*.md`：各 Agent 的角色定义与执行约束
- `issues/`、`bundles/`、`fix-reports/`、`reports/`：运行期间生成的产物目录

## 快速开始

1. 打开本仓库作为工作区。
2. 确认 Agent 运行环境能读取 `.claude/agents/` 与 `.claude/rules.md`。
3. 通过 Agent 入口启动 `orchestrator`。
4. 观察 `.claude/agents/state.json`、`.claude/agents/heartbeats/` 和 `.claude/agents/issues/` 的变化。

如果你需要看各阶段职责、输入输出格式和运行顺序，请优先阅读 [`.claude/agents/README.md`](.claude/agents/README.md)。

## 流水线概览

```text
product-validator
  -> frontend-tester
  -> backend-admin-test
  -> crawler-stability-agent
  -> cross-system-check
  -> issue-collector
  -> dev-fix-agent (有缺陷时)
  -> release-final-agent
```

默认配置来自 `.claude/agents/workflow.json`：

- 串行执行，不开启阶段并行
- 最多 50 轮
- 需要连续 3 轮零异常才视为最终通过
- 有缺陷时自动进入修复回路

## 适用边界

- 本仓库当前主要提供 Agent 编排定义与文档，不包含业务应用源码。
- `.claude/rules.md` 中提到的 monorepo 约束属于目标业务仓库规范，供 Agent 在真实项目中执行时遵守。
- 如果要把这套体系移植到其他仓库，至少需要同步更新路径、Git 策略、状态文件位置和被测项目约束。
