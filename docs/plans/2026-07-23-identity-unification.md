# 计划:身份统一(第一根跨仓缝)

Plan Status: deferred
Source: `requirements/identity-unification.md`

> **2026-07-23 暂缓**:用户叫停——先把整个产品讨论清楚(`design/` `requirements/` 尚未成形),再回来推进身份映射(Shape A/B 未决)。下方调研结果已保留,可直接续用。

## Current Baseline

- OQ-1 已定:agent 平台为主干、IM 为原生 surface(L2 目标)、IM 为唯一入口。见 `architecture/project-vision.md` / `architecture/system-baseline.md`。
- 两侧身份模型细节未核实(Server 用户体系、平台人员注册表)。

## Goals

- 建立 IM 账号 ⟷ 平台人员的一一映射 + 单点身份解析。

## Non-Goals

- 权限投影、组织架构同步、旧连接器退役(各是单独缝/任务)。

## Task Route

- 类型:架构 + 跨仓契约(plan-first;受保护区:Server API 契约、身份)。
- 影响仓:OrgReOrgServer(身份接口)、OrgReOrg(人员注册/解析);Admin 只读关联。

## Execution

1. **调研(先行,只读子仓、不改)**:读 Server 用户/登录模型 + 平台 `workspaces/<person-id>` 注册表 + 现有企微 userid 用法;产出两侧身份模型对照 + 事实源候选。
2. 定身份事实源与映射位置(解 requirements 的两个 OQ)。
3. 定契约:Server「登录态换/校验平台身份」接口草案(先写契约进 `system-baseline.md`,不实现)。
4. 定新账号 → 平台人员注册触发路径(复用平台既有 register-person 能力?)。
5. 各子仓在自己 harness 内实现(本层只出契约 + 验收,不写子仓代码)。

## Verification Commands

- 契约/文档层:人工核 + 本层一致性。
- 子仓实现后:各仓自己的验证命令(见 `context/project-context.md`)+ 合成员工登录冒烟。

## 调研结果(2026-07-23,Execution step 1 完成)

**Server(IM)侧**:主键 `uid` = 服务端随机 UUID(不透明,是所有表的 join key);但 `modules/user/service.go:313 AddUser` **允许调用方传入自定义 uid**(目前只在测试调用、未暴露 HTTP)。Token = 缓存里不透明 UUID(`{uid,name,role}`),**同一 token 同时认证 HTTP API + WuKongIM**,无 JWT 签名。已有逐 provider 外部 id 列(wx/github/gitee/web3),但**无通用 `(源系统,外部id,uid)` 映射表**。已有企业字段 `OrganizationID`/`EmployeeNo`。

**平台侧**:`person-id` = 企微 userid localpart(如 `yangyijun`),**同时**用作 Matrix localpart `@yangyijun:server`。当前规范身份 = Matrix MXID,但 `<id>` 本身 = 企微 userid(种子)。ADR-099:真正绑定键 = `WORKSPACE_ID` == person-id,进程启动时绑定;**企微只是测试环境、非产品**,目标是 frontend-agnostic。

**★关键发现**:平台**已内建** frontend-agnostic 身份解析 —— `framework/projection/permission_view.py:334-356` 的 `external_ids: {frontend_type -> id}`,代码注释明说「新前端(matrix/feishu/**我们的 IM**)无需改引擎即可解析」。**但建好未填**:任何 committed person 记录都没 `external_ids`,`profile.json` 只有 `contact: "wecom://<id>"`。真实人员数据 gitignored,身份 system-of-record 落 runtime/`outputs` 或 `/data`,不进 git。

**候选映射形态**:
- **Shape A(同一 localpart,零映射表)**:IM 用户经 `AddUser(uid=<person-id>)` 建号 → IM uid == Matrix localpart == person-id,全 surface 一个身份串;平台 person 记 `external_ids["orgreorg-im"]`。最省、贴合现有 Matrix 约定、「登录 IM = 进入工作区」字面成立。
- **Shape B(解耦)**:IM 保留 UUID uid,平台 person `external_ids: {"orgreorg-im": "<uuid>"}`。直接用现成 resolver,但要建/填映射存储。

**待拍(requirements 的 OQ)**:
- 身份事实源:建议 = **平台 person**(与愿景「agent 里的 IM」+ ADR-099 frontend-agnostic 一致);IM 账号由平台 provision(`register_person.py` 加第 4 步)。
- person-id 语义:建议脱离「企微 userid」语义、视为**平台自有 principal id**(现恰好种子来自企微),别把被取代的企微焊进新 IM 身份。
- 映射形态:Shape A vs B → 待用户拍。

## Review Record

- (待填)

## Closure Gates

- requirements 验收标准全绿 + 两侧身份模型已记录 + 契约进 `system-baseline.md` + 合成冒烟通过。

## Closure

- (未闭)
