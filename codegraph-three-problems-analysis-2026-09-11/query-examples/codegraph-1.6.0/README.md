# Django 查询示例与完整结果

本目录由 `cross-agent-eval.mjs export` 从 `.agent-eval/verification-v2/results.json` 导出。

- CodeGraph：1.6.0
- 记录时间：2026-09-10T12:25:23.857Z
- Django 项目：`D:\工作空间\harbor\exp\django`

| 序号 | 问题 | 查询示例 | 字符数 | 事实覆盖率 | 重复比例 | Gate |
|---:|---:|---|---:|---:|---:|---|
| 1 | P1 | [请求分发链：关键证据与输出预算](./01-p1-request-dispatch-budget.md) | 24944 | 1 | - | PASS |
| 2 | P1 | [Model.save：大型核心文件中的内容分配](./02-p1-model-save-allocation.md) | 24862 | 1 | - | PASS |
| 3 | P2 | [自然语言查询：从请求到 view](./03-p2-natural-request-language.md) | 24996 | 0.429 | - | FAIL |
| 4 | P2 | [噪声与拼写错误：get_respnse 查询](./04-p2-noisy-typo-query.md) | 24090 | 0.2 | - | FAIL |
| 5 | P3 | [一次查询充分后是否停止](./05-p3-one-call-sufficiency.md) | 20078 | 1 | - | PASS |
| 6 | P3 | [完全重复查询是否缩减输出](./06-p3-repeat-dedup-stop.md) | 49724 | 1 | 1 | FAIL |

## 重新导出

```powershell
node scripts/agent-eval/cross-agent-eval.mjs export `
  --from .agent-eval/<run>/results.json `
  --out docs/benchmarks/query-examples-<version>
```

建议每个 CodeGraph 版本使用独立输出目录，不要覆盖旧结果。
