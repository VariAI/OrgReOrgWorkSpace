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

### 管理端组织群只读契约

- `GET /v1/manager/group/list` 与 `/group/disablelist` 的群项返回 `category`；`GET /v1/manager/groups/{group_no}/members` 顶层同样返回 `category`。
- `category=department|team` 表示组织架构托管群。Admin 仅允许查看、发消息，不提供禁言、封禁/解禁或移除成员入口；组织管理页是名称与成员关系的事实源。
- Server 写接口继续拒绝组织架构托管群的手工修改，前端只读不是安全边界。

### 企业通讯录成员口径

- Admin 组织管理与 Web 企业通讯录的部门成员口径一致：展示该部门全部有效归属，包含主部门和兼任部门成员。
- Web 的组织架构视图包含当前登录员工本人；普通联系人列表、建群选人等需要排除自己的场景仍可使用去除当前用户的联系人口径。

## 集成契约(OQ-1 已定,2026-07-23 用户拍板)

- **方向**:agent 平台(OrgReOrg)= 主干/宿主;IM(Server/Web/Admin)= 平台的**原生 surface**。做「agent 里的 IM」,不做「IM 里的 agent」(见 `project-vision.md`)。
- **深度**:目标 **L2 原生 surface**、朝 L3 演进;L1 连接器/bot 仅作跳板。分水岭 = 平台拥有身份与上下文,IM 只渲染。
- **surface 唯一性**:自有 IM 为员工**唯一**人机入口,**取代 Matrix/Element/企微**(旧连接器逐步退,退役本身另立任务)。
- **四根缝**(受保护:凡跨仓接口 plan-first):
  1. 身份统一(先行,L2 地基)→ 立项 `plans/2026-07-23-identity-unification.md`
  2. 消息即证据(IM 会话 → 上下文引擎)
  3. 投影回 IM(上下文/知识/主动协助 → 客户端)
  4. agent 作一等参与者(不止 robot API)

## 契约:智能体 provision(缝 1 切片,2026-07-23 定稿)

> 来源 `../plans/2026-07-23-agent-provisioning.md`。身份统一缝的第一个落地切片;真人 SSO 后续复用同一链路。**受保护:改动 plan-first,评估平台与 Server 双方。**

**鉴权(双向)**:服务间凭证,HTTP Header `X-Service-Token`,两侧配置注入、不进 git,独立于用户 token 体系。

### A. 平台 → Server(新模块,路由前缀 `/v1/agents`)

**A1 `POST /v1/agents/provision`** — 创建智能体投影(幂等:同 `platform_id` 重复调用返回既有记录)

```jsonc
// req
{
  "platform_id": "ag_xxx",       // 必填。平台智能体ID,全局唯一,ag_ 前缀;
                                  // Shape A:直接用作 IM 的 uid/username
  "kind": "personal | project",
  "name": "小助理",               // IM 显示名
  "agent_no": "A0001",            // 智能体编号(展示)
  "display_unit_id": "",          // 可选:通讯录挂靠展示的组织单元
  "assignment": {
    "subject_type": "employee | project",
    "subject_uid": "<员工uid>",    // =employee 时必填
    "subject_ref": "<平台项目id>", // =project 时必填(IM 只存不解释)
    "group_nos": ["g1"]           // =project 时:要加入的群
  }
}
// resp
{ "uid": "ag_xxx", "status": "synced" }   // 无逐实例凭证:数据面走统一服务间 token
```

派生动作:`personal` → 建单聊会话 + 初始化;`project` → 加入 `group_nos` 群成员。

**A2 `POST /v1/agents/{platform_id}/deactivate`** — 停用(幂等):user 禁用、会话保留;`remove_from_groups` 默认 true(project 类退群)。

**A3 `GET /v1/agents/{platform_id}`** — 查询投影:`{uid, kind, status(active|disabled), assignment, synced_at}`。

**A4 数据面·出站(平台 → Server,同 `X-Service-Token`)**:
- `POST /v1/agents/{platform_id}/messages` — 发消息(薄包 WuKongIM 发送;字段:channel_id/channel_type/payload/stream_no?)
- `POST /v1/agents/{platform_id}/stream/start|end`、`POST /v1/agents/{platform_id}/typing` — 薄包 `internal/integration/wukongim` 既有 IMStreamStart/End、typing。

### B. Server → 平台(入职自动开通回环)

**B1 `POST <平台>/agents/register-for-employee`**(具体路径平台仓定,语义受保护):
req `{employee_uid, employee_no, name, organization_id}` → 平台建智能体档案 → 平台回调 A1 → resp `{platform_id}`。
触发:Server 员工建档领域事件(organization 模块**新增** `organization.employee.created`,建档与 CSV 导入路径统一发);agent 模块监听并调用,失败重试,按 `employee_uid` 幂等。

**B2 数据面·入站(Server → 平台单一 webhook,`X-Service-Token`)**:
`POST <平台>/agents/events` — Server 的 agent 模块挂 `ctx.AddMessagesListener`,过滤发给智能体的消息(单聊 to agent uid / 群内 @agent),推送事件 `{event_id, platform_id, message{...}}`;失败重试,按 `event_id` 幂等,事件内 `platform_id` 路由到具体智能体。**取代 TSDD robot 逐实例长轮询**(那是"少数第三方 bot"模型,不适配每员工一 agent 的舰队规模)。

### 数据面决策(2026-07-23,用户拍板)

**不复用、并计划删除 `modules/robot`(1518 行)**。理由:①全服务仅 bootstrap 一行引用,可整体摘除;②管道地基(消息监听 `AddMessagesListener`、发送、流式 `IMStreamStart/End`)在 base/WuKongIM 集成层,**不属于 robot 模块**,agent 模块直接用地基;③robot 的逐实例长轮询+app_key(绑 app 表)凭证模型不适配舰队规模,B2 单 webhook + 统一服务间 token 取代;④inline query/菜单/系统机器人种子均为包袱。`user.robot=1` 字段保留(属 user 表)。删除时机:agent 数据面跑通后(registry 一行 + 模块目录 + robot/robot_menu 表),删前核对系统机器人种子无消费方。

### 实现缺口备忘(2026-07-23 核实)

- `AddUser` 不设 `robot`/`category` → 需扩展入参或模块内自建插入(`robot=1, category='agent'`)。
- organization 模块目前无员工建档事件,需新增(见 B1)。
- Web 端无流式渲染代码(`streamNo` 零命中)→ 智能体会话打字机体验需后续补 Web 侧,不阻塞本契约。

## 仍开放

- **OQ-2:整体部署 / 反代拓扑**(4 仓 + 悟空IM 如何一起起、端口 / 域名怎么排)未记录。

## 非目标(当前)

- 不在本层重述各子仓内部实现。
- 不改子仓代码。
