# frog-skills

Frog CLI 远程 SOP 仓：决策模型分类（SystemOne）→ Worker agent 任务分派时动态注入的 SOP 集。

## 定位（与 Frog 内置面分工）

| 面 | 来源 | 注入通道 |
|---|---|---|
| 本仓 SOP | GitHub 远程（本仓） | `frog` 拉取缓存 → 决策模型分类 → Worker 分派注入（动态提示词） |
| 本地挖掘 SOP | 会话挖掘（REQ-AUTORECOVERY-2） | session-start / dispatch（同 injection-log 审计与上限机制） |

两类 SOP 共用既有注入底座（provenance 尾注 / 8 条上限 / injection-log），只是数据源不同——远程 SOP 带 `source=remote` provenance。

## SOP 目录契约

每个 SOP = 一个目录：`sops/<domain>/<sop-slug>/sop.json`

```json
{
  "id": "debug-re/firmware-dump-workflow",
  "version": 1,
  "domains": ["debug-re"],
  "title": "Firmware dump acquisition workflow",
  "applicability": {
    "target_kinds": ["firmware"],
    "phases": ["triage"]
  },
  "steps": [
    { "n": 1, "action": "...", "tool_binding": "find_writers" }
  ],
  "decision_hooks": [
    {
      "key": "sop_applicability",
      "type": "choice",
      "criteria": "Which SOP fits the dispatched task",
      "options": { "apply": "task matches this SOP's applicability", "skip": "task does not match" }
    }
  ],
  "provenance": { "source": "remote", "repo": "gsc-org/frog-skills", "path": "sops/debug-re/firmware-dump-workflow" }
}
```

- `domains`：与 Frog experience-domain wire 对齐（dispatch 过滤键）
- `decision_hooks`：SystemOne typed question（noul/choice/score）——Frog 侧分类路由用（choice 选 SOP 域、noul 门适用性、score 分级注入预算），控制流留代码
- `steps[].tool_binding`：可选 Frog 工具 id（debug 语义工具族）
- 结构即契约：新增域=新增目录；`version` 单调递增；破坏性变更需新 id

## 同步

`frog` 侧按本仓 raw 内容地址拉取（`https://raw.githubusercontent.com/gsc-org/frog-skills/main/sops/...`），缓存于 `~/.frog/remote-sops/`；详细行为以 Frog 客户端实现为准。
