# 计划:企业微信作为组织与用户来源(通讯录同步)

Plan Status: deferred

> **2026-07-24 暂缓**:用户拍板——先不做第三方(企微)用户同步。调研结果已完成并保留于下方,重启时可直接续用。
> 已定口径:**同步方式 = 每次全量对账**(不做事件回调,无公网依赖)。

## 背景与目标

- 服务端(OrgReOrgServer)以企业微信通讯录为**组织与用户的上游数据源**:部门树 + 员工全量同步进本地 organization 模块。
- 与 2026-07-23 拍板的「自有 IM 取代企微」**不冲突**:取代的是员工人机入口(surface);企微在此仅作组织数据 SoR。重启本计划时需把该定位补进 `architecture/system-baseline.md`,否则构成 drift。
- 与暂缓中的身份统一计划(`2026-07-23-identity-unification.md`)强相关:平台 person-id 种子恰為企微 userid;其调研指出 Server 缺通用 `(源系统,外部id,uid)` 映射表——本计划落地时应顺手补该地基(而非建企微专表)。

## Non-Goals

- 企微作为消息/会话通道(仅数据源,不做 IM 互通)。
- 事件回调增量同步(已拍板:全量对账即可)。

## 调研结果 A:Server 侧承接点(2026-07-24 核实)

- organization 模块已完备:`organization`(单企业 `org_default`)、`org_unit` 部门树(`parent_unit_id`,`unit_id` 业务键)、员工并入 `user` 表(`organization_id`/`employee_no`/`employment_status`/`title`)、`org_membership` 主/兼任关系(`is_primary`)。
- **落库必须走 `organization.Service` 现有方法**(CreateUnit/CreateMember/memberships/employment-status),不直写表——部门→IM群投影(`org_outbox`+projector)与 `organizationUpdate` CMD 推送自动触发,天然满足数据新鲜度约定。
- 定时设施三选:organization projector(outbox+5s 轮询)、timingwheel `Container.Schedule`(全量对账建议用它)、machinery 队列。
- OAuth 接入范式可参考 `modules/user/api_gitee.go`/`api_github.go`;`AddUser` 支持自定义 uid。
- 现状零企微/钉钉/飞书/LDAP 代码,外部通讯录同步为空白。

## 调研结果 B:企微 API 口径(2026-07-24 官方文档核实)

- **姓名拿得到**——用**自建应用 secret**(勿用「通讯录同步 secret」):
  - 自建应用调「读取成员」/「获取部门成员详情」**正常返回 name**;2022-06-20 收紧砍掉的是头像、性别、手机、邮箱、企业邮箱、个人二维码、地址(姓名不在列)。
  - 「通讯录同步 secret」2022-08-15 起对新 IP 只返回 userid+部门 ID(姓名/部门名全无)→ 不可用。
  - 第三方/代开发应用 name 返回 userid 代替 → 与我们无关(自建)。
- 可得字段覆盖需求:部门(名称/parentid/order)、成员(name、position→title、main_department→主部门、department+is_leader_in_dept→兼任/负责人、别名、启用状态)。
- **拿不到手机号/邮箱**(新建自建应用一律不给;需成员本人 oauth `snsapi_privateinfo` 授权)→ 同步员工无登录凭证,须走「用户名/工号+默认密码+强制改密」(`org_security_setting`+`password_change_required` 现成)或企微扫码登录(重启时拍板)。
- 实操前提:应用**可见范围设为全公司**;服务器出口 IP 配入应用「企业可信 IP」。
- 文档:获取部门成员详情 `developer.work.weixin.qq.com/document/path/90201`、读取成员 `/90196`、通讯录同步接口调整公告 `/96079`、概述 `/90193`。

## 方案骨架(已定部分)

1. 全量对账:定时(timingwheel)拉部门列表+逐部门成员详情 → 与本地 diff → 调 organization.Service 落库。无回调、无公网依赖。
2. 映射:通用 `user_external_identity(source, external_id, uid)` + `org_unit` 记 source/external_id(不建企微专表)。
3. 离职:企微删人 → `employment_status=0` 软离职,不硬删。
4. 编辑权:企微托管的部门/成员在 Admin 只读(沿用「组织架构托管群只读」先例);本地仍可建非企微账号(如 `category='agent'`)。

## 待拍(重启时)

1. 范围:纯同步 vs 同步+企微扫码登录(登录凭证问题,见调研 B)。
2. uid 策略:UUID+通用映射表(推荐,合身份统一调研「别把企微焊进 IM 身份」) vs 企微 userid 直作 uid(Shape A,牵动身份统一)。
3. 企微是否组织数据唯一权威(Admin 同步范围内只读)——建议是。

## Review Record

- (待填)

## Closure

- (未闭)
