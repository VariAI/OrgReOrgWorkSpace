# 文档路由(根治理层)

本层用 AGE 治理 4 个子仓,收敛到「企业级上下文管理,IM 为载入口」。找**最小拥有答案**的文档:

| 需要 | 先读 |
| --- | --- |
| 本层是什么、规则、验证命令 | `context/project-context.md` |
| 能不能自主改、受保护区 | `context/ai-autonomy-policy.md` |
| 4 个仓各是什么、怎么协同、入口 | `context/codebase-map.md` |
| 产品愿景与差异化(agent 里的 IM) | `architecture/project-vision.md` |
| 目标产品形态 / 功能 | `design/app-overview.md` |
| 第一版 MVP 边界 | `requirements/mvp.md` |
| 跨仓拓扑、边界、集成契约 | `architecture/system-baseline.md` |
| 计划(跨仓契约/顺序/验证/回滚) | `plans/` |
| 产品讨论轨迹 | `discussions/` |
| 下一步做什么、未决问题 | `backlog/README.md` |
| 每天做了什么 | `logs/2026/` |

## 所有权

- `context/` 约束本层工作。
- `architecture/` 记录当前 4 仓拓扑、愿景与集成事实(接受变更后更新)。
- `design/` 记录目标产品形态与功能(接受变更后更新)。
- `requirements/` 记录已确认的产品行为与 MVP 边界。
- `plans/` 记录跨仓契约、顺序、验证与回滚(一轮一文件)。
- `discussions/` 存产品/需求澄清的决策轨迹。
- `backlog/` 起工作 + 存未决问题。
- 子仓内部事实由各子仓自己的 `README`/`AGENTS` 拥有,本层不复制。

已激活:`design/` `requirements/` `plans/` `discussions/`。按需激活(用到再建):`audits/` `testing/`。
