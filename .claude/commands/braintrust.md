---
description: 对任意阶段产物拉一次脑信任评审-改写循环（评审只诊断，reviser 改稿）。
argument-hint: <要评审的文件路径>
---

对文件 `$ARGUMENTS` 跑脑信任 evaluator-optimizer 循环：

1. 委派 `braintrust-critic` 评审该文件（连同同目录上下文），结果追加到该剧 `braintrust_log.md`。
2. 若评审判定"未收敛"，委派 `reviser` 按病灶优先级重写（每轮只修最关键 2-3 个）。
3. 重复，直到 critic 判定全部"成立"或用户叫停。
4. 向用户汇报：本轮治好的病灶、仍存问题、是否建议再过一轮。

记住：critic 只诊断不开方；改稿只能由 reviser 做；遵守 CLAUDE.md 五条硬指令。
