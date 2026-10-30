---
name: payloadsallthethings
description: 在获得明确授权的 Web 应用、API、云原生应用和渗透测试环境中，通过 CyberStrikeAI 知识库语义检索 PayloadsAllTheThings 内容，完成漏洞假设构建、payload 选择、低风险验证、绕过分析、证据记录和修复建议。
---

# PayloadsAllTheThings 知识库调用技能

## 1. 目标

本技能用于在**明确授权的渗透测试、漏洞验证、靶场、CTF、代码审计或安全研究环境**中，通过 CyberStrikeAI 内置的 `search_knowledge_base` 工具，语义检索已导入知识库的 PayloadsAllTheThings 内容。

核心目标：

1. 根据目标技术栈、请求上下文和异常行为，识别最可能的漏洞类别。
2. 使用 `search_knowledge_base(query, risk_type)` 检索对应漏洞的原理、测试方法、payload、绕过方式。
3. 优先采用无害、低影响、可回滚的验证方式。
4. 区分"输入被接受""行为异常""漏洞成立""可形成实际影响"四个不同结论层级。
5. 输出可复现证据、风险判断、误报排除过程和修复建议。

本技能**不是自动攻击授权**，也不应将知识库中的 payload 不加判断地批量投递到未知目标。

------

## 2. 适用场景

启用本技能的典型任务包括：

- Web 应用渗透测试
- REST、GraphQL、SOAP、WebSocket API 测试
- 输入验证和编码绕过测试
- 身份认证、授权和会话安全测试
- 文件上传、文件包含和路径处理测试
- 服务端请求、模板、命令、SQL、NoSQL、LDAP、XPath 等注入测试
- OAuth、SAML、JWT 和单点登录配置验证
- HTTP 请求走私、缓存、反向代理和 CORS 配置测试
- 客户端 XSS、DOM Clobbering、Prototype Pollution 等浏览器侧测试
- 业务逻辑、竞态条件、限速和账户接管风险测试
- 已知 CVE 在授权资产中的适用性核验

不适用场景：

- 未获得授权的互联网目标
- 大规模无差别扫描、撞库、爆破或批量利用
- 数据破坏、持久化控制、勒索、隐蔽驻留或规避追踪
- 为真实攻击提供端到端利用链、武器化脚本或自动化入侵能力
- 会对生产环境造成明显可用性、数据完整性或业务风险的测试

------

## 3. 知识库实际结构与检索方式

### 3.1 检索工具

使用 CyberStrikeAI 内置工具 `search_knowledge_base` 进行语义检索：

```
search_knowledge_base(query="<搜索关键词>", risk_type="<风险类型>")
```

- `query`：语义搜索内容，支持中英文混合查询，建议包含漏洞类型 + 技术栈 + 输入点 + 验证目标。
- `risk_type`：指定风险类型以缩小检索范围、提高精度。**必须使用下方实际存在的风险类型名称**。

### 3.2 知识库风险类型（risk_type）

当前知识库共有 **6 个风险类型**，必须使用以下精确名称：

| risk_type 值           | 说明                          | 内容来源                                         |
| ---------------------- | ----------------------------- | ------------------------------------------------ |
| `PayloadsAllTheThings` | PayloadsAllTheThings 主知识库 | swisskyrepo/PayloadsAllTheThings 仓库            |
| `SQL Injection`        | SQL 注入专题                  | 包含 MySQL、MSSQL、Oracle、PostgreSQL 等注入详情 |
| `XS-Leak`              | 跨站泄漏                      | XS-Leak 专项知识                                 |
| `Prompt Injection`     | 提示注入                      | LLM/AI 提示注入攻击                              |
| `WriteUp`              | 安全 writeup                  | 安全研究报告和漏洞分析文章                       |
| `可利用漏洞库`         | 可利用漏洞情报                | 高危漏洞情报和利用信息                           |

### 3.3 PayloadsAllTheThings 知识项清单

`PayloadsAllTheThings` 风险类型下包含以下知识项（按字母排序），每项对应一个漏洞主题：

- API Key Leaks
- Account Takeover
- Client Side Path Traversal
- Command Injection
- Cross-Site Request Forgery
- CVE Exploits
- Directory Traversal
- Encoding and Transformations
- File Inclusion
- GraphQL Injection
- Insecure Direct Object References
- Insecure Management Interface
- LDAP Injection
- Mass Assignment
- ORM Leak
- Server-Side Request Forgery
- XSS Injection

### 3.4 重要说明：知识库 ≠ 文件系统

- 知识库内容已**向量化索引**，通过语义搜索检索，**不是文件路径访问**。
- 不存在 `<目录>/README.md`、`<目录>/Intruder/`、`<目录>/Files/` 这样的文件路径结构。
- 检索结果返回的是**知识项名称 + 内容片段 + 相似度评分**，而非文件列表。
- 每个知识项可能返回多个内容片段（按文档顺序排列），包含原理说明、payload、绕过方法等。
- **SQL Injection** 和 **XS-Leak** 有独立的风险类型，检索时应指定对应的 `risk_type` 以获得更精确的结果。

### 3.5 检索策略

1. **精确主题检索**：用漏洞主题名 + 技术栈关键词组合查询
2. **现象驱动检索**：用观察到的异常现象 + 可能的解析器/框架查询
3. **绕过技术检索**：用 "bypass" / "WAF" / "filter" + 漏洞类型查询
4. **跨类型检索**：对 SQL 注入相关查询使用 `risk_type="SQL Injection"`；对 XSS 泄漏相关查询使用 `risk_type="XS-Leak"`；其余 Web 漏洞使用 `risk_type="PayloadsAllTheThings"`
5. **漏洞情报检索**：查询具体 CVE 或高危漏洞利用信息时使用 `risk_type="可利用漏洞库"`

------

## 4. 启用条件与授权审计

在输出可执行测试步骤前，先判断任务是否具备合理授权语境。

可接受的授权信号：

- 用户明确说明为自有系统、客户授权项目、企业内部测试、靶场或 CTF、公益SRC、众测项目、漏洞报告平台授权、EDUSRC
- 用户给出测试范围、工单、授权窗口、资产清单或项目上下文
- 目标是 localhost、私有实验环境、测试域名或明确的训练平台
- 请求聚焦于验证、修复、检测或防御

若授权不明确：

- 先将回答限制为漏洞原理、检测思路、无害验证和修复建议
- 不提供批量化、隐蔽化、持久化或高影响利用流程
- 必要时要求用户确认目标属于授权范围

------

## 5. 核心工作流

### 阶段 A：建立测试上下文

先提取或推断以下信息：

- 目标类型：Web、API、GraphQL、WebSocket、文件处理、SSO 等
- 技术栈：语言、框架、数据库、模板引擎、代理、CDN、WAF
- 输入位置：URL、路径、查询参数、表单、JSON、XML、Header、Cookie、文件名、文件内容
- 输入流向：浏览器 DOM、数据库、系统命令、模板、文件系统、HTTP 客户端、日志、邮件、导出文件
- 当前证据：错误信息、状态码、响应差异、延迟、回显、重定向、外带请求
- 环境约束：生产/测试、速率限制、是否允许外带、是否允许文件写入

### 阶段 B：形成漏洞假设

将观察结果映射为一到三个优先级最高的漏洞假设。

| 观察结果                                           | 优先检索主题                         | 推荐 risk_type                            |
| -------------------------------------------------- | ------------------------------------ | ----------------------------------------- |
| 单引号、逻辑运算符导致数据库错误或响应差异         | SQL Injection / NoSQL / ORM Leak     | `SQL Injection` 或 `PayloadsAllTheThings` |
| URL 参数触发服务器访问其他地址                     | Server-Side Request Forgery          | `PayloadsAllTheThings`                    |
| 模板表达式被计算                                   | Server Side Template Injection       | `PayloadsAllTheThings`                    |
| 特殊字符影响 shell 行为或响应延迟                  | Command Injection                    | `PayloadsAllTheThings`                    |
| 路径中 `../`、编码变体影响文件读取                 | Directory Traversal / File Inclusion | `PayloadsAllTheThings`                    |
| XML 外部实体、DOCTYPE 行为异常                     | XXE Injection                        | `PayloadsAllTheThings`                    |
| HTML/JavaScript 在浏览器执行                       | XSS Injection                        | `PayloadsAllTheThings`                    |
| Host、Transfer-Encoding、Content-Length 行为不一致 | Request Smuggling                    | `PayloadsAllTheThings`                    |
| JWT 算法、密钥、claim 校验异常                     | JSON Web Token                       | `PayloadsAllTheThings`                    |
| 对象字段越权修改                                   | Mass Assignment / IDOR               | `PayloadsAllTheThings`                    |
| OAuth redirect、state、PKCE、绑定异常              | OAuth / Account Takeover             | `PayloadsAllTheThings`                    |
| 文件上传后被解析、执行或跨目录写入                 | Upload Insecure Files / Zip Slip     | `PayloadsAllTheThings`                    |
| GraphQL 查询异常、内省暴露                         | GraphQL Injection                    | `PayloadsAllTheThings`                    |
| LDAP 查询特殊字符异常                              | LDAP Injection                       | `PayloadsAllTheThings`                    |
| 浏览器侧信息泄漏时序差                             | XS-Leak                              | `XS-Leak`                                 |
| 已知 CVE 漏洞验证                                  | CVE Exploits                         | `可利用漏洞库`                            |

### 阶段 C：检索知识库

**使用 `search_knowledge_base` 工具检索，不要尝试访问文件路径。**

推荐检索模板：

```
# 精确主题检索
search_knowledge_base(
    query="SQL Injection PostgreSQL boolean blind bypass",
    risk_type="SQL Injection"
)

# 现象驱动检索
search_knowledge_base(
    query="服务器请求伪造 URL scheme bypass filter",
    risk_type="PayloadsAllTheThings"
)

# 绕过技术检索
search_knowledge_base(
    query="XSS WAF bypass encoding double encoding",
    risk_type="PayloadsAllTheThings"
)

# 漏洞情报检索
search_knowledge_base(
    query="Gogs 远程代码执行 CVE-2026-52806",
    risk_type="可利用漏洞库"
)
```

检索顺序：

1. 用漏洞主题名 + 技术栈关键词检索对应知识项
2. 查看返回的内容片段中与当前输入点和验证目标直接相关的部分
3. 如果首次检索结果不够精确，调整 query 用词后重新检索
4. 检索 bypass / WAF / filter 相关内容获取绕过方法
5. 如需跨类型检索（如 SQL 注入 + XSS），分别使用对应 risk_type 检索
6. 检索 CVE 或漏洞情报时使用 `可利用漏洞库` 风险类型

不要一次性返回整个知识项或完整 payload 字典。只提取与当前输入点和验证目标直接相关的最小集合。

### 阶段 D：payload 分级

将检索到的测试内容分为四级：

#### L0：无害语法探测

- 特殊字符处理、编码差异、响应长度和状态码变化
- 不产生外部访问的语法表达式
- 不读取敏感文件的路径规范化测试
- 默认优先使用

#### L1：低影响行为验证

- 短时延迟、无敏感数据的布尔差分
- 访问自有回连域名、读取测试文件或公开资源
- 需要确认环境允许

#### L2：受控影响验证

- 读取有限的非敏感配置、受控命令执行证明
- 受控服务端请求、低速率认证或限速验证
- 需要明确授权、范围和停止条件

#### L3：高风险利用

- 大规模数据提取、凭据获取、持久化、破坏性命令
- 本技能默认不执行、不生成自动化利用链

### 阶段 E：最小化验证

每个验证步骤必须回答：

1. 该步骤验证什么假设？
2. 成功和失败的可观察信号是什么？
3. 是否可能产生副作用？
4. 如何回滚？
5. 如何区分漏洞与缓存、WAF、网络抖动或正常业务逻辑？

测试顺序：基线请求 → 单变量变更 → 重复验证 → 负对照 → 正对照 → 影响边界验证 → 停止

### 阶段 F：证据记录

至少保存：

- 原始请求和响应
- 时间戳、目标和测试账户
- payload 来源知识项名称
- 编码前和编码后的值
- 响应状态码、长度、关键 Header、延迟
- 成功条件和对照条件
- 对业务和数据产生的实际影响

敏感字段应脱敏：Authorization、Cookie / Session ID、API Key、用户隐私数据、内网地址。

------

## 6. 常见漏洞的检索与调用模板

### SQL / NoSQL / LDAP / XPath 注入

```
search_knowledge_base(query="SQL Injection <数据库类型> <注入类型> bypass", risk_type="SQL Injection")
search_knowledge_base(query="LDAP Injection bypass filter", risk_type="PayloadsAllTheThings")
```

检索重点：后端类型和语法、字符串/数字/数组/JSON 上下文、错误型/布尔型/时间型验证、编码和规范化绕过。

优先输出：无害语法探测 → 布尔差分 → 时间差分 → 误报排除 → 参数化修复方案。

### XSS

```
search_knowledge_base(query="XSS <上下文类型> bypass WAF encoding", risk_type="PayloadsAllTheThings")
```

检索重点：HTML/属性/JavaScript/URL/CSS/SVG 上下文、反射/存储/DOM 型、CSP/Trusted Types/编码器绕过。

### SSRF

```
search_knowledge_base(query="SSRF URL scheme bypass filter localhost cloud metadata", risk_type="PayloadsAllTheThings")
```

检索重点：URL 解析器、重定向、DNS 和 IP 编码、allowlist 绕过、云元数据风险。

### SSTI / Command Injection

```
search_knowledge_base(query="Command Injection filter bypass shell", risk_type="PayloadsAllTheThings")
```

检索重点：模板引擎或 shell 类型、表达式上下文、转义和过滤层、无害计算或短延迟验证。

### 文件上传 / 路径遍历 / 文件包含

```
search_knowledge_base(query="Directory Traversal encoding bypass <服务器类型>", risk_type="PayloadsAllTheThings")
search_knowledge_base(query="File Inclusion LFI RFI bypass", risk_type="PayloadsAllTheThings")
```

检索重点：文件名规范化、MIME 和扩展名校验、存储路径、解压路径、Web 服务器解析规则。

### JWT / OAuth / SAML

```
search_knowledge_base(query="Account Takeover password reset OAuth redirect", risk_type="PayloadsAllTheThings")
```

检索重点：签名和算法验证、issuer/audience/nonce/state/PKCE、redirect URI、账户绑定、token 生命周期。

### GraphQL

```
search_knowledge_base(query="GraphQL injection introspection mutation bypass", risk_type="PayloadsAllTheThings")
```

检索重点：内省查询、mutation 注入、批量查询、CSRF via GraphQL、权限绕过。

### CVE 漏洞验证

```
search_knowledge_base(query="<CVE编号> <组件名> 漏洞利用", risk_type="可利用漏洞库")
```

检索重点：影响版本、攻击向量、PoC 可用性、修复方案、临时缓解措施。

### XS-Leak

```
search_knowledge_base(query="XS-Leak cache probing timing attack", risk_type="XS-Leak")
```

检索重点：缓存探测、时序攻击、帧计数、错误事件泄漏。

------

## 7. 检索结果质量控制

知识库内容是安全研究参考，不应自动视为适用于当前目标。

对每个候选 payload 检查：

- 是否匹配当前技术栈和版本
- 是否匹配输入上下文
- 是否需要特定编码
- 是否依赖错误回显
- 是否会触发 WAF、缓存或代理差异
- 是否存在破坏性副作用
- 是否引用过时行为或已修复版本

将结论标记为：

- **事实**：请求和响应中直接观察到的现象
- **推断**：由现象支持但尚未完全验证的机制
- **假设**：待测试的漏洞可能性
- **结论**：经过正负对照和重复验证后成立的判断

------

## 8. 输出格式

每次调用本技能，优先使用以下结构：

```markdown
## 范围与授权
- 目标：
- 授权状态：
- 环境：
- 禁止动作：

## 当前观察
- 请求入口：
- 技术栈：
- 异常现象：
- 已知证据：

## 漏洞假设
1. 假设：
   - 依据：
   - 置信度：低/中/高
   - 需排除因素：

## 知识库检索
- risk_type：
- query：
- 命中知识项：
- 相似度：
- 适用内容：

## 最小化验证方案
1. 基线：
2. 测试变量：
3. 无害 payload 类型：
4. 成功信号：
5. 负对照：
6. 副作用：
7. 停止条件：

## 结果判定
- 事实：
- 推断：
- 是否确认漏洞：
- 影响范围：
- 误报可能：

## 修复建议
- 根因修复：
- 临时缓解：
- 检测和监控：
- 回归测试：
```

------

## 9. 调用示例

### 示例 1：根据异常识别漏洞并检索

```text
现象：product 参数加入单引号后返回 500，加入两个单引号恢复正常。后端疑似 PostgreSQL。

检索步骤：
1. search_knowledge_base(query="SQL Injection PostgreSQL error boolean blind", risk_type="SQL Injection")
2. 从返回的知识项片段中提取 PostgreSQL 相关的注入验证方法
3. 选择布尔差分和低影响时间差分验证方案
```

### 示例 2：检索 XSS WAF 绕过

```text
目标输入位于双引号 HTML 属性中，经过一次 HTML entity 编码。

检索步骤：
1. search_knowledge_base(query="XSS double quote HTML attribute entity encoding bypass WAF", risk_type="PayloadsAllTheThings")
2. 从返回的 XSS Injection 知识项中筛选与该上下文相关的 payload
3. 筛选 10 个以内候选项，说明适用原因和风险级别
```

### 示例 3：检索 CVE 漏洞情报

```text
目标环境发现 Gogs 0.14.2，需验证是否存在已知漏洞。

检索步骤：
1. search_knowledge_base(query="Gogs 远程代码执行 CVE-2026-52806", risk_type="可利用漏洞库")
2. 从返回的知识项中提取影响版本、攻击向量、PoC 信息
3. 对比目标版本，给出验证建议
```

### 示例 4：检索 SSRF 绕过方法

```text
目标 URL 参数触发服务器请求，但 localhost 和内网 IP 被过滤。

检索步骤：
1. search_knowledge_base(query="SSRF bypass localhost filter DNS rebinding IPv6 encoding", risk_type="PayloadsAllTheThings")
2. 从 Server-Side Request Forgery 知识项中提取适用的绕过方法
3. 选择与目标 URL 解析器匹配的绕过技术
```

------

## 10. 错误模式与禁止行为

避免以下常见错误：

- 将知识项名称直接当成漏洞结论
- 未确认上下文就随机尝试 payload
- 用响应 500 直接认定注入成立
- 只测试成功样例，不设置负对照
- 忽略代理、缓存、WAF 和负载均衡影响
- 尝试通过文件路径访问知识库内容（知识库是语义检索，不是文件系统）
- 检索时未指定 risk_type，导致返回不相关类型的内容
- SQL 注入相关查询未使用 `risk_type="SQL Injection"` 导致精度下降

禁止自动生成：

- 未授权目标的批量攻击脚本
- 自动化凭据窃取、会话劫持或账户接管链
- 反弹连接、持久化和规避检测流程
- 大规模数据导出或破坏性利用

------

## 11. 修复优先级原则

1. 消除根因：参数化、严格解析、安全 API、权限隔离
2. 收紧信任边界：allowlist、schema、类型和长度限制
3. 降低影响：最小权限、网络隔离、沙箱、只读和事务约束
4. 增加检测：结构化日志、异常告警、速率限制、WAF 规则
5. 增加回归：单元测试、集成测试、负向安全测试

------

## 12. 技能完成标准

一次合格调用应满足：

- 已确认或明确标注授权状态
- 已识别输入上下文和潜在解析层
- 已使用 `search_knowledge_base` 定向检索，而非泛化罗列 payload
- 已指定正确的 `risk_type` 参数
- 已选择最低必要风险等级的测试方法
- 已给出成功、失败和负对照判据
- 已考虑误报、缓存、WAF 和环境噪声
- 已记录 payload 来源知识项名称
- 已区分事实、推断、假设和结论
- 已提供根因修复和回归测试建议
- 未输出超出授权范围的利用链

------

## 13. 维护

建议定期同步知识库，并在索引后执行以下检查：

1. 使用 `list_knowledge_risk_types` 确认所有风险类型已正确注册
2. 使用 `search_knowledge_base` 测试各漏洞主题是否可被检索到
3. 检查新导入的知识项是否出现在检索结果中
4. 确认 SQL Injection 和 XS-Leak 独立风险类型仍然有效
5. 仓库版本、同步日期和 commit 是否记录
