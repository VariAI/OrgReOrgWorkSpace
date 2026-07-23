# Agent Instructions — OrgReOrg 根治理层

## 这是什么层

`example/` 根**不是产品代码,是治理层**:统筹 4 个子仓库,让它们朝一个吸引子收敛。
采用 AGE(Attractor-Guided Engineering)。owner 文档在 `docs/`,是这一层的吸引子。
**本层不改子仓内部代码**,只治理跨仓的边界、集成契约与协同。

## 吸引子(收敛方向)

- 产品 = **企业级上下文管理**;用户 = **企业员工,经 IM 使用**。
- IM(`OrgReOrgServer`/`Web`/`Admin`)= **载入口 / 前端触点**。
- 上下文引擎 = `OrgReOrg`(Harness,`framework/`)。
- 待澄清(open):引擎 ↔ IM 的集成尚未记录(OQ-1);产品差异化定位待定(OQ-3)。见 `docs/backlog/README.md`。

## 4 个子仓(载体,本层不改其内部)

见 `docs/context/codebase-map.md`。每个子仓有自己的 `AGENTS.md` 与验证命令;
内部改动走各自 harness,本层不代劳。

## 源真相优先级

1. `docs/context/project-context.md`(本层基线 + 验证命令)
2. `docs/context/ai-autonomy-policy.md`(受保护区)
3. `docs/architecture/system-baseline.md`(4 仓拓扑与集成契约)
4. 各子仓自己的 `README` / `AGENTS.md`(其内部事实)

冲突时先记录 drift 再改,不把旧说明当现真相。

## 任务路由

动手前先分类:

- 跨仓集成 / 契约 / 拓扑 / 协同规则 → **本层**(`docs/`)。
- 某个子仓**内部**实现 → 交给该子仓、走它自己的 `AGENTS.md`,本层不代劳。
- 分不清归谁 → 先写进 `docs/backlog/README.md` 或澄清,不硬猜。

## 何时要计划

跨仓契约、集成、部署、版本对齐、影响 >1 个子仓的改动 → 先写计划(`docs/plans/`,用到再建)。
本层单文件、低风险的文档更新可跳过。

## 完成检查

- 对照真实改动,不靠记忆。
- 跑受影响子仓的真实验证命令(见 `project-context.md` 表)。
- 行为 / 结构变了就更新 `docs/architecture/` + 记一条 `docs/logs/2026/MM-DD.md`。

## CLAUDE.md

`CLAUDE.md` 是本文件的指针,两者保持一致。
