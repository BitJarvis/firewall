# Clash 自用规则订阅

与 Egern 自用规则保持一致：直连集包含 1 条 `cn-ssl` 关键词，代理集包含其余 37 条匹配条件。适用于支持 `rule-providers` 和 `classical` 的 Clash / Mihomo 内核。文件只保存匹配条件，策略在客户端配置中指定，不引用其他订阅。

## 当前分支测试地址

- 直连：https://raw.githubusercontent.com/BitJarvis/firewall/chore/organize-configs/clash/rules/custom-direct.yaml
- 代理：https://raw.githubusercontent.com/BitJarvis/firewall/chore/organize-configs/clash/rules/custom-proxy.yaml

PR 合并后将 URL 中的 `chore/organize-configs` 替换为 `main`，作为长期订阅地址；合并前 main 地址尚不可用。

## 添加到现有配置

将以下条目合入已有的 `rule-providers` 和 `rules`，不要覆盖现有配置，也不要重复创建同名顶层字段。这是配置片段，不是完整配置。

```yaml
rule-providers:
  custom-direct:
    type: http
    behavior: classical
    path: ./ruleset/custom-direct.yaml
    url: "https://raw.githubusercontent.com/BitJarvis/firewall/chore/organize-configs/clash/rules/custom-direct.yaml"
    interval: 172800
  custom-proxy:
    type: http
    behavior: classical
    path: ./ruleset/custom-proxy.yaml
    url: "https://raw.githubusercontent.com/BitJarvis/firewall/chore/organize-configs/clash/rules/custom-proxy.yaml"
    interval: 172800

rules:
  - RULE-SET,custom-direct,DIRECT
  - RULE-SET,custom-proxy,自动选择
  # 后面接现有规则，最终 MATCH 放在最后
```

将 `自动选择` 改为配置中实际存在的代理策略组名。两条自用 RULE-SET 应放在可能先匹配这些域名的其他规则和最终 MATCH 之前，直连集放在代理集前面。更新间隔为两天。

Clash 文件使用 `payload` 列表，匹配类型一一对应：

| Egern | Clash |
| --- | --- |
| domain_keyword_set | DOMAIN-KEYWORD |
| domain_suffix_set | DOMAIN-SUFFIX |
| domain_set | DOMAIN |

## 同步维护

统一修改 `egern/rules/custom-direct.yaml` 和 `egern/rules/custom-proxy.yaml`，随后在仓库根目录执行：

```bash
python3 scripts/sync_clash_rules.py
python3 scripts/sync_clash_rules.py --check
git diff --check
```

脚本只依赖 Python 3 标准库，支持本仓库当前的简单 YAML 格式（集合标题、未加引号的域名/关键词列表、空行、整行注释），不支持的语法会报错。自动生成的 `clash/rules/` 文件应和源文件一起提交，避免手动修改后被重新生成覆盖。`--check` 检测输出是否缺失或过期，异常时返回非零退出码。

客户端验收：更新两个规则集后检查加载成功、`cn-ssl` 命中 DIRECT，以及代理列表中的域名命中所选策略组。

格式依据：[Mihomo rule-providers](https://wiki.metacubex.one/en/config/rule-providers/)、[classical 文件内容](https://wiki.metacubex.one/en/config/rule-providers/content/)。
