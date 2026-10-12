# Firewall · 代理配置与分流规则

个人维护的 **Quantumult X / Egern / Clash** 代理配置与规则片段，包含 Apple AI 分流、AI 服务、媒体服务及广告过滤相关配置。

> [!IMPORTANT]
> `quantumultx.conf` 和 `qxv2.conf` 是两套**独立的 Quantumult X 完整配置**，不要把它们当作普通规则列表导入，也不要直接拼接两份配置。
>
> 第一轮仓库整理只补充文档与维护约定，**没有修改任何现有分流规则或策略行为**。

## 配置文件一览

| 文件 | 用途 | 类型 / 注意事项 |
| --- | --- | --- |
| [`quantumultx.conf`](./quantumultx.conf) | Quantumult X 完整配置：策略组、DNS、Blackmatrix7 远程规则与复写 | 完整配置；需自行配置代理节点/订阅 |
| [`qxv2.conf`](./qxv2.conf) | 另一套 Quantumult X 配置：区域节点策略、ConnersHua 与 Blackmatrix7 规则 | 完整配置；与上一份二选一使用 |
| [`apple-ai-egern.conf`](./apple-ai-egern.conf) | Apple AI / Siri 相关域名分流 | Egern 风格的 `DOMAIN` / `DOMAIN-SUFFIX` 规则片段；检查客户端策略名 `proxy` 是否适配 |
| [`apple-ai.conf`](./apple-ai.conf) | Apple AI 域名规则草稿 | 混用了 `HOST` 与 `DOMAIN` 风格，部分条目没有策略；**需要适配后再使用** |
| [`bitjarvis.conf`](./bitjarvis.conf) | 自定义 `tv.okxbot.com` 直连规则 | 单条 `HOST` 规则片段，不是完整配置 |

## 快速使用

### Egern 自用分流补充订阅

新增两份普通 Egern 规则集：[`custom-direct.yaml`](./egern/rules/custom-direct.yaml)（`cn-ssl` 直连）和 [`custom-proxy.yaml`](./egern/rules/custom-proxy.yaml)（其余 37 条域名共用一个代理策略）。规则集只保存匹配条件，不引用其他订阅。

在 Egern「工具 → 规则」添加规则集地址，直连集绑定 `DIRECT`，代理集绑定你选择的同一个策略组。分支测试地址、main 长期地址、规则顺序和迁移步骤见 [`docs/EGERN_SUBSCRIPTIONS.md`](./docs/EGERN_SUBSCRIPTIONS.md)。

### Clash 自用分流补充订阅

同一批匹配条件也提供 Clash `classical` 规则集：[`custom-direct.yaml`](./clash/rules/custom-direct.yaml)（1 条直连）和 [`custom-proxy.yaml`](./clash/rules/custom-proxy.yaml)（37 条代理）。直连绑定 `DIRECT`，代理绑定同一个自选策略组。订阅地址及配置示例见 [`docs/CLASH_SUBSCRIPTIONS.md`](./docs/CLASH_SUBSCRIPTIONS.md)。

维护时修改 Egern 源文件，再运行 `python3 scripts/sync_clash_rules.py` 同步生成 Clash 文件；提交前运行 `python3 scripts/sync_clash_rules.py --check` 检查一致性。

### 原有配置与规则片段

1. **完整配置**：在 Quantumult X 中导入 [`quantumultx.conf`](./quantumultx.conf) **或** [`qxv2.conf`](./qxv2.conf)，随后按自己的节点/订阅、DNS 和策略需求进行设置。导入前建议备份现有配置。
2. **独立规则片段**：根据实际客户端的规则语法，将 Apple AI 或自定义规则添加到合适的规则区域；不要把规则片段当作完整配置导入。
3. **策略检查**：检查 `proxy`、`direct`、`ProxyMain`、`美国节点` 等策略的引用和匹配逻辑是否符合自己的客户端与节点环境。
4. **实际验证**：在客户端检查配置解析、规则命中、AI 服务连接、国内直连，以及 HTTPS/MitM 设置。

### 原始文件订阅地址（Raw）

以下地址保留既有的仓库根目录路径，方便已有订阅继续使用：

- [Quantumult X 配置](https://raw.githubusercontent.com/BitJarvis/firewall/main/quantumultx.conf)
- [Quantumult X V2 配置](https://raw.githubusercontent.com/BitJarvis/firewall/main/qxv2.conf)
- [Apple AI · Egern 规则](https://raw.githubusercontent.com/BitJarvis/firewall/main/apple-ai-egern.conf)
- [Apple AI 规则草稿](https://raw.githubusercontent.com/BitJarvis/firewall/main/apple-ai.conf)
- [自定义规则](https://raw.githubusercontent.com/BitJarvis/firewall/main/bitjarvis.conf)

> Raw 链接仅代表源文件可订阅/获取，**不代表任意客户端都能直接解析**；应根据“类型 / 注意事项”选择用途。

## 仓库组织约定

```text
firewall/
├── README.md                 # 入口与使用说明
├── quantumultx.conf          # Quantumult X 主配置（保留原有路径）
├── qxv2.conf                 # Quantumult X 另一套配置（保留原有路径）
├── apple-ai-egern.conf       # Apple AI Egern 风格规则片段
├── apple-ai.conf             # Apple AI 待统一格式的草稿
├── bitjarvis.conf            # 自定义规则片段
├── docs/
│   └── MAINTENANCE.md        # 已知问题与后续整理建议
└── .gitignore                # 忽略系统/本地临时文件
```

**为什么暂时不移动 `.conf`？** 这些路径可能已经被客户端以 `raw.githubusercontent.com` 地址引用，移动会改变订阅 URL。后续如需分类到目录，应保留兼容入口并制定迁移方案。

## 维护与安全

- 现有规则引用第三方远程规则仓库与脚本；使用前确认来源与权限，并注意上游可能发生变化。
- 部分配置包含 `skip_validating_cert = true` 和 MitM/复写设置；这可能降低 TLS 证书验证保护，建议在了解风险后再使用。
- 导入时在本地填写自己的代理节点/订阅；**不要提交订阅密钥、密码、Token 或 MitM 私钥等个人凭据**。
- 新增域名规则时记录用途、目标客户端、策略与测试结果，避免不同客户端语法混用。

更多审查结果与后续修复清单：[`docs/MAINTENANCE.md`](./docs/MAINTENANCE.md)。

## 来源说明

现有完整配置引用了 [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)、[ConnersHua/RuleGo](https://github.com/ConnersHua/RuleGo) 等第三方资源。相关规则及脚本的维护、许可证和兼容性由各上游项目决定。
