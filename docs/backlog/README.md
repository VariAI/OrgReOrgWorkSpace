# Backlog(根治理层)

跨仓治理工作队列。子仓**内部**工作在各自仓里排,不进这里。
自主度:`implement` / `plan-first` / `ask-first`。

## Ready

| P | 工作 | 自主度 | 说明 |
| --- | --- | --- | --- |
| P0 | **IM 地基:聊天 / 群聊 / 企业通讯录·组织 + 前端 UI·交互**(近两天首要) | 子仓实现 | **现状(2026-07-23 探查):后端 `organization` 模块 + Admin 组织控制台 + Web 通讯录组织树均已建好**。活 = 接通验证(跑栈+建数据+E2E)+ Web 建群接组织树(现死代码)+ 统一两套 org API + 通讯录打磨。群看板/工作台/日程属智能体层,不在此。详见 log。 |
| P1 | **智能体 provision 通道**(平台→IM,身份统一缝第一切片) | plan-first | 身份层四决策已拍(`discussions/2026-07-23-agent-identity.md`);plan `plans/2026-07-23-agent-provisioning.md`(in progress,契约已定稿进 system-baseline);下一步 Execution 2/3(Server 模块 / 平台注册表,可并行) |
| P1 | 智能体 MVP 闭环:圈定 scope + 拆跨仓计划 | ask-first → plan-first | 地基之上;据 `requirements/mvp.md`(scope 待圈定);provision 通道是其身份地基 |
| P2 | 记录整体部署 / 反代拓扑(OQ-2) | plan-first | 4 仓 + 悟空IM 如何一起部署;补进 `architecture/system-baseline.md` |
| P3 | 校验 Server API 契约与 Web/Admin 消费一致 | implement | 对齐 `:8090/v1` 字段口径,受保护区 |

## 暂缓(pending 产品讨论)

- **身份统一**(`plans/2026-07-23-identity-unification.md`,原 P0):2026-07-23 用户叫停——先讨论清楚整个产品再回来。plan 已标 deferred,调研结果留存,Shape A/B 未决。

## Blocked / 待用户拍板(open questions)

- (当前无)

## 已解决

- **产品定义已成形(2026-07-23)**:讨论毕业为 `design/app-overview.md`(企业 AI 工作空间 · 每个员工的统一工作台)+ `requirements/mvp.md`(dogfood MVP)。轨迹见 `discussions/2026-07-23-product-definition.md`。
- **OQ-1(集成深度)已定**:「agent 里的 IM」,L2 为目标 / L1 只作跳板,IM 为唯一人机入口(取代 Matrix/Element/企微)。落地见 `plans/2026-07-23-identity-unification.md`;契约见 `architecture/system-baseline.md`。
- **OQ-3(差异化定位)已定**:做「agent 里的 IM」,不做「IM 里的 agent」——见 `architecture/project-vision.md`。("openclaw" 具体所指待补,不阻塞。)

## 备注

- open questions 按 AGE「未决问题一等公民」登记,不猜、不封口。
