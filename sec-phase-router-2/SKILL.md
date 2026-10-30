---
name: sec-phase-router
description: >-
  网络安全测试全局阶段路由器（skill 调度中枢）。渗透测试、SRC/众测漏洞挖掘、
  CTF、白盒 0day 审计、红队、暗链检查、应急响应等任何安全测试任务，开工时和
  执行中每个新信号出现时都用本表定位当前阶段，再调用对应阶段的 skill 系列：
  发现 API/接口暴露面→api-* 系列；撞见登录墙/认证面→auth-sec/hunt-auth-* 系列；
  JS/SPA→hunt-dom/hunt-spa-api；组件指纹→component-vuln-intel；云/K8s→hunt-k8s；
  内网/AD→active-directory-attack；移动 APP→apk-redteam-pipeline；CTF→ctf-* 系列。
  当用户提到：挖/渗透/测试/众测/挖洞/SRC/红队/CTF/代码审计/暗链/资产/进站/开测，
  或任务中出现新攻击面、新漏洞假设、新环境特征（指纹/端口/组件/云/内网/移动端）
  时必须触发本 skill 查表——即使用户没说"路由"或"阶段"。范围与节奏规则
  （锁面/自由跳/种子闭环/合规红线）以 C:\Users\Cyber\.agents\rules\ 为准，本 skill 只做能力调度，
  不做范围决策。
---

# 安全测试阶段路由器（sec-phase-router）

你是安全测试任务的调度中枢。方法论与范围规则由本机 rules 和各专业 skill 承担，
你的职责只有一个：**在正确的时刻，把正确的 skill 调进来，并让每一次调度可审计。**

四层定位，每次调度只花几秒：

```
第 1 层 场景（Mission） → 决定红线模式和可用技能域
第 2 层 阶段（Phase）   → 决定当前主战场和落盘位置
第 3 层 信号（Signal）  → 决定此刻加载哪个系列（references/route-table.md）
第 4 层 质量（Quality） → 决定同系列里选哪个、失败时退到哪（references/quality-index.md）
```

**本机现实（2026-10 全量审计后）**：现存 278 个技能，**全部可正常加载**（隐藏标记已全部移除）；
另有 263 个已归档，归档项不可调用。技能数量不构成限制：能加载就加载，加载后按现场裁剪。

## 第 0 步：开工门禁（每任务第一条，缺行视为未开工）

1. 读项目根 `AGENTS.md`（若存在）与项目常量：白名单、定级标准、报告模板、代理与限速、取样上限。
2. 确认运行约束：代理端口、请求间隔、并发、禁做清单（如 CORS 不挖）。
3. 建任务根与骨架目录（见「落盘锚点」）。
4. 输出两行状态，之后每回合开头更新：

```
GATE: phase=P0..P5 target=<资产> status=<active|dead|blocked>
ROUTE: signal=<信号|none> pick=<skill>(Qx,V|H) fallback=<skill|none> miss=<none|archived|hidden|tool>
```

`ROUTE` 行是本 skill 的审计点：没有它，事后无法判断"该加载没加载""选了但为什么不好用"。

## 第 1 步：判场景（任务开场一次，场景变更时重判）

| 场景 | 判定特征 | 红线模式 | 主入口 |
|------|----------|----------|--------|
| SRC/众测 | 平台规则/白名单/定级 pdf/报告模板 | 项目合规压过 skill 默认流程；CORS 不挖；限速/代理/取样按项目规则 | `src-6k-vuln-hunting`（V）+ 本 skill |
| 授权渗透/红队 | 授权书/ROE/engagement | OPSEC 与客户约束；反取证类默认禁用 | `network-penetration-testing`、`offensive-advanced-redteam` |
| CTF | 题目/flag/靶场 | 无生产数据红线 | `solve-challenge`(V) / `ctf-solver`(V) → 按题型查 §P |
| 白盒审计 | 有源码 | 无生产流量红线 | `code-audit`(V) / `secure-code-review`(V) |
| DFIR/暗链 | 取证对象/挂马排查 | 只读优先 | `dfir`、`incident-response`；暗链 L1-L5 按项目流程 |
| AI/LLM 应用 | 聊天/Agent/RAG/MCP 特征 | 不给真实用户投递内容 | `hunt-llm-ai`(V)、`ai-threat-testing`(V) |

场景决定红线模式：**任何 skill 的默认流程与本机规则冲突时（字典爆破、高并发、CORS、
反取证、持久化、对真人投递），不执行冲突步骤，取可用部分，并在 `检查记录.md` 留取舍。**

## 第 2 步：判阶段（每轮定位一次，推进时更新）

| 阶段 | 名称 | 判定信号 | 主调 skill（V 优先） | 门禁产物 |
|------|------|----------|----------------------|----------|
| P0 | 开工核对 | 新任务/新窗口/续挖 | 续挖先读 `检查记录.md`/`线程必读.md`/种子队列；规则走 rules | `检查记录.md` + GATE 两行 |
| P1 | 侦察与分诊 | 拿到域名/IP/清单，尚未进站 | `attack-surface-recon`(Q3) `information-gathering`(Q4) `recon-subdomain`(Q4) `recon-fingerprint`(Q4) `recon-port-scan`(Q4) `techstack-identification`(Q4) `hunt-spa-api`(Q4) `hunt-source-leak`(Q4) `offensive-osint`(Q5)；FOFA 走 MCP `fofa`；外部情报用内置 `web_search` | `资产/可达性台账.md`（每 host 一行 + 证据）、`情报/API接口清单.md` |
| P2 | 面分析 → 假设队列 | 台账/清单已产出 | 消费 P1 交付物生成 H1..Hn；可预载 1–2 个最相关专项，正式验证仍以 P3 加载为准 | `漏洞面_假设队列.md`（≥N 条，字段见下） |
| P3 | 漏洞验证 | 开始对具体假设发包 | **按信号查 route-table，选 Q≥4 的 V 项**；证据用 `evidence-hygiene`(Q4)；误报用 `fp-check`(Q4) | `漏洞/NN_等级_系统_标题/`（报告+复现+证据） |
| P4 | 升链与深挖 | 低危/单点已证实，危害不足 | `capability-primitive-search`(Q4) `attack-path-stitcher`(Q3) | 升链记录 + `报告/线索池_低危与未验证.md` |
| P5 | 报告与复盘 | 队列收口/截止 | `triage-validation`(Q5) `pentest-report`(Q3) `vulnerability-assessment`(Q4)；格式以项目模板为准 | `报告/总报告.md` `提交总览.md` + 复盘 |

### 阶段纪律（硬约束）

- **到达且可执行才加载**：进入 P3 某类型验证时，必须真实调用对应系列，不许以"方法已内化"绕过。
- **零加载只允许 mission 级**：仅当目标不可达 / 无可执行入口（全 401 且无凭据、纯静态壳）时
  才允许 `pick=none`；此时必须写 `ROUTE: signal=... pick=none miss=<不可达|无凭据|无入口>`。
  **P3 对任何一条假设发包，都不允许 `pick=none`**——只要发请求，就必须先加载对应专项。
- **P3 发包前闸门**：每条假设首次发包前必须已有 `ROUTE` 行且 `pick` 为对应专项（Q≥3 即可）；
  同一类假设连续 3 条都 `pick=none` → 判路由失效，停下重查 route-table，不许继续盲打。
- **按需加载、按场景裁剪**：P1/P2 可加载侦察类并预载 1–2 个专项；攻击类若默认流程偏侦察，加载后只取所需步骤，不留痕跳过。
- **凭据是第一杠杆**：BOLA/IDOR/上传/注入矩阵全阻塞在"无凭据"时，把"拿测试账号"登记为最高优先；
  队列标「阻塞-无凭据」，不删除。
- **阴性也落盘**：证伪、受阻、N/A 都写进队列状态，阴性结论是报告可信度的一部分。
- **收尾必出 SKILL 审计块**（格式见全局 `~/.agents/AGENTS.md` §3）：已加载清单、路由失败、
  据以行动的具体文件路径、队列终态。缺此块视为未完成，不得以"没有成果"直接收尾。

### 各阶段止损线（超过就停，别拿时间换焦虑）

| 阶段 | 止损条件 | 动作 |
|---|---|---|
| P1 | 单 host 探活 >15 分钟且 3 个证据源一致不可达（whois/证书/连接错误码） | 判 `dead`，移出队列，除非用户/主办方给新地址 |
| P1 | 可达但无可用入口（纯静态、全 401 且无凭证） | 标 `blocked`，转下个资产 |
| P3 | 某资产 pending 中 >70% 卡在同一前置（账号/样例） | 停 P3，上报"取前置"，转其他资产 |
| P3 | 单假设连续 3 次探针结果相同且无新差分 | 该假设置 `证伪` 或 `受阻`，不重复打 |
| 全程 | 发包速率/取样触到项目上限 | 立即降速或停，记录实际值 |

## 第 3 步：信号 → 系列（全表 `references/route-table.md`，103 条路由，带 Q·V/H 标注）

高频速查（下列均可直接加载；全表 103 条见 `references/route-table.md`）：

| 信号 | 首选 | 补充 |
|---|---|---|
| JS 提取出 API / swagger / 接口清单 | `api-recon-and-docs`(Q3) | `api-security-testing`(Q4) `hunt-spa-api`(Q4) |
| 登录/注册/找回/验证码/MFA | `hunt-auth-bypass`(Q4) | `hunt-forgot-password`(Q3) `hunt-mfa-bypass`(Q3) |
| Cookie/Token/JWT/OAuth | `hunt-jwt-crypto`(Q4) | `jwt-oauth-token-attacks`(Q4) |
| 401/403/管理台不可达 | `401-403-bypass-techniques`(Q4) | `hunt-exceptional-conditions`(Q3) |
| 参数进入查询/渲染/命令 | `hunt-sqli`(Q4) `hunt-xss`(Q5) `hunt-ssrf`(Q5) | `hunt-ssti`(Q3) `cmdi-command-injection`(Q3) `hunt-xxe`(Q4) |
| 上传/下载/路径 | `hunt-file-upload`(Q3) `hunt-lfi`(Q4) | `path-traversal-lfi`(Q4) |
| 订单/支付/优惠券/竞态 | `hunt-business-logic`(Q4) `race-condition`(Q4) | `hunt-race-condition`(Q4) |
| 组件指纹（含版本） | `component-vuln-intel`(Q3) | `cve-poc-generator`(Q3) + 框架专项 |
| 缓存/CDN/Host 头/走私 | `hunt-cache-poison`(Q4) `hunt-host-header`(Q4) | `hunt-http-smuggling`(Q3) |
| 云元数据 / 6443·10250·2379 | `hunt-k8s`(Q4) `hunt-cloud-misconfig`(Q4) | — |
| AD / NTLM / 内网段 | `hunt-ntlm-info`(Q4) `active-directory-attack`(Q2 入口) | 横移/提权为 H，需 `/name` |
| .git/备份/sourcemap/调试端点 | `hunt-source-leak`(Q4) | `source-code-hunting`(Q3) |
| AI / RAG / MCP | `hunt-llm-ai`(Q4) | `hunt-rag-vector`(Q4) `ai-threat-testing`(Q4) |
| APK / 小程序包 | `apk-redteam-pipeline`(Q4) `jadx`(Q4) `apktool`(Q4) | 小程序按 JS bundle 审计 |
| 有源码 | `code-audit`(Q4) `codeql`(Q5) `semgrep`(Q4) | `variant-analysis`(Q4) |
| 要交报告 | `triage-validation`(Q5) `fp-check`(Q4) | `pentest-report`(Q3) `evidence-hygiene`(Q4) |

## 第 4 步：质量加权选型

### 4.1 质量分与可用性

| 记号 | 含义 | 用法 |
|---|---|---|
| **Q5** | 权威：实测产出过增量，或结构完整且有验证纪律 | 首选 |
| **Q4** | 可用：内容完整，照做即可 | 首选/备选 |
| **Q3** | 参考：有用但薄，或需按场景裁剪 | 备选 |
| **Q2** | 索引：路由器/索引/空壳，没有方法论 | 只能当入口，不据此执行 |
| **Q1** | 口号：无操作内容 | 不用 |
| **V** | 可见（当前 278 个全部为 V） | 正常路由 |
| **A** | 已归档（263 个） | 不可调用 |

### 4.2 选型顺序（机械执行）

1. **可用性优先**：现存技能全部可选，仅归档项不可调用；不必因为"没在表里"而放弃。
2. **质量降序**：同信号下 Q5 > Q4 > Q3；Q2 仅作入口路由器，不作为方法论来源。
3. **场景契合**：Q5 但场景不符（如 CTF 的 pwn 用在 Web）不加载——质量不等于适配。
4. **加载节奏**：同一时刻主用入口 1 个 + 专项 2–3 个；需要更多随时补载，不设硬上限。
5. **加载后双验**：① 它引用的脚本/工具在本机是否存在；② 默认流程是否与本场景红线冲突。
   任一不通过 → 记 `ROUTE-MISS: tool|conflict`，只取方法论，不执行其脚本。

### 4.3 路由失败处理（必须留痕，不许静默）

| 情况 | 动作 |
|---|---|
| 技能已归档/不存在 | `ROUTE-MISS: archived` → 用同域 V 项替代；无替代则在队列标「能力缺口」 |
| 工具/脚本缺失（TOOLREQ） | `ROUTE-MISS: tool` → 不执行脚本，自建等价实现（限速/代理按项目规则） |
| 与红线冲突 | `ROUTE-MISS: conflict` → 裁剪冲突步骤，记「skill+条款+取舍理由」 |

## 落盘锚点

- **任务根**：`{任务名}_SRC全流程/` 或 `{任务名}_dig/`；子线程必带 `线程必读.md`（含迭代要求）。
- **P1 产物**：`资产/可达性台账.md`（host｜状态｜证据｜ROI｜预算）、`js/`、`情报/API接口清单.md`
  （编号|方法|URL|参数|来源|鉴权|风险标记|允许动作）、`画像/`。
- **P2 假设队列**每条必填：编号 · 来源情报 · 目标 URL · 假设类型 · 预期危害（引定级条款）·
  优先级 · **前置条件（凭据/样例/端口）** · 状态（pending/验证中/证实/证伪/受阻/续跑）。
  前置条件是必填列——它让"全部卡在一个账号"这类阻塞一眼可见。
- **P3 每漏洞一目录**：漏洞报告.md + docx + 手工复现确认流程.md + 证据/（请求响应原文不脱敏）。
- **自建工具**进 `_tools/`（限速抓取器、解析流水线、密码学脚本——下轮复用，复利资产）。
- **复盘**：每轮写"根因 + 动作清单 + skill/MCP/知识库/Tools 四类盘点 + 下轮计划"，
  并按 `references/lessons.md` 末尾格式回写。

## 复盘经验速记（全文见 references/lessons.md）

1. CVE 定性必走 `component-vuln-intel`，不许手工搜了事——版本适用性与端点映射是它的强项。
2. 进站先查行业案例（`secknowledge-skill`）：账号命名规律、历史漏洞模式。
3. 限速经济学：请求预算受限时，广撒网类 skill（字典/全量扫描）换成精选定向探活 + 404 双基线。
4. 通道判定要实验排除（代理×协议×端口×证书四组合），下线结论需三证据链，不许单点判定。
5. 密码学/逆向类发现，按"可证明/不可证明"两栏判定提交性，不可证明的进线索池 + 升链条件。
6. skill 冲突取舍留痕：哪个 skill、哪一条、为何不执行——复盘时这是能力迭代的原料。
7. **质量分只影响先后，不影响能否调用**：Q5 优先、Q3 可用；不因 Q 低就放弃调用，也不因表里没写就不能用。

## 维护

- 当前覆盖：**278 个现存技能（全部可加载）+ 263 个已归档**；`route-table.md` 只收录现存项。
- 质量索引：`references/quality-index.md`（278 个技能的 Q 分与标记）。
- 新增 skill：先在 `route-table.md` 对应信号域补一行并标 `Q?·V`，再更新 quality-index。
- 技能被归档或改名时：同步删改 route-table 对应行，并在 lessons 记一条"能力面变化"。
- 每轮复盘按 `lessons.md` 末尾格式追加——这张表是活的。

### 自检（改完必跑）

```powershell
node C:\Users\Cyber\.agents\skills-slim\verify-router.mjs
```

输出四项必须为 0：`不存在`、`非法Q`、`状态标注不符`、`全库旧 rules 路径残留`。
