# 管理端组织群只读

状态：completed

## 目标

部门群和团队群由组织架构投影生成，管理端群列表只允许查看、发消息，不提供手工禁言、封禁、解禁或移除成员入口。

组织管理页仍是名称、成员归属与群成员范围的事实源，不在本计划中限制。

## 契约 diff

- `GET /v1/manager/group/list`
- `GET /v1/manager/group/disablelist`
- `GET /v1/manager/groups/{group_no}/members`

前两者的 `groupManagerResp` 增加：

```json
{ "category": "department | team | ..." }
```

该字段只读、向后兼容；未知或空类别继续按普通群处理。

成员列表响应顶层同样增加 `category`，保证直接访问或刷新成员页时仍能可靠识别只读状态。

## 实施

1. Server 在后台群列表响应中透传 `group.category`，同步 Swagger，并加回归测试。
2. Admin 将 `department`、`team` 识别为组织架构托管群：
   - 列表展示类型标签与“组织架构管理”只读提示；
   - 不展示全员禁言、封禁/解禁操作；
   - 进入成员页时透传类别，不展示“移除”操作。
3. 保留 Server 已有的写接口拒绝逻辑作为最终安全边界。

## 消费方影响

- `OrgReOrgAdmin`：新增消费 `category`，用于只读交互。
- `OrgReOrgWeb`：不调用后台 `/manager/group/*` 接口，无行为变化。
- 旧版 Admin：忽略新增响应字段，兼容。

## 验证

- Server：相关 group 单测、`go build ./...`、`go vet ./...`。
- Admin：`pnpm lint`、`pnpm build`，核对普通群仍可操作、组织群只读。
- 根治理层：核对契约文档和当日日志一致。

## 回滚

先回滚 Admin 的类别判断，再删除 Server 响应字段与 Swagger 字段；Server 既有写接口保护不回滚。

## 完成记录

- Server 后台群列表、封禁群列表的 `groupManagerResp` 已返回 `category`，成员列表响应顶层也返回 `category`。
- Admin 已展示群类型；`department`、`team` 群的禁言、封禁、解禁、成员移除入口已替换为明确的组织架构只读提示。
- 验证通过：Server 精准回归测试、`go build ./...`、`go vet ./...`；Admin `pnpm lint`、`pnpm build`。
- 已知基线：`go test ./modules/group` 会因测试启动器重复注册 `/v1/manager/group/list` panic；本次新增的不依赖路由回归测试单独通过。
