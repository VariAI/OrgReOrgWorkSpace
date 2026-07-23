# 讨论：智能体身份层建模（2026-07-23，已定稿）

> 背景：做"每个员工一个个人 AI 助手 + 每个项目一个项目 agent"。用户确认**不走好友体系**（好友系统已下线，见 `../audits/2026-07-23-friend-system-status.md`）。本文记录身份层的建模讨论与四项决策，作为后续 plan 的输入。

## 乱源诊断：三个正交概念被"身份"一词混住

| 概念 | 回答的问题 | 真人对应（现有表） |
| --- | --- | --- |
| ① 通信身份 | 怎么收发消息 | `user` |
| ② 组织身份 | 在企业里"是谁" | `employee`（只存任职属性） |
| ③ 服务关系 | 为谁工作 | `organization_membership` |

**核心结论：智能体照抄真人的三层同构**，不发明新模式：

```
真人:      user ──────── employee ──────────── organization_membership
智能体:  user(robot=1) ── 智能体档案(平台) ── assignment(服务于 员工X/项目Y)
```

现有地基：`user` 表已有 `robot` 标记与 `category` 先例（system/customerService）；`modules/robot/` 提供引擎接入凭证与完整 API（事件长轮询/sendMessage/流式/@提及）——robot 表是"工牌门禁卡"，不承担组织档案职责。

## 四项决策（用户 2026-07-23 拍板）

| # | 决策 | 结论 | 理由/代价 |
| --- | --- | --- | --- |
| D1 | 账号粒度 | **一实例一 IM 账号**（含每人的个人助手） | 进群/权限/审计全部复用现有用户与群成员语义；几百 robot 用户无成本。代价：引擎侧按实例管理凭证。 |
| D2 | 档案建模 | **独立表，不并入 employee**（技术决策，由 Claude 推荐定案） | employee 是 HR 语义（统计/CSV 导入/主部门约束），混入即处处过滤；两类实体字段无交集（owner/capabilities vs 入转调离）；且 D4 决定 IM 侧只是投影表，与 employee 天生异类。"一等身份"落在通讯录可见+同套权限审计，不落在同表。 |
| D4 | 事实源 | **平台（OrgReOrg）为主，IM 存投影** | 与身份统一缝"平台拥有身份，IM 只是渲染"一致；与 `organization_unit_group` 投影模式同构。硬约束：真人与智能体事实源方向必须一致。代价：先要打通平台→IM 同步链路。 |
| D5 | 个人助手开通 | **入职自动开通** | 监听员工建档/注册事件自动创建助手+初始会话。⚠️ 旧"注册→系统账号白名单"链路（`event_friend.go handleUserRegister`）已随 NewFriend 未注册而死亡，须在新模块干净重建，不复用。 |

D3（服务关系）随讨论定型：统一 `assignment` 表（subject_type ∈ {employee, project} + subject_id）。注意 **IM 侧没有"项目"实体**（工作主题是平台侧跨群聚合），项目 agent 的绑定对象是平台 ID，IM 侧只体现为"robot 是哪些群的成员"——这恰好落实"按群授权、默认读不到入群前历史"。

## 定稿模型

**平台侧（事实源）**：智能体注册表——编号（独立段，如 A0001）、owner、capabilities、scope、模型配置、状态、审计日志。形态归平台仓。

**IM 侧（全部是投影/凭证）**：
1. `user` 一行：`robot=1`，建议新增 `category='agent'`（沿用 category 先例）。
2. `robot` 一行：引擎接入凭证（现成机制，robot_id+app_key）。
3. **智能体投影表**（新）：platform_id ↔ uid 映射、展示字段（编号/名称/挂靠展示单元）、同步状态（pending/synced/error，仿 organization_unit_group 收敛模式）。
4. **assignment 投影**：personal → 派生动作 = 自动建单聊会话（ChannelTypePerson，全员互通已不依赖好友）；project → 派生动作 = robot 加入项目关联群的群成员。

**生命周期**：员工建档 → 平台建智能体 → 同步 IM（user+robot+投影+初始会话）；停用 → robot 禁用、会话保留；员工离职 → 助手归档、记录留审计；项目结束 → agent 退群归档。

**通讯录呈现**：展示层合并——智能体作为通讯录独立分区或挂靠展示单元（由投影表展示字段决定），Web 端在现有组织架构树上加分区。

## 明确不做

- 不写 friend 表（不复制 `addSystemFriend`/`addFileHelperFriend` legacy 模式）。
- 不在 employee 表加 kind。
- 不共享账号（包括个人助手）。

## 下一步

展开为 `plans/` 跨仓计划：平台注册表形态 + 平台→IM 同步契约（受保护区）+ Server 投影表/事件链路 + Web 通讯录分区与会话入口 + 引擎按实例凭证接入。

## D6 命名决策(2026-07-23 追加,用户拍板)

「数字员工」更名为「**智能体 / Agent**」。理由:①名字带"员工"与 D2 决策(不是 employee、不进 employee 表)语义打架,employee/digital_employee 长期混淆;②"数字员工"在国内企业软件语境被 RPA 厂商占用,暗示"替代人的自动化",与本产品"只建议不执行、human-in-the-loop、有真人 owner"相反;③过度拟人,责任归属应始终在人。
落点:与架构语言("agent 里的 IM"、缝 4「agent 作一等参与者」)同一套词汇;个人实例的显示名仍可叫"我的助理"。代码约定:模块 `modules/agent`、uid 前缀 `ag_`、`category='agent'`、编号 `agent_no`(A0001 段)。已全量更新:app-overview、mvp、system-baseline 契约、本文、plan、backlog(product-definition 存档保留原词+后记)。
