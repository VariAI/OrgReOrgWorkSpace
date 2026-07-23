# 代码地图(4 仓 + 集成缝)

本层治理下列 4 个独立 git 仓库(各有自己的 `AGENTS.md`;**本层不改其内部**):

## OrgReOrgServer — IM 业务后端(Go)

- 入口 `main.go`;按 feature 分 `modules/`(group/message/user…);共享在 `internal/`、`pkg/`。
- 对外 API `http://localhost:8090`(Swagger `/swagger-ui/`,业务 `/v1/`)。
- 与悟空IM(通讯层)经 Webhook/GRPC + API 协作。

## OrgReOrgWeb — IM 客户端(React17 / Turborepo)

- `apps/web/`(浏览器 / Electron / Tauri 外壳);UI 与服务在 `packages/`(`orgreorgbase` 聊天核心等)。
- 走 `WKApp.apiClient` 调 Server API;IM 长连接走 `wukongimjssdk`。基于 TangSengDaoDao 改造。

## OrgReOrgAdmin — IM 管理后台(Vue3 / Vite)

- `src/pages` 业务页,`src/api` 接口封装。
- 消费 Server API(`src/config/modules/dev.ts` 里 `APP_URL=http://localhost:8090/v1/`,生产 `/api/v1/` 反代)。

## OrgReOrg — 上下文引擎 / Harness(Python)

- `framework/` 可移植底座;`workspaces/<id>/` 各工作区;`scripts/ci_check.sh` 单一门禁。
- 自带成熟 harness(五组等价文档 + ~200 治理脚本)。**本层不覆盖它内部。**

## 集成缝(本层治理重点)

- **Web/Admin → Server**:HTTP API `:8090`(`/v1`,生产 `/api/v1` 反代)。**契约 = 受保护区。**
- **Server ↔ 悟空IM**:通讯层。
- **上下文引擎(OrgReOrg)↔ IM(Server/Web/Admin):当前无记录的集成缝** —— 这是吸引子(「经 IM 用上下文管理」)尚未落地的关键缺口。见 `backlog/README.md` **OQ-1**。
