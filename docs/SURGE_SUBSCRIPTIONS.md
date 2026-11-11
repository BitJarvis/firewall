# Surge 规则集订阅（AI 细分、综合与自用分流）

本仓库通过 [`scripts/sync_clash_rules.py`](../scripts/sync_clash_rules.py) 自动从 `egern/rules/*.yaml` 同步生成原生 **Surge `RULE-SET`** 规则集（`surge/rules/*.list`），并在 `gemini.list` 与 `ai.list` 中额外注入 Surge 专属的 Google 登录跳转 `URL-REGEX` 保护规则。

## Raw 订阅地址一览

| 规则集 | 推荐绑定策略 | Raw 订阅地址（`main` 分支） |
| --- | --- | --- |
| `gemini.list` | `💬 Gemini`（仅非香港节点，推荐美/新/德） | `https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/gemini.list` |
| `claude.list` | `💬 Claude`（仅非香港节点） | `https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/claude.list` |
| `openai.list` | `💬 OpenAi`（仅非香港节点） | `https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/openai.list` |
| `grok.list` | `💬 Grok`（优先美国节点） | `https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/grok.list` |
| `apple-ai.list` | `💬 Ai` 或 `🇺🇲 美国节点` | `https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/apple-ai.list` |
| `ai.list` | `💬 Ai`（兜底 Cursor/Windsurf/Perplexity/OpenRouter 等全部 AIGC） | `https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/ai.list` |
| `custom-direct.list` | `DIRECT` | `https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/custom-direct.list` |
| `custom-proxy.list` | `🚀 节点选择` / `♻️ 自动选择` | `https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/custom-proxy.list` |
| `ourbit.list` | 交易所/自选策略 | `https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/ourbit.list` |
| `meta.list` | 国外媒体 / Meta 策略 | `https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/meta.list` |

## 关键优先级原则（避坑必读）

1. **细分 AI 规则集必须在 `ai.list` 之前**：`ai.list` 是包含 Sukka AIGC + 主流 AI 平台的超集。若对 Gemini、Claude、OpenAI、Grok 单独设置了策略组，务必按 **细分规则在前、`ai.list` 在后** 的顺序排列，避免被超集提前截获。
2. **AI 策略组严禁混入香港（HK）节点**：若 AI 策略组默认指向 `🚀 节点选择` → `♻️ 自动选择`，低延迟测速极易选中香港节点，导致 Google Gemini / Claude / OpenAI 触发 `Unsupported country / region` 封锁或污染账号地区 Cookie。建议为 AI 策略组单独指定自建美/欧节点或美国/新加坡/日本节点。
3. **`apple-ai.list` 需置于通用 `Apple.list` 之前**：否则 `*.apple.com` 与 `*.icloud.com` 会被通用苹果直连规则提前放行。

## Surge `[Rule]` 配置片段示例

```ini
[Rule]
# 1. 自用直连优先
RULE-SET,https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/custom-direct.list,DIRECT,update-interval=86400

# 2. AI 细分规则（置于 Apple.list 与通用 AI.list 之前）
RULE-SET,https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/gemini.list,"💬 Gemini",update-interval=86400
RULE-SET,https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/claude.list,"💬 Claude",update-interval=86400
RULE-SET,https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/openai.list,"💬 OpenAi",update-interval=86400
RULE-SET,https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/grok.list,"💬 Grok",update-interval=86400
RULE-SET,https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/apple-ai.list,"💬 Ai",update-interval=86400
RULE-SET,https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/ai.list,"💬 Ai",update-interval=86400

# 3. 自用代理补充
RULE-SET,https://raw.githubusercontent.com/BitJarvis/firewall/main/surge/rules/custom-proxy.list,"🚀 节点选择",update-interval=86400
```
