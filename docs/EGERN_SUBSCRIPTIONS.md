# Egern 自用规则集订阅

个人补充域名分成两个普通 Egern 规则集：`cn-ssl` 走 DIRECT，其余 37 条域名共用一个由客户端选择的代理策略。规则集只保存匹配条件，不绑定策略、不引用其他订阅。

## 订阅地址

PR 合并前使用分支地址：

| 文件 | 策略 | 测试地址 |
| --- | --- | --- |
| custom-direct.yaml | DIRECT | https://raw.githubusercontent.com/BitJarvis/firewall/chore/organize-configs/egern/rules/custom-direct.yaml |
| custom-proxy.yaml | 自己选择的一个代理策略组 | https://raw.githubusercontent.com/BitJarvis/firewall/chore/organize-configs/egern/rules/custom-proxy.yaml |

PR 合并后使用长期地址：

```text
https://raw.githubusercontent.com/BitJarvis/firewall/main/egern/rules/custom-direct.yaml
https://raw.githubusercontent.com/BitJarvis/firewall/main/egern/rules/custom-proxy.yaml
```

合并后将测试地址切换到 main，避免分支删除后更新失败。

## 添加与迁移

1. 备份当前 Egern 配置。
2. 打开「工具 → 规则」，添加两条规则集，分别填写对应 URL；直连集绑定 `DIRECT`，代理集绑定你希望统一使用的策略组。
3. 将直连集放在代理集前面，两者放在通用规则及默认兜底之前。这样包含 `cn-ssl` 的域名优先直连；其他匹配域名统一使用代理策略。关键词规则可能与后缀规则交叉匹配，检查命中日志确认符合预期。
4. 更新间隔设为 `172800` 秒（两天）。仓库修改后可以在客户端手动更新；Raw 缓存可能导致短暂延迟。
5. 确认两个规则集加载并命中正常，再移除对应旧域名规则。之前添加过 `custom-routing.yaml` 模块的，应禁用或删除该模块，避免旧的分地区策略继续生效；该模块文件已被两份规则集替代。
6. 本规则集不包含抖音、AppleAI、Twitter 远程规则引用；若仍需要这些功能，在客户端单独管理。

如直接编辑主配置，将以下两条追加到已有 `rules` 列表的合适位置；不要创建重复顶层键或覆盖整份配置。`自动选择` 仅为示例，可换成现有的任何目标代理策略组。PR 合并前将 URL 中的 main 替换为 chore/organize-configs。

```yaml
rules:
  - rule_set:
      name: 自用直连
      match: https://raw.githubusercontent.com/BitJarvis/firewall/main/egern/rules/custom-direct.yaml
      policy: DIRECT
      update_interval: 172800
      disabled: false
  - rule_set:
      name: 自用代理
      match: https://raw.githubusercontent.com/BitJarvis/firewall/main/egern/rules/custom-proxy.yaml
      policy: 自动选择
      update_interval: 172800
      disabled: false
```

## 规则与维护约定

- 原始 43 条规则移除两个完全重复项和三个远程规则集引用后，保留 38 个匹配条件：直连 1 条、代理 37 条。
- 保留匹配类型和匹配值：`domain` 对应 `domain_set`，`domain_suffix` 对应 `domain_suffix_set`，`domain_keyword` 对应 `domain_keyword_set`。集合内为逻辑或关系，不再按域名选择不同地区或 Apple AI/数字货币策略。
- 保留 `cursor.com` 的精确匹配，以及 Supabase、Workers 的原有父域名与单独子域名条目。
- 粘贴开头孤立的 `- DIRECT` 不属于规则，不导入；不新增默认规则。
- 根目录五个既有 `.conf` 文件的路径与内容不变。

以后只编辑 `egern/rules/custom-proxy.yaml` 和 `egern/rules/custom-direct.yaml` 中的匹配列表，通过分支和 PR 合并到 main 后更新客户端规则集即可。代理出口由客户端 `policy` 控制，更换节点或策略组无需改仓库文件。

不要将这些 URL 添加为模块、节点订阅或完整配置。加载失败时禁用新规则集并恢复备份。

格式依据：[Egern 官方规则集文档](https://egernapp.com/docs/configuration/rules/)。静态校验不代替 iOS 客户端导入与实际命中验证。
