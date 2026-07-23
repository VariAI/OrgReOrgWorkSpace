# 好友系统现状审计（2026-07-23）

> 结论：**前后端的好友"功能面"均已下线且口径一致**（后台砍路由、前端摘入口、双方把关系判定改成"企业目录内全员互通"）；但"代码面/数据面"均未清干净。本文记录截至 2026-07-23 静态核查（OrgReOrgWeb + OrgReOrgServer）的事实基线。

## 一、Web 端（OrgReOrgWeb）：入口全摘，底层残留

### 已脱离（UI 入口层面）

- **通讯录页已换成组织架构**：`/contacts` 路由挂的 `ContactsList`（`packages/orgreorgcontacts/src/Contacts/index.tsx`）拉取 `user/directory/units` 组织单元目录，不再是好友列表逻辑。
- **contacts 模块注册表无好友入口**：`packages/orgreorgcontacts/src/module.tsx` 目前只注册"保存的群"、"黑名单"和组织架构建群工具；"新的朋友"、"添加好友"均未注册。
- **`NewFriend/`、`FriendAdd/` 组件已成死代码**：文件仍在 `orgreorgcontacts/src/` 下，但包的 `index.tsx` 不导出，全仓无引用（仅互相引用），用户从任何界面不可达。
- `UserInfo`（用户资料卡）无加好友按钮或相关文案。

### 未脱离（底层链路残留，部分仍在"暗跑"）

- `packages/orgreorgbase/src/App.tsx`：仍保留完整 `FriendApply` 模型及 `getFriendApplys` / `addFriendApply` / `updateFriendApply` 等 localStorage 逻辑，以及红点接口 `/user/reddot/friendApply` 的调用。
- `packages/orgreorgbase/src/module.tsx`（约 277、302 行）：**仍监听好友申请/通过的 CMD 消息**——若服务端下发此类 CMD，会继续写 localStorage、更新未读红点，只是无 UI 消费。
- `packages/orgreorgdatasource/src/datasource.ts`：仍封装 `friend/apply`、`friend/sure`、`friends/{uid}`（删除）、`friend/remark` 四个 API。

## 二、服务端（OrgReOrgServer）：路由下线，数据面仍活跃

### 已脱离（对外接口层面）

- `modules/user/api_friend.go` 写有整套路由：`/v1/friend/*`（apply / sure / sync / search / remark / refuse）与 `/v1/friends/:uid`（删好友），**但 `NewFriend()` 在生产代码中无任何调用点**——`modules/user/module.go` 只注册了 `New(ctx)`（用户 API）与 `NewManager(ctx)`（管理后台）。运行中的服务上这些接口不存在（404）。仅 `friend_test.go` 还在使用。
- `modules/user/event_friend.go` 的好友确认/删除事件处理器挂在 `Friend` 结构上，随 `NewFriend` 未注册而一并失效。

### 语义切换：全员互通

- `modules/user/service.go:205`：`GetUserDetail` 给 `NewUserDetailResp` 传的 `follow` 参数**硬编码为 1**，用户详情不再查好友关系。
- `modules/user/api.go:286`：企业目录接口（`directoryUserResp`）同样硬编码 `Follow: 1`。

即服务端已改为"同企业目录内人人视同好友"，与 Web 端通讯录换成组织架构配套。

### 未脱离（friend 表 / friendDB 仍在活跃运行，非死代码）

- **注册流程**仍调 `addSystemFriend` / `addFileHelperFriend`，把系统号、文件助手写进 friend 表（`api.go:2408` 附近、`api_manager.go`）。
- **`webhook.go` 在线状态推送**仍按 friend 表查询推送对象（`getFriendUidsAndSetCache`，带 Redis 缓存）。
- `service.go` 的 `AddFriend` / `GetFriends` / `IsFriend`、`source.go`、`api_maillist.go` 仍引用 friendDB。

## 三、前后端对应关系

- Web 暗跑的红点请求 `/user/reddot/friendApply` 在服务端仍是活接口（`api.go:170` 通用 `reddot/:category` 路由），不报错，只是永远不会再有新的好友申请红点。
- Web datasource 里的 `friend/apply`、`friend/sure` 若被调用会 404——好在入口已摘，实际触发不到。

## 四、已知业务后果 ⚠️

**在线状态推送可能名存实亡**：普通用户之间不再互加好友后，`webhook.go` 按 friend 表推送的在线状态 CMD 实际只会推给系统账号那几条关系，**同事之间收不到彼此的在线状态**。若该功能仍需要，应改为按组织目录取订阅人。

## 五、彻底清除清单（如决定不再回退）

| 仓 | 待清理项 |
| --- | --- |
| OrgReOrgWeb | 删 `NewFriend/`、`FriendAdd/` 两个目录；删 base 里 FriendApply 存储/红点逻辑与 CMD 监听；删 datasource 四个好友 API |
| OrgReOrgServer | 删 `api_friend.go` / `event_friend.go` / `friend_test.go` / `swagger/friend.yaml`；`webhook.go` 在线推送改用组织目录；注册流程系统号关系另行存放（或保留 friend 表仅作系统号用途并注明） |

若保留回退可能，现状不影响运行；唯一建议留意的是 CMD 监听暗线（仍会实际发起红点 HTTP 请求）与在线状态推送的业务后果。
