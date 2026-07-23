# 产品愿景与差异化(根治理层)

## 一句话

做 **「agent 里的 IM」**,不是 **「IM 里的 agent」**。

## 吸引子

产品本体 = **agent 原生的企业上下文管理平台**(OrgReOrg / Harness:evidence → ontology → 按任务/权限/成本投影)。
IM 是这个平台的**原生人机界面 / 载入口**——员工经 IM 进入的是 agent 的上下文世界,而不是「一个加了机器人的聊天软件」。

**IM 为唯一人机入口**:这套自有 IM(Server/Web/Admin)取代 Matrix/Element/企微,成为员工触达平台的唯一 surface——正因为 IM 自有,agent 才能深织进去(见下集成深度)。

## 差异化(non-goals)

- **不做「IM 里接入 agent」**:以 IM 为本体、把 agent 当成会话里的一个 bot / 插件挂上去(openclaw、Hermes(?) 一类)。那样 agent 是客人,受 IM 的数据模型与 bot API 约束,只能「贴」不能「融」。
- 我们**反过来**:agent / 上下文平台是**主干与宿主**,IM 是它的一个**原生 surface**。因为 IM 栈(Server/Web/Admin)是自有的,agent 得以**编织进** IM(原生消息类型、上下文投影、agent 作为一等参与者),而不是外挂一个 bot。

## 集成深度(已定,细化见 `system-baseline.md`)

**目标 L2「原生 surface」、朝 L3「agent-first」演进;L1「连接器/bot」只作技术跳板,绝不停在 L1**(停在 L1 = 白建 IM = openclaw/Hermes 那类)。L1 与 L2 的分水岭 = **谁拥有身份和上下文**:L2 里平台拥有,IM 只是渲染。

四根缝:

- **身份统一**(先行,L2 地基):IM 用户 ⟷ 平台组织/人员注册表,一套身份。
- **消息即证据**:IM 会话流入上下文引擎(取代 / 优于企微 msgaudit 那套)。
- **投影回 IM**:上下文 / 知识 / 主动协助投影进 IM 客户端。
- **agent 作为一等参与者**,而非仅 bot API 应答。

## 待澄清(次要)

- "openclaw" 具体指哪个产品 / 方案,待用户补(不影响上面的方向)。
