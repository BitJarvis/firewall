# 维护说明与待处理事项

## 第一轮整理原则

后续新增的 Egern 自用规则集订阅说明见 [`EGERN_SUBSCRIPTIONS.md`](./EGERN_SUBSCRIPTIONS.md)。以下“本轮”及四文件差异校验指最初的仓库整理提交；规则集订阅提交另外新增 YAML 和迁移文档，仍不修改原有根目录 `.conf`。

本轮仅整理仓库说明、风险清单及忽略规则；**不修改现有 `.conf` 内容，也不改变策略顺序或远程规则源**。现有根目录文件的 URL 可视为外部订阅接口，在确认订阅方已完成迁移前不要重命名/移动。

## 当前文件与兼容性

- `quantumultx.conf`：完整 Quantumult X 配置，使用 Blackmatrix7 远程规则，具有 `ProxyMain`、`Final`、AI/流媒体等策略组。
- `qxv2.conf`：另一套完整 Quantumult X 配置，使用 ConnersHua 与 Blackmatrix7 等规则源，包含按地区选择的节点策略。
- `apple-ai-egern.conf`：使用 `DOMAIN` / `DOMAIN-SUFFIX` 风格，并写有 `proxy` 动作的 Egern 规则片段。
- `apple-ai.conf`：同时包含 `HOST` / `HOST-SUFFIX` 和 `DOMAIN` / `DOMAIN-SUFFIX`，且后半段缺少动作字段；不能假定可直接导入某一种客户端。
- `bitjarvis.conf`：只有 `HOST,tv.okxbot.com,DIRECT` 一条规则，适合作为定制覆盖规则。

## 待处理问题（尚未修改）

| 优先级 | 位置 | 观察结果 | 建议处理方式 |
| --- | --- | --- | --- |
| 高 | `apple-ai.conf` | Apple AI 列表的规则类型不统一，部分 `DOMAIN` / `DOMAIN-SUFFIX` 行没有策略字段 | 确认目标客户端后，拆分或统一语法，再逐条测试 |
| 高 | 两套 Quantumult X 完整配置 | 均设置了 `skip_validating_cert = true` | 核实是否必要；在客户端验证后再考虑恢复证书验证 |
| 中 | `quantumultx.conf` | 远程 `Global` 列表使用 `policy=Proxy`，而自定义主要代理组命名为 `ProxyMain` | 核对客户端内置 `proxy` 行为及实际命中策略；不要只凭命名推断错误 |
| 中 | `qxv2.conf` | ConnersHua 的 AI 列表及 Blackmatrix7 的 OpenAI/Claude/Gemini/Copilot 列表共存 | 核对规则重叠、优先级、命中和预期出口地区 |
| 中 | 两套完整配置 | 依赖第三方规则、脚本、复写及解析器 | 建立定期更新、可用性及供应链检查流程 |
| 低 | `quantumultx.conf` | 注释序号 `12` 与 `8` 分别重复 | 后续仅调整注释，不改变规则顺序 |

## 本轮已完成

- 补充 README 的配置索引、使用说明与原始文件订阅地址。
- 新增 `.gitignore`，忽略 macOS 元数据、编辑器状态、日志及本地环境变量文件。
- 从 Git 索引移除 `.DS_Store`；本地 Finder 文件可继续保留。
- 保留全部 `.conf` 文件的路径与内容，现有订阅地址不变。

## 提交前校验

本轮只允许 README、忽略规则、维护文档及 `.DS_Store` 删除进入差异。在整理分支中执行：

```bash
git diff --cached --check
git diff --cached --name-status
git diff --exit-code main -- '*.conf'
git ls-files -- '*.DS_Store'
git check-ignore .DS_Store docs/.DS_Store .env .idea/workspace.xml
```

配置差异与已追踪的 `.DS_Store` 列表应为空；忽略检查应列出对应文件。提交后还应检查 `git diff --check main...HEAD`。这些检查验证整理范围和路径兼容性；客户端规则解析与第三方资源可用性仍需按下方清单单独验收。

## 之后如需重构，按这个顺序进行

1. **确定主配置**：确认 `quantumultx.conf` 与 `qxv2.conf` 哪一份是当前实际在用的版本；另一份可标为备用/历史版本，但先别移动。
2. **统一 Apple AI 规则**：明确 Quantumult X 与 Egern 各自的文件，补全动作并减少重复。
3. **统一策略命名**：梳理主配置内的代理组、最终规则、地区组与 AI 分流出口，在客户端验证后逐步改动。
4. **审查远程资源**：逐项检查远程规则 URL 是否有效、`opt-parser` 是否必要、脚本权限是否合理。
5. **增加验证**：为文本格式、策略引用、订阅地址建立自动化静态检查；由于不同客户端方言不同，不能仅靠通用 linter 判定正确性。

## 每次修改配置的人工验收清单

- [ ] 导入/更新成功，客户端没有语法错误
- [ ] 代理节点与策略组正常显示，默认策略符合预期
- [ ] 国内常用应用、局域网、DNS 工作正常
- [ ] OpenAI、Claude、Gemini、Apple AI 等所需服务分流按预期生效
- [ ] 第三方远程规则、重写脚本没有异常日志
- [ ] 若开启 MitM，已评估证书验证设置和解密范围
- [ ] 对比 Raw 链接，确保已有订阅 URL 未被意外改变

## 再次出现已被 Git 追踪的 `.DS_Store` 时

本轮已经移除根目录 `.DS_Store`。若以后有文件被强制加入，可在本地仓库执行（将路径替换为实际被追踪的文件，只解除 Git 跟踪，不要求删除本机 Finder 文件）：

```bash
git rm --cached .DS_Store
git commit -m "chore: stop tracking macOS metadata"
```

之后 `.gitignore` 会阻止其再次被普通的 `git add .` 加入。若本机文件本来不存在，`git rm --cached` 仍可以移除索引记录。

## Egern 与 Clash 自用规则同步

自用规则以 `egern/rules/custom-{direct,proxy}.yaml` 为维护源，Clash 版本由 `scripts/sync_clash_rules.py` 生成。修改后执行生成命令，并将两种格式一起提交。提交前运行 `python3 scripts/sync_clash_rules.py --check`，检查直连/代理两份输出与源文件一致。Clash 导入示例见 [`CLASH_SUBSCRIPTIONS.md`](./CLASH_SUBSCRIPTIONS.md)。
