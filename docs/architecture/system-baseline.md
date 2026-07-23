# 系统基线:4 仓拓扑与集成契约(根治理层首个 owner 文档)

## 目标

记录 4 个子仓当前**如何拼成一个产品**、边界在哪、跨仓契约是什么。这是「治理 4 仓」的核心事实源。
接受变更后更新;子仓**内部**结构由各仓自己拥有,这里只记**跨仓**事实。

## 吸引子

企业级上下文管理,IM 为员工载入口。IM 三仓 = 触点;OrgReOrg/Harness = 上下文引擎。

## 当前拓扑(2026-07-23,据各仓 README/AGENTS)

```text
员工 ──(IM 客户端)──> OrgReOrgWeb  ─┐
管理员 (后台)────────> OrgReOrgAdmin ┼─ HTTP :8090 /v1 ─> OrgReOrgServer ─(Webhook/GRPC)─> 悟空IM(通讯层)
                                    ┘
上下文引擎:OrgReOrg(Harness / framework)  = 主干/宿主;上面整套 IM 是它的【原生 surface,L2 目标 — 见集成契约】
```

## 跨仓契约(受保护)

- **Server API `:8090`**:Web 与 Admin 的唯一后端契约(Admin dev `/v1/`,生产 `/api/v1/` 反代)。改它 = plan-first,评估两个消费方。
- **Server ↔ 悟空IM**:通讯层 Webhook / API。
- **部署**:各仓有独立 Dockerfile;整体编排 / 反代未在本层记录(OQ-2)。

## 集成契约(OQ-1 已定,2026-07-23 用户拍板)

- **方向**:agent 平台(OrgReOrg)= 主干/宿主;IM(Server/Web/Admin)= 平台的**原生 surface**。做「agent 里的 IM」,不做「IM 里的 agent」(见 `project-vision.md`)。
- **深度**:目标 **L2 原生 surface**、朝 L3 演进;L1 连接器/bot 仅作跳板。分水岭 = 平台拥有身份与上下文,IM 只渲染。
- **surface 唯一性**:自有 IM 为员工**唯一**人机入口,**取代 Matrix/Element/企微**(旧连接器逐步退,退役本身另立任务)。
- **四根缝**(受保护:凡跨仓接口 plan-first):
  1. 身份统一(先行,L2 地基)→ 立项 `plans/2026-07-23-identity-unification.md`
  2. 消息即证据(IM 会话 → 上下文引擎)
  3. 投影回 IM(上下文/知识/主动协助 → 客户端)
  4. agent 作一等参与者(不止 robot API)

## 仍开放

- **OQ-2:整体部署 / 反代拓扑**(4 仓 + 悟空IM 如何一起起、端口 / 域名怎么排)未记录。

## 非目标(当前)

- 不在本层重述各子仓内部实现。
- 不改子仓代码。
