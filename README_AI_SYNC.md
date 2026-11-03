# AI 规则原地址同步

Surge 继续使用原来的订阅地址，无需更改客户端配置：

```text
https://raw.githubusercontent.com/yinyues2/surge/main/AI.list
```

例如在 Surge 的 `[Rule]` 中保留：

```ini
RULE-SET,https://raw.githubusercontent.com/yinyues2/surge/main/AI.list,AI,no-resolve
```

`AI` 为自己的策略或策略组名称。文件采用 Surge 纯文本规则格式，每行一条规则，没有 YAML 外壳或策略目标；IP 类规则带 `no-resolve`。

## 每天的更新顺序

| 北京时间 | 工作 |
| --- | --- |
| 01:00 | [Clash 仓库工作流](https://github.com/yinyues2/Clash/actions)合并上游与手工规则，生成同一集合的 Clash YAML 和 Surge 文本；任务最长 15 分钟。 |
| 01:20 | 本仓库工作流校验两种输出一致后，将 Clash 的 `Surge/AI.list` 原样镜像到根目录 `AI.list`。 |
| 02:00 | 另外配置的 ChatGPT 计划任务核对两个仓库的运行和输出；该任务不在本仓库运行。 |

GitHub 定时工作流可能排队延迟，这个顺序不是跨仓库的即时触发保证。若 Clash 尚未完成，本仓库读取其最近一次已提交的有效输出；ChatGPT 后续检查两仓库状态。客户端仍按自己的规则集刷新周期拉取，也可以手动刷新。GitHub 的公共仓库定时工作流还可能因连续 60 天无活动而停用，详见 [GitHub schedule 说明](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule)。

可在本仓库 **Actions → Sync AI rules → Run workflow** 选择 `main` 手动同步。向 `main` 推送该工作流文件的改动也会触发运行；规则输出自身的提交不会再次触发。

## 输入和手工规则统一在 Clash 仓库维护

- 上游清单：[rules/ai_sources.json](https://github.com/yinyues2/Clash/blob/main/rules/ai_sources.json)。
- 需要长期保留的手工规则：[rules/ai_custom.yaml](https://github.com/yinyues2/Clash/blob/main/rules/ai_custom.yaml)。
- 校验与生成逻辑：[scripts/sync_ai_rules.py](https://github.com/yinyues2/Clash/blob/main/scripts/sync_ai_rules.py)。
- 来源、许可与原文副本：[THIRD_PARTY_NOTICES.md](https://github.com/yinyues2/Clash/blob/main/THIRD_PARTY_NOTICES.md)。

`AI.list` 是生成结果，直接修改会在下次同步时被输入重新生成的内容覆盖。文件头中的手工规则、脚本和许可相对路径均指上述 Clash 仓库。其他五个既有 `.list` 文件不参与这个同步。

## 发布前的检查

工作流使用 Clash 当前的严格解析器，验证两种来源格式合法、非空且规范化规则集合完全相同，并验证本仓库旧 `AI.list`。总条数减少超过 20% 时停止，需核对后再处理；任何下载、解析或校验失败都不会提交更新。通过后原样复制 Surge 文本字节，只有 `AI.list` 内容改变才提交。

首次接入时，从 89 条更新到 233 条；原有 89 条规范化后全部保留。其中旧的单 IP 写法 `IP-CIDR,17.253.4.125,no-resolve` 规范为 `IP-CIDR,17.253.4.125/32,no-resolve`。

发布任务仅使用本仓库的 `GITHUB_TOKEN` 和 `contents: write` 权限，无需额外 PAT 或跨仓库写入凭据。工作流串行运行，提交前确认 `main` 未变化，再普通推送；遇到并发提交或分支保护会失败并保留已发布版本，不会强制推送。可在处理原因后重新运行。
