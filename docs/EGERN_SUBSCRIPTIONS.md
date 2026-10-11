# Egern 自用分流订阅

`egern/modules/custom-routing.yaml` 收录手动补充的分流规则。使用 Egern 的远程模块功能订阅，一条 URL 即可更新全部规则，同时保留不同规则的策略和顺序。

## 订阅地址

PR 合并后的长期地址：

```text
https://raw.githubusercontent.com/BitJarvis/firewall/main/egern/modules/custom-routing.yaml
```

PR 尚未合并时可使用当前分支地址测试：

```text
https://raw.githubusercontent.com/BitJarvis/firewall/chore/organize-configs/egern/modules/custom-routing.yaml
```

合并后将测试地址改为 `main` 地址，避免整理分支删除后无法更新。

## 在 Egern 中添加

1. 先备份当前配置。
2. 打开「工具 → 模块」，点击 `+`，添加上述远程模块 URL，并启用模块。
3. 更新间隔设为 `172800` 秒（两天）。修改仓库后如需立即生效，在客户端手动更新模块；Raw 缓存可能导致短暂延迟。
4. 确认主配置中已有下面六个策略组，名称必须完全相同：`台湾节点`、`狮城节点`、`自动选择`、`美国节点`、`数字货币`、`Apple AI`。`DIRECT`、`REJECT` 使用客户端内置动作。模块不提供代理节点或创建策略组。
5. 检查客户端实际合并后的规则顺序及命中日志。模块内部保留粘贴域名规则的顺序，但它与其他模块、主配置规则的优先关系仍需在客户端确认；特别检查 Bybit/MEXC、`cn-ssl`、`api-siri-prod.apple.com` 和数字货币规则。
6. 确认模块成功加载后，移除或禁用本地对应的旧域名规则；保留主配置中其他规则和最终兜底规则。抖音、AppleAI、Twitter 远程规则集已从本模块移除，如仍需使用，应在客户端单独管理。本模块中的 Apple AI 策略只应用于直接列出的 `api-siri-prod.apple.com` 后缀规则。

如直接编辑主配置，将下面条目追加到已有 `modules` 列表；不要创建重复的顶层 `modules` 键，也不要用它替换完整配置：

```yaml
modules:
  - name: BitJarvis 自用分流补充
    url: https://raw.githubusercontent.com/BitJarvis/firewall/main/egern/modules/custom-routing.yaml
    update_interval: 172800
    enabled: true
```

## 为什么用模块订阅

普通 `rule_set` 将集合中的匹配条件统一路由到引用处指定的 `policy`。本批规则需要多个策略，因此使用包含 `rules` 列表的原生 YAML 模块，逐条保存策略。

此 URL 应添加为「模块」，不能作为节点订阅、完整配置或普通单策略规则集导入。

## 整合范围

- 原始 43 条规则移除两个完全重复项及三个远程规则集引用后保留 38 条域名规则；`hotbookzone` 和 `flytogarden` 各保留第一次出现的规则。
- 保留所有规则类型、匹配值、策略和剩余条目的顺序，包括 `cursor.com` 的精确域名匹配。
- 保留 `supabase.co` 及其单独子域名规则，以及 `workers.dev` 和单独的 `wloc-spoofer.wloc.workers.dev`，本轮不做覆盖关系精简。
- 移除抖音、AppleAI、Twitter 三个 `rule_set` 条目，不展开或复制其上游规则；模块内部不再引用任何订阅链接。
- 粘贴开头孤立的 `- DIRECT` 属于前一段配置的残留，不作为规则导入；不添加默认规则。
- 根目录五个既有 `.conf` 文件的路径与内容不变。

## 后续维护

新增或修改个人补充域名时，只编辑 `egern/modules/custom-routing.yaml` 中的 `rules`。在 GitHub 上通过分支和 PR 修改，合并到 main 后客户端更新模块即可获取。

新规则应记录匹配类型、域名、策略与用途，并插入预期的优先位置；避免按策略重新排序后改变首次命中的行为。更换策略组名称时同步修改模块中的 `policy`。不要把代理节点、订阅密钥或证书私钥提交到仓库。

迁移失败时禁用新模块并恢复备份或旧规则。

依据 Egern 官方文档：[规则与规则集](https://egernapp.com/docs/configuration/rules/)、[远程模块格式与更新间隔](https://egernapp.com/docs/configuration/modules/)、[模块入口](https://egernapp.com/docs/faq/)。仓库中的静态校验不代替 iOS 客户端的导入及命中验证。
