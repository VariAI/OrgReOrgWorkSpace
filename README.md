# OrgReOrgWorkSpace

**企业级上下文管理**产品的**根治理层 / 超级仓库(superproject)**。

本仓库**不是产品代码**,而是治理层:统筹 4 个独立子仓库、协调它们的跨仓边界、集成契约与协同,让它们朝同一个吸引子收敛。采用 AGE(Attractor-Guided Engineering),owner 文档在 [`docs/`](docs/),是这一层的吸引子。

> **铁律:本层不改任一子仓的内部代码。** 子仓内部改动走各自的 `AGENTS.md` 与验证流程。

- 产品 = 企业级上下文管理;用户 = 企业员工,经 IM 使用。
- IM(Server / Web / Admin)= 载入口 / 前端触点;上下文引擎 = `OrgReOrg`。

---

## 仓库结构

以 git 子模块形式纳管 4 个项目,超级仓库只记录各子模块的**提交指针**:

| 子模块 | 角色 | 栈 | 跟踪分支 |
| --- | --- | --- | --- |
| [OrgReOrgServer](OrgReOrgServer) | IM 业务后端(好友/群/消息),API `:8090`,悟空IM 提供通讯 | Go 1.20 | `develop` |
| [OrgReOrgWeb](OrgReOrgWeb) | IM PC/桌面客户端(Web/Electron/Tauri) | React17 / Yarn1 / Turborepo | `develop` |
| [OrgReOrgAdmin](OrgReOrgAdmin) | IM 管理后台 | Vue3 / Vite / pnpm | `main` |
| [OrgReOrg](OrgReOrg) | 上下文引擎(Harness,可移植 `framework/` + 飞轮) | Python / uv | `main` |

详见 [`docs/context/codebase-map.md`](docs/context/codebase-map.md)。

---

## 快速开始

克隆时带上子模块:

```bash
git clone --recurse-submodules https://github.com/VariAI/OrgReOrgWorkSpace.git
```

已克隆但缺子模块内容:

```bash
git submodule update --init --recursive
```

把各子模块更新到其跟踪分支的远程最新:

```bash
git submodule update --remote
```

---

## 协作与提交流程

**两级仓库,两级提交:**

- 改某个子仓的**内部实现** → 进入该子模块目录提交、推送,走它自己的 `AGENTS.md`。本层不代劳。
- 改**跨仓契约 / 集成 / `docs/`**,或要把子模块指针固化到某个新版本 → 在根仓库提交。

```bash
# 1) 子仓内部改动(最常见)
cd OrgReOrgServer
git switch develop
# ...改代码... 
git commit -m "feat: xxx" && git push origin develop

# 2) 若要让治理层记录该新版本:回根仓库固化指针
cd ..
git add OrgReOrgServer
git commit -m "chore(submodule): bump OrgReOrgServer to <短hash>"
git push
```

> ⚠ **先推子模块,再推根仓库**,否则根仓库指针会指向远程尚不存在的 commit。

**提交信息**沿用 Conventional Commits:`type(scope): 摘要`(`feat`/`fix`/`docs`/`chore`/`refactor`/`test`/`build`)。

**跨仓契约、集成、部署、版本对齐、影响 >1 个子仓**的改动 → 先写 `docs/plans/`,再动手,并补一条 `docs/logs/2026/MM-DD.md`。

---

## 验证

本层自身(跨仓文档)暂无构建,验证 = 文档链接/一致性人工核 + 受影响子仓的命令。各子仓验证命令见 [`docs/context/project-context.md`](docs/context/project-context.md)。

---

## 文档导航

- 治理规则与任务路由:[`AGENTS.md`](AGENTS.md)(`CLAUDE.md` 为其指针)
- 项目基线 / 身份 / 验证命令:[`docs/context/project-context.md`](docs/context/project-context.md)
- 4 仓拓扑与集成缝:[`docs/context/codebase-map.md`](docs/context/codebase-map.md)
- 架构与产品愿景:[`docs/architecture/`](docs/architecture/) · [`docs/design/`](docs/design/)
- 待澄清事项(open questions):[`docs/backlog/README.md`](docs/backlog/README.md)
