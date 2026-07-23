# AI 自主策略与受保护区(根治理层)

默认自主度:`implement`(受下面受保护区 + freshness 限制)。

## 受保护区

| 区域 | 规则 | 需要的证据 |
| --- | --- | --- |
| 子仓内部代码(Server/Web/Admin/OrgReOrg 的 `src`) | 本层不改;交该仓自己的 harness | —— |
| 跨仓 API 契约(Server `:8090/v1` ↔ Web/Admin) | plan-first | 契约 diff、两个消费方影响、回滚 |
| 部署 / 反向代理 / 端口 / 域名 | ask-first | 拓扑对齐、回滚、冒烟 |
| 版本 / 子仓指针对齐 | plan-first | 各仓可达提交、对齐记录 |
| 密钥 / 凭据 / 真实组织数据 | blocked from git | ignored 路径、脱敏 |
| 上下文引擎 ↔ IM 集成(尚未建,OQ-1) | ask-first(产品决策) | 需用户拍板集成形态 |

无其它 → 记 `none`。

## 判据

发现自己要改某子仓 `src/` = 出界,停下交回该仓;**本层只碰 `docs/` 与跨仓协同**。
