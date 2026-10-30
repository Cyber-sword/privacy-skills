# 信号 → skill 路由表（质量加权版）

> **同步状态**：541 个技能 → 现存 278（全部可加载），归档 263。
> **每条标注格式**：`技能名（Q质量·V可见）`。全部技能均可加载，Q 只决定先后次序，不决定能否调用；
> 表里没列到的技能照样可以用，只要它在库中存在。
> **选型顺序**：同信号按 Q 降序优先；Q2 可作入口也可细读；同一时刻主用 1 个入口 + 2–3 个专项，需要更多随时补载，不设硬上限。

## A. 侦察 / 资产 / 情报

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| 开局测绘与资产归簇 | attack-surface-recon（Q3·V）、information-gathering（Q4·V）、recon-scope-triage（Q4·V） | recon-fingerprint（Q4·V）、techstack-identification（Q4·V） | 先出可达性台账，再进站 |
| 子域枚举 / 接管 | recon-subdomain（Q4·V）、hunt-subdomain（Q5·V） | — | SRC 只记接管指纹，不做占位验证 |
| 端口与服务 | recon-port-scan（Q4·V） | — | nmap 技能已归档，recon-port-scan 内含 nmap/masscan/rustscan |
| 目录与内容发现 | recon-dir-scan（Q4·V） | — | 叠加项目限速与预算 |
| 外部情报 / OSINT | offensive-osint（Q5·V） | identity-provider-recon（Q5·V）、osint-methodology（Q5·V）、org-attack-surface（Q4·V） | 均可直接加载 |
| 源码 / 泄露物定位 | hunt-source-leak（Q4·V）、source-code-hunting（Q3·V） | — | 命中开源同款 → 转 N 域白盒 |
| SPA / 前端 bundle 接口提取 | hunt-spa-api（Q4·V） | — | 懒加载 chunk 也要落盘 |

## B. API / 接口暴露面

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| API 面刚发现，定测试顺序 | api-sec（Q2·V） | — | 入口路由器，2KB，别当方法论 |
| 端点与文档发现 | api-recon-and-docs（Q3·V）、api-security-testing（Q4·V） | — | 文档变体清单 + 版本漂移 |
| 对象 id / 越权 | api-authorization-and-bola（Q3·V）、hunt-idor（Q4·V） | — | 无凭据先标「阻塞-无凭据」 |
| Token / JWT / API key | api-auth-and-jwt-abuse（Q3·V）、hunt-jwt-crypto（Q4·V） | jwt-oauth-token-attacks（Q4·V） | 签名伪造只认 hunt-jwt-crypto |
| 影子 / 未文档化 API | hunt-shadow-api（Q3·V） | — | 版本与网关归一化 |
| GraphQL | hunt-graphql（Q4·V） | hunt-fintech-graphql（Q4·V） | 金融业务用后者补 delta |
| gRPC 反射 / HTTP2 | hunt-grpc（Q4·V） | — | 需 grpcurl |
| WebSocket / socket.io | hunt-websocket（Q4·V） | — | CSWSH 要有双会话 |
| 批量赋值 / 方法篡改 | hunt-api-misconfig（Q4·V） | — | 额外 JSON 字段先小样本 |

## C. 认证 / 会话 / 身份

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| 认证面刚发现，定打法 | auth-sec（Q2·V） | — | 入口路由器 |
| 认证绕过 / 锁定策略 | hunt-auth-bypass（Q4·V） | — | ≤10 组小字典，不实爆 |
| 找回密码 / 账号恢复 | hunt-forgot-password（Q3·V） | — | token 回显 / Host 投毒 / 可预测 |
| MFA / OTP | hunt-mfa-bypass（Q3·V） | authenticated-session-acquisition（Q3·V） | SRC 禁全 key 空间爆破 |
| 验证码 | hunt-captcha-bypass（Q3·V） | — | 目读走人工通道 |
| 限速缺失 / 用户名枚举 | hunt-brute-force（Q4·V） | — | 只定性，不实爆 |
| 会话管理 | hunt-session（Q4·V） | 401-403-bypass-techniques（Q4·V） | SRC 禁调登出/吊销 |
| OAuth / OIDC | oauth-oidc-misconfiguration（Q2·V） | — | redirect_uri / state / PKCE |
| SAML / 企业 SSO | hunt-saml（Q3·V） | — | XSW / 签名剥离 |
| CSRF | hunt-csrf（Q4·V） | — | SameSite 绕过 / 链 ATO |
| 账号接管链 | hunt-ato（Q4·V） | — | 9 条路径，链式升危 |

## D. 注入与代码执行

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| 输入去向不明，先分流 | injection-checking（Q2·V） | — | 入口路由器 |
| SQLi | hunt-sqli（Q4·V） | sql-injection-testing（Q4·V） | sqlmap 纪律 |
| NoSQL / Redis | hunt-nosqli（Q3·V） | — | 操作符注入 |
| XSS | hunt-xss（Q5·V） | xss-testing（Q3·V） | 多上下文 / mXSS |
| SSRF | hunt-ssrf（Q5·V） | ssrf-testing（Q4·V） | 云 metadata 优先 |
| SSTI | hunt-ssti（Q3·V） | — | 引擎指纹后再升 RCE |
| 命令注入 | cmdi-command-injection（Q3·V） | — | 分内容类型，别一律表单编码 |
| RCE 面 | hunt-rce（Q4·V） | offensive-rce（Q4·V） |  |
| XXE | hunt-xxe（Q4·V） | xxe-injection-testing（Q3·V） | OOB 为先 |
| LDAP / XPath | hunt-ldap（Q4·V） | — | 过滤器语义 |
| 反序列化 | hunt-deserialization（Q3·V） | deserialization-testing（Q4·V）、jndi-injection（Q4·V） | Java 链版本矩阵 |
| 框架级 RCE | fastjson-exploitation（Q4·V）、log4shell-exploitation（Q5·V）、shiro-exploitation（Q4·V） | spring-exploitation（Q4·V） | spring-exploitation |
| Java char 窄化绕过 | ghost-bits-cast-attack（Q4·V） | — | WAF 已拦时再上 |
| WAF 拦截 | waf-bypass-techniques（Q4·V） | — | 换编码 / 换位置 |
| 原型链污染 | prototype-pollution（Q4·V） | — | → RCE 需 gadget |

## E. 文件 / 上传 / 下载

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| 文件面入口 | file-access-vuln（Q2·V） | — | 入口路由器 |
| 上传 | hunt-file-upload（Q3·V） | file-upload-testing（Q4·V） | SRC 无害探针即停 |
| LFI / 路径穿越 | hunt-lfi（Q4·V）、path-traversal-lfi（Q4·V） | — | php://filter 链 / 日志投毒 |
| 任意下载 | hunt-lfi（Q4·V）、hunt-source-leak（Q4·V） | — | exploit-file-download 已归档，用 hunt-lfi 替代 |

## F. 业务逻辑 / 资金 / 竞态

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| 逻辑面入口 | business-logic-vuln（Q2·V） | — | 入口路由器 |
| 流程 / 状态机 / 价格 | hunt-business-logic（Q4·V） | business-logic-testing（Q4·V） |  |
| 竞态 / 一次性操作 | race-condition（Q4·V）、hunt-race-condition（Q4·V） | — | SRC 禁单包高并发 |
| IDOR / 越权 | hunt-idor（Q4·V） | — | A/B 双账号互验优先 |
| 金融交易 / 账本 | hunt-fintech-graphql（Q4·V） | — | 精度 / 幂等 / 双花 |

## G. 前端 / DOM / 客户端

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| DOM clobbering / postMessage / SW | hunt-dom（Q4·V） | — | 需真实浏览器上下文 |
| HTML 注入但不执行 | hunt-html-injection（Q3·V） | — | dangling markup 亦可 |
| 点击劫持 | hunt-clickjacking（Q3·V） | — | 缺头不等于可利用 |
| Next.js / Node 栈 | hunt-nextjs（Q4·V）、hunt-nodejs（Q3·V） | — | Server Actions / 原型链 |
| sourcemap / 构建产物 | hunt-source-leak（Q4·V） | — | 还原 TS 源码 |

## H. 协议 / 缓存 / 反代

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| 缓存投毒 / 欺骗 | hunt-cache-poison（Q4·V） | — | 先确认缓存键 |
| Host 头注入 | hunt-host-header（Q4·V） | — | 重置链接投毒 → ATO |
| 请求走私 | hunt-http-smuggling（Q3·V） | — | 前后端分帧差异 |
| TLS / DNS / 邮件安全 | hunt-tls-network（Q4·V） | — | 多为 Info/Low，挑能付钱的 |
| 异常条件 / 栈回溯 | hunt-exceptional-conditions（Q3·V） | — | 畸形输入触发 fail-open |

## I. 组件 / 框架 / CVE

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| 指纹 → CVE 定性（必走） | component-vuln-intel（Q3·V）、cve-poc-generator（Q3·V） | cve-risk-score（Q2·V） | PoC 只做最小验证 |
| Spring Boot | hunt-springboot（Q4·V） | spring-exploitation（Q4·V） | actuator → heapdump |
| ASP.NET | hunt-aspnet（Q4·V） | — | ViewState / machineKey / trace.axd |
| Laravel | hunt-laravel（Q3·V） | — | Ignition / Telescope / .env |
| PHP 源码审计 | php-audit-skills（Q4·V） | — | 危险函数 + 正则扫描 |
| SharePoint / IIS | hunt-sharepoint（Q4·V） | — | ToolShell 前置链 |
| CI/CD 面 | hunt-cicd（Q4·V） | offensive-cicd-secrets（Q4·V）、offensive-cicd-pipeline（Q4·V） | 后两者含外发 PoC（已加护栏） |

## J. 云 / 容器 / K8s

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| K8s / 容器暴露 | hunt-k8s（Q4·V） | — | 6443/10250/2379 |
| 云配置 / 存储桶 / IMDS | hunt-cloud-misconfig（Q4·V） | — | S3 公开 + SSRF 打元数据 |
| 云攻击方法论 | cloud-security-audit（Q4·V） | cloud-attack-methods（Q3·V） | 均可直接加载 |

## K. 内网 / AD / 后渗透（红队场景）

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| AD 域 | active-directory-attack（Q2·V） | offensive-active-directory（Q4·V） | SRC 场景禁用横移 |
| NTLM 信息泄露 | hunt-ntlm-info（Q4·V） | — | 公网 IIS/Exchange 可触发 |
| 横向 / 凭据 | offensive-lateral-movement（Q4·V）、offensive-netexec（Q4·V） | — |  |
| 提权 | offensive-windows-privesc（Q4·V）、offensive-linux-privesc（Q4·V） | — |  |
| 内网总纲 | network-penetration-testing（Q4·V） | — |  |

## L. 移动 / APP / 小程序

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| Android 流水线 | apk-redteam-pipeline（Q4·V）、apktool（Q4·V）、jadx（Q4·V） | — | APK → smali → Frida |
| 移动逆向 | binary-mobile-reversing（Q4·V）、rev-frida（Q4·V） | rev-idapython（Q4·V）、rev-unicorn-debug（Q3·V）、reverse-engineering（Q3·V）、dwarf-expert（Q3·V） | Native / 反调试 |
| 移动端总纲 | mobile-app-security-testing（Q4·V） | — |  |
| 小程序 | hunt-spa-api（Q4·V）、hunt-source-leak（Q4·V） | binary-mobile-reversing（Q4·V） | wxmini-security-audit 已归档；小程序包按 JS bundle 审计（解包 wxapkg → 端点/密钥提取） |

## M. AI / LLM 应用

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| LLM 特征面 | hunt-llm-ai（Q4·V）、ai-threat-testing（Q4·V） | — | 注入 / 工具滥用 / ASCII 走私 |
| RAG / 向量库 | hunt-rag-vector（Q4·V） | — | 语料投毒 / 跨租户检索 |

## N. 白盒审计 / SAST

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| 审计总纲 | code-audit（Q4·V）、secure-code-review（Q4·V）、audit-skills（Q4·V） | — | 入口 + 语言专项 |
| 工具化 SAST | codeql（Q5·V）、semgrep（Q4·V） | — | Q5 工具链，先确认本机二进制存在；缺失则退回 audit-skills 手工审计 |
| 变体猎杀 | variant-analysis（Q4·V）、source-code-hunting（Q3·V） | — | 一个洞 → 全库同类 |
| 切片 / 调用图 | slicing-code-context（Q3·V） | — | 依赖 Trailmark，先探工具 |
| 0day 流程 | variant-analysis（Q4·V）、source-code-hunting（Q3·V） | — | zero-day-discovery 已归档；用变体猎杀 + 源码狩猎组合替代 |

## O. 密码学

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| 自定义加密 / 国密 / 签名 | cryptography（Q4·V） | — | padding oracle / 弱 RNG / 离线复原 |

## P. CTF

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| 按题型选 | solve-challenge（Q3·V）、ctf-solver（Q3·V） | — | ctf-solver 的 scripts/search.py 缺失，检索功能不可用；以 ctf-* 题型专项为主 |
| Web / Pwn / Reverse / Crypto | ctf-web（Q4·V）、ctf-pwn（Q5·V）、ctf-reverse（Q5·V）、ctf-crypto（Q4·V） | — | Q5 在 pwn/reverse |
| 取证 / 杂项 / OSINT / AI | ctf-forensics（Q4·V）、ctf-misc（Q4·V）、ctf-osint（Q4·V）、ctf-ai-ml（Q4·V） | — |  |
| 恶意样本 / 源码审计题 | ctf-malware（Q4·V）、ctf-source-audit（Q3·V） | — |  |

## Q. 取证 / 应急

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| 数字取证 | dfir（Q4·V） | ctf-forensics（Q4·V） | 事件日志 / PCAP / 时间线 |
| 应急响应 | incident-response（Q4·V） | — |  |

## R. 报告 / 证据 / 误报治理

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| 交付前验证门禁 | triage-validation（Q5·V） | fp-check（Q4·V） | 7 问门禁，不过就杀 |
| 证据与脱敏 | evidence-hygiene（Q4·V） | — | Cookie/PII/HAR 处理 |
| 报告成文 | pentest-report（Q3·V） | vulnerability-assessment（Q4·V） | 格式以项目模板为准 |
| 升链 | capability-primitive-search（Q4·V）、attack-path-stitcher（Q3·V） | — | 低危凑原语 |

## S. 元能力 / 载荷 / 字典

| 观察到的信号 | 首选（质量·可用性） | 补充 / 降级 | 备注 |
|---|---|---|---|
| 载荷与绕过表 | payloadsallthethings（Q2·V） | security-patterns（Q3·V） | 均可直接加载 |
| 行业案例对照 | secknowledge-skill（Q4·V） | — | 进站先查账号命名 |
| 项目黑板 / 状态机 | pentest-agent-os（Q3·V） | pentest-verification（Q2·V） | CyberStrikeAI 原语 |
| 心态纠偏 | redteam-mindset（Q4·V） | — | 挖不动时 |
| 代理 / 工具自举 | proxy-tool-bootstrap（Q2·V） | — | 被封 403/429 时 |

## 路由失败处理（必须留痕）

| 情况 | 动作 |
|---|---|
| 目标技能已归档 / 不存在 | 记 `ROUTE-MISS: archived`，用同域 V 项替代；无替代则在队列里标「能力缺口」 |
| 技能能加载但依赖的工具/脚本缺失 | 记 `ROUTE-MISS: tool`，只取方法论，不执行其脚本；转自建等价实现 |
| 技能默认流程与项目红线冲突 | 裁剪冲突步骤，记「skill+条款+取舍理由」到 `检查记录.md` |
