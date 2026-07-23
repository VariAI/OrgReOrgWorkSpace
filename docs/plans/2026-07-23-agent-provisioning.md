# 计划:智能体 provision 通道(平台→IM,身份统一缝的第一个落地切片)

Plan Status: in progress(Execution 1✅ 契约;2✅ Server `modules/agent`;3✅ 平台侧 `orgreorg_im` 连接器(external_ids 回填除外,推迟到身份统一);待 4/5 部署联调冒烟)
Source: `../discussions/2026-07-23-agent-identity.md`(身份层四决策已拍)+ `../requirements/identity-unification.md`(本 plan 是其真子集)+ `../design/app-overview.md`(智能体对象模型)

> 定位:**不是绕开身份统一缝,而是它的第一个落地切片**。完整缝的主体是真人 SSO(登录态互换、历史账号映射),智能体只需要其中一小段——平台创建/停用智能体时,IM 侧出现/禁用对应账号。契约方向、事实源方向(平台为主)、投影模式与完整缝同向,后续真人身份映射直接复用本切片打通的链路。

## Current Baseline(2026-07-23 核实)

- **平台↔IM 零对接**:Server 无任何平台集成代码;`modules/openapi` 只有 OAuth 三接口;robot 仅内部种子创建(`insertSystemRobot`),无外部开号通道。身份统一 plan deferred,仅完成调研。
- **现成钩子**:① Server `AddUser` 支持自定义 uid(仅测试调用、未暴露 HTTP)→ Shape A 的落点;② 平台 `permission_view.py` 的 `external_ids` frontend-agnostic 解析器建好未填;③ 管道地基在 base/WuKongIM 集成层:消息入站 `ctx.AddMessagesListener`、发送、流式 `IMStreamStart/End`(`internal/integration/wukongim/stream.go`)——**不属于 robot 模块**,agent 模块直接取用。
- **投影先例**:`organization_unit_group`(组织单元是事实源,IM 群是投影,带收敛状态)。
- **不可复用**:好友体系已下线(`../audits/2026-07-23-friend-system-status.md`);旧"注册→系统账号白名单"链路已死(`event_friend.go` 随 NewFriend 未注册失效),须新模块干净重建。
- **已拍决策**(discussions 文档):一实例一账号 / 档案独立不并 employee / 平台为事实源 IM 存投影 / 个人助手入职自动开通。
- **robot 模块决策(2026-07-23 用户拍板,改判):不复用、计划删除 `modules/robot`(1518 行)**。核实依据:仅 bootstrap 一行引用;管道地基不在其中(见上);其逐实例长轮询+app_key(绑 app 表)是"少数第三方 bot"模型,不适配每员工一 agent 的舰队规模;inline query/菜单/系统机器人种子皆包袱。数据面改为 B2 单 webhook 入站 + A4 出站端点(见 system-baseline 契约)。`user.robot=1` 字段保留(属 user 表)。Web 端对 robot 字段仅透传(`orgData.robot`)、无"机器人"UI 渲染,删除无前端影响。删除时机:agent 数据面跑通后;删前核对系统机器人种子无消费方。
- **已知缺口(不阻塞本 plan)**:Server 流式消息 API 完整,但 **Web 端无流式渲染代码**(grep `streamNo` 零命中)——agent 会话打字机体验需后续补 Web 侧,记入 Execution 4 之后的 UI 任务。

## Goals

1. 平台→IM provision 通道最小可用:平台创建智能体 → IM 自动出现智能体账号(user+投影行);停用同理。
2. 个人助手闭环:员工建档 → 自动开通助手 → 员工会话列表出现"我的助手"单聊 → 发消息 → 平台经 B2 webhook 收事件、经 A4 端点回消息。
3. 项目 agent 可入群:assignment(project) → 智能体成为指定群成员。

## Non-Goals

- 真人 SSO / 登录态互换 / 历史账号映射(完整身份统一缝,后续)。
- 权限投影细粒度、跨群读审批、智能体的建议/确认业务流(智能体 MVP 闭环,另一层)。
- 组织架构同步缝、Web 通讯录"智能体分区"UI(呈现增强,可后置为子仓任务)。
- Admin 智能体管理控制台(一期只读列表即可,甚至可缓)。

## Task Route

- 类型:架构 + 跨仓契约(plan-first;受保护区:Server API 契约、身份)。
- 影响仓:OrgReOrg(事实源 + provision 调用方 + 引擎接入)、OrgReOrgServer(provision 接口 + 投影表 + 派生动作)、OrgReOrgWeb(仅验证呈现)、OrgReOrgAdmin(可选只读)。

## 关键设计约定(进契约前先固定)

- **ID 形态:智能体先走 Shape A**——平台智能体 ID 直接作 IM uid(经 AddUser 自定义 uid 钩子),带前缀段防碰撞(如 `ag_`);平台侧同时登记 `external_ids["orgreorg-im"]`(把建好未填的解析器用起来)。智能体无历史 uid 负担,此选择不预决真人侧的 Shape A/B。
- **鉴权:服务间凭证**(平台↔Server 专用 secret/token,独立于用户 token 体系),配置注入、不进 git。
- **账号语义**:`user` 行 `robot=1` + 新增 `category='agent'`。**无逐实例凭证**:数据面双向统一走 `X-Service-Token`(robot 表/app 表不参与)。
- **投影表**(Server 新模块,仿 organization_unit_group):platform_id ↔ uid、展示字段(编号/名称/挂靠展示单元)、assignment(subject_type ∈ {employee, project} + subject_id)、同步状态(pending/synced/error)。
- **派生动作**:assignment=employee → 建单聊会话 + 必要白名单(新实现,不碰 friend 表);assignment=project → 加入指定群成员(群由平台在请求中给出 group_no 列表,IM 不理解"项目")。

## Execution

1. **定契约(本层,先行)** ✅ 2026-07-23 完成(同日修订):契约(A1 provision / A2 deactivate / A3 query / **A4 出站数据面** + B1 入职回环 / **B2 入站 webhook**)已定稿进 `architecture/system-baseline.md` 受保护区,含实现缺口备忘(AddUser 不设 robot/category;organization 无员工建档事件需新增;Web 无流式渲染)。要点——
   - 平台→Server:`provision / deactivate / query` 智能体(入参:platform_id、类型、展示字段、assignment;出参:uid、status,**无逐实例凭证**)+ A4 messages/stream/typing。
   - 入职自动开通的触发回环(一期最简):Server 监听自身员工建档事件 → 调平台"为员工 X 注册个人助手" → 平台创建智能体 → 回调 Server provision。平台侧员工注册表尚未同步(组织同步是另一根缝),故一期由 IM 侧事件驱动、平台被动登记,方向仍是"平台拥有档案"。
2. **Server 实现** ✅ 2026-07-23 完成(明细):
   - 新模块 `modules/agent`(model/db/api/event/inbound/module + sql/agent-20260723-01.sql 投影表 + agent_test.go 5 个单测)。
   - A1/A2/A3:`X-Service-Token` 常量时间比较鉴权;provision 幂等(先查投影行);建号直插 `user.Model{Robot:1, Category:'agent'}`(新增 `user.CategoryAgent` 常量);派生动作**先于投影行落库**保证重试可整体重放,欢迎消息用确定性 `client_msg_no`(md5 agent-welcome:platform_id)防重复。
   - A4:messages/stream/typing 薄包 wukongim 集成层,校验智能体启用态。
   - B2:`AddMessagesListener` 过滤(单聊解 fake channel;群聊按 mention.uids **支持多 @**,优于 robot 只支持单 @)+ Redis ZSet 全局队列(score=GenSeq event_id)+ 2 秒 tick 顺序推送,网络/5xx 中断保序重试、4xx 出队、损坏条目 ZRem 单删;智能体自身消息不回流防循环;`platform.apiURL` 为空整条链路静默。
   - B1:监听新事件 `organization.employee.created`(base/event 新增常量;organization 的 `CreateMember` 与 CSV `CommitImport` 两路径在事务内 EventBegin、提交后 EventCommit,已是成员的更新分支不触发);失败语义=网络/5xx 不 commit 走 60 秒兜底重投、4xx 终态。
   - 群接缝:group 模块新增 `JoinGroup/LeaveGroup`(幂等,含 IM 订阅同步,复用部门群投影的 member 事务原语),入 IService。
   - 配置:`Platform{APIURL, ServiceToken}` + viper 读取 + 两个 yaml 占位(空值,生产用 env 注入)。
   - 验证:`go build ./...` + `go vet ./...` 全绿;单测 5 个通过(迁移文件清单/入参校验/单聊与群 @ 过滤/gin 路由树静态+参数段并存防 panic)。**未跑连库冒烟(本机无 Docker)**。
3. **平台实现** ✅ 2026-07-23 完成(明细):
   - 新连接器 `framework/connectors/orgreorg_im/`(client/registry/service/http_server + `scripts/orgreorg_im_server.py` 壳),完全循平台既有形态:纯逻辑进 framework、socket 壳进 scripts(仿 wecom_callback_server)、stdlib urllib + 可注入 transport(复用 matrix.client 的 `JsonHttpTransport`,防烟囱)、一个进程服务全部智能体(仿 ADR-086 舰队)。
   - B1 接收:`POST /agents/register-for-employee` → 注册表建档(platform_id 确定性 = `ag_`+员工uid,幂等零记账;agent_no 自增 A0001 段)→ 回调 Server A1 provision → 成功标 synced;5xx 让 Server 事件总线重投。
   - B2 接收:`POST /agents/events` → event_id 去重(有界 seen 集)→ 单聊回发送者/群回群,经 A4 发**确定性最小回复**;回复失败不记 seen(至少一次)。
   - 注册表 = 平台侧事实源:`workspaces/variai/registry/im-agents.json`(**gitignored**,含真实员工标识;原子写复用共享件 `ledger_io.write_ledger`)。
   - 鉴权:双向 `X-Service-Token`(常量时间比较);env 注入(`ORGREORG_IM_SERVICE_TOKEN`/`ORGREORG_IM_API_URL`),token 未设拒绝启动。
   - 顺手修:共享件 `ledger_io` 模块级 `import fcntl`(Unix-only)改条件导入,非 Unix 锁退化 no-op(degrade-not-fail;生产 Linux 行为不变),其 21 个既有测试全过。
   - 验证:新增 16 个单测全过;shared_helper / package_boundary / timeout / conflict_marker 四个门禁 lint 全绿。**完整 `ci_check.sh` 需在 Linux 环境跑**(Windows 无 bash 门禁全链);`external_ids` 回填**推迟**——它需要"平台 person ↔ IM 员工"映射,那正是身份统一缝主体,本切片不含。
   - 部署备忘:平台侧起 `scripts/orgreorg_im_server.py`(建议 18130 端口,systemd 单元后续按仓内模板加);Server 配 `platform.apiURL=http://<平台机>:18130`。服务间内网直连,不经 Caddy。
4. **Web 验证(最小)**:员工登录后会话列表出现助手单聊、收发正常、头像/名称显示正确。通讯录分区不在本 plan。
5. **冒烟验收(合成数据)**:建一个合成员工 → 助手自动出现 → 互发消息 → 平台侧审计可见该智能体及其 external_ids;停用 → IM 侧账号禁用、会话保留。
6. **删除 `modules/robot`**(数据面跑通后,Server 子仓内):registry 一行 + 模块目录 + robot/robot_menu 表迁移;删前核对系统机器人种子(`insertSystemRobot`)无消费方。

## Verification Commands

- 契约/文档层:人工核 + 本层一致性(system-baseline 与本 plan 口径一致)。
- Server:`go build ./...` + `go vet ./...` + 模块单测;**注意本机无 Docker,连库冒烟需在有栈环境跑**(与后台审计修复时同一限制)。
- Web:类型检查基线"错误数不增"(见 OrgReOrgWeb/CLAUDE.md)。
- 端到端:上述第 5 步合成冒烟。

## Review Record

- (待填)

## Closure Gates

- 契约进 `system-baseline.md` 受保护区并与两侧实现一致。
- 合成冒烟通过(建档→助手出现→互发消息→停用)。
- friend 表零新增写入方(核对)。
- `modules/robot` 已删除且 `go build ./...` 通过(Execution 6 完成后)。

## Closure

- (未闭)
