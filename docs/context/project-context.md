# 项目上下文(根治理层)

## 用途

AI 在这一层做事前需要的最短基线:身份、验证命令、治理范围。原地更新,不建带日期副本。
「当前在做什么」看 `docs/plans/` 未完成计划,不记在这。

## 身份

- 项目:OrgReOrg(根治理层 / `example`)
- 产品类型:**企业级上下文管理**,IM 为员工载入口
- 主要用户:企业员工(经 IM 使用上下文管理)
- 本层角色:**治理 4 个子仓库**、统筹跨仓边界与集成,**不改子仓内部**
- 文档 freshness:`fresh`(2026-07-23;产品形态已成形 → `architecture/project-vision.md` + `design/app-overview.md` + `requirements/mvp.md`;身份统一 deferred)

## 4 仓技术基线

| 仓 | 角色 | 栈 |
| --- | --- | --- |
| OrgReOrgServer | IM 业务后端(好友/群/消息),API `:8090`,悟空IM 提供通讯 | Go 1.20 |
| OrgReOrgWeb | IM PC/桌面客户端(Web/Electron/Tauri) | React17 / Yarn1 / Turborepo |
| OrgReOrgAdmin | IM 管理后台 | Vue3 / Vite / pnpm |
| OrgReOrg | 上下文引擎(Harness,可移植 `framework/` + 飞轮) | Python / uv |

## 验证命令(按受影响的仓跑;本层不改子仓内部,验证委托给各仓)

| 仓 | 装依赖 | 构建 | 检查 / 测试 |
| --- | --- | --- | --- |
| OrgReOrgServer | `go mod download` | `go build ./...` | `go test ./pkg/... ./internal/...`;完整 `make env-test` 后 `go test ./...`;`go vet ./...` |
| OrgReOrgWeb | `yarn install` | `yarn build` | 类型检查 `tsc -p apps/web/tsconfig.json --noEmit`(⚠ `yarn lint` 空转、无单测、基线 4 个 TS 错;判据=错误数不增) |
| OrgReOrgAdmin | `pnpm install` | `pnpm build` | `pnpm lint`(无自动化测试;改后 `pnpm dev`/`pnpm preview` 验流程) |
| OrgReOrg | `uv sync` | `uv run mkdocs build --strict` | `bash scripts/ci_check.sh`(单一门禁,全绿=就绪) |

本层自身(跨仓文档)暂无构建;验证 = 文档链接/一致性人工核 + 受影响子仓命令。

## 在用的可选层

只勾本层真维护的:

- [x] `discussions/`  [ ] `audits/`  [ ] `testing/`  [ ] `skills/`  [ ] `analysis/`  [ ] `retrospectives/`  [ ] `lessons/`

## AI 阻塞条件

- 要改任一子仓**内部**代码却没走该仓自己的 harness → 停,交回该仓流程。
- 要动跨仓契约(Server API ↔ Web/Admin)、部署、版本对齐 → plan-first。
- 没有 owner 文档描述预期行为就不实现(不对真空实现)。

## 给 AI 的备注

- 跨仓事实以 `architecture/system-baseline.md` 为准;子仓内部事实以各仓 README/AGENTS 为准。
- 可依据真实仓库证据订正上下文,但不得把 stale 标 fresh、不得擅自降级受保护区。
