# 审计:OrgReOrgServer 后端(2026-07-23)

> 触发:用户担心「后台业务逻辑 + 继承的原有业务代码烂七八糟」。只读审计(2 个探查),不改代码。详细代码证据在子仓;本文是根治理层的结论存档。

## 结论(先给判断)

**不是「烂七八糟」(结构层面)。** 更准确:**「大而陌生 + 少量具体安全项」**。是一套维护中的生产 IM 后端 fork,不是拼凑原型。~49.5k LOC / 254 go 文件 / 35 个测试文件。

**「烂七八糟」的真实成分**:① 你没写、又大(陌生感)② 几处 TSDD 死代码残留 ③ 几个 2–3k 行的巨型 `api.go`(继承的)④ 一小撮明确的安全/卫生项。**没有一样需要重写。**

**深层问题(要不要换底座 / 自研核心)→ 不用。底座可信,建在上面即可,做一轮短的加固。**

## 结构(好)

- 插件式模块注册(`internal/bootstrap/registry.go` 单点挂载),DI 容器(非全局单例),模块契约统一(`api_/db_/service_` 约定真的贯彻),重复模块名会 panic。
- 真测试套件(35 个 `_test.go` + `testenv` 集成)+ 低债标记(49.5k LOC 仅 ~29 处 TODO/FIXME)。
- 消费级重货(朋友圈/红包/钱包)**已被 fork 砍掉**,只剩装饰性死引用。
- 模块化好砍:`workplace`/`report`/`qrcode`/`statistics`/`search`/`robot` 各删一行即可(零核心改动);`sms` 焊进登录(中等);`file`/`system`/`source` 是基础设施要留;`organization`(自加)隔离在配置开关后。

## 风险(真问题,按优先级)

1. **密码 `MD5(MD5(pwd))` 无盐 — 高。** 所有认证路径都用(`modules/user/{api.go,api_manager.go,api_usernamelogin.go,service.go}`)。企业产品这是 #1。→ 换 bcrypt/argon2id + 盐,登录时机会性迁移。
2. **提交的密钥 — 中,且违反你们自己 AGENTS.md。** `configs/tsdd.yaml`:redis `comtom@2019`、minio `orgreorg/orgreorg`。→ 清掉、轮换、走 `TS_*` env + 加 secret-scan hook。
3. **卫生两处**:`modules/group/db.go:372` 把 loginUID 直接拼进 SQL(来源是 token、可利用性低但坏范式);`pkg/util/md5.go:22` 有个 `fmt.Println` 把哈希输入打到 stdout。
4. **Admin 鉴权靠每个 handler 手动 `CheckLoginRole*`**(非中间件)→「漏写一个 = 一个没保护的管理接口」。→ 收进 `/v1/manager` 中间件。
5. **巨型 `api.go` 拆分**(group 2999 / user 2949 / message 2114 行)沿已有 `api_*.go` 切线增量拆 + 给最薄的关键路径补测试。这是从「还行」到「建得踏实」之间主要的那步。

## 建议路径

- **不重写、不换底座。** 先做 #1–#4 的「地基加固」(几天量,非重构);#5 增量长期做。
- 瘦身(砍 `workplace`/`report`/`qrcode`/`statistics`…)很便宜,可随手做——但先确认产品真不用(`robot` 可能留作 agent 通道)。

## 修复记录(2026-07-23,用户选 #1/#3/#4)

在 `OrgReOrgServer` 子仓实现,`go build ./...` + `go vet ./...` + `gofmt` 全绿,helper 单测通过。**认证/迁移全流程需连库冒烟——本机无 Docker,未跑。**

- **#1 密码(已修)**:新增 `pkg/util/password.go` —— bcrypt(先 sha256 预哈希规避 72B 上限)+ 兼容旧双重 MD5 的 `VerifyPassword` + `IsLegacyPasswordHash`;`password_test.go` 3 例过。17 处存储改 `HashPassword`、校验改 `VerifyPassword`;3 处登录加机会性迁移(旧哈希登录成功即升级 bcrypt);组织默认密码同改。**新增迁移 `modules/user/sql/user-20260723-01.sql` 把 `user.password` VARCHAR(40)→100**(bcrypt 60 字符,否则截断)——启动自动跑,须先连库。
- **#3 卫生(已修)**:删 `pkg/util/md5.go` 的 `fmt.Println`;`modules/group/db.go` 的 SQL 拼接改 `dbr.Expr(?)` 参数化。
- **#4 admin 鉴权(已修)**:新增 `pkg/wkhttp/http.go` 的 `AdminRoleMiddleware()`,挂到 7 模块的 8 个 `/v1/manager` auth 组(登录组不挂);handler 漏写 `CheckLoginRole` 也不再裸奔。随后按 DRY **删除这 7 组内冗余的 `CheckLoginRole()`**(中间件已覆盖),**保留全部 `CheckLoginRoleIsSuperAdmin`(更严一档)+ workplace/statistics/非 manager 路由(`*/api.go`)的检查**(中间件覆盖不到)。删除后 2 处 handler 需补 `var err error`,build+vet+gofmt 全绿。

**未做(用户未选)**:#2 提交的密钥、#5 巨型 api.go 拆分。

**待用户验证**:连库跑 `make env-test` + `go test ./modules/user/...` + 登录/改密/新员工默认密码 冒烟;确认迁移已应用后再上线。
