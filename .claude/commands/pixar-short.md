---
description: 皮克斯式短剧全流程编排器。从一句话创意跑到生产规格，逐关守红线、关键阶段跑脑信任循环、每关停下等导演确认。
argument-hint: "<一句话创意>"
---

你现在是**编排器（lead agent）**。用户的一句话创意：$ARGUMENTS

严格遵守 `CLAUDE.md` 五条硬指令。采用 orchestrator-workers 模式：你不亲自写每阶段内容，而是按阶段委派子代理，并守关卡。

## 启动
1. 先和用户敲定**剧名**（用于建目录）。复制 `productions/_template/` 为 `productions/<剧名>/`。
2. 按下表逐阶段推进。**每完成一阶段，自检通过红线，向用户汇报产物摘要 + 红线结果，等用户确认再进下一阶段。**

## 阶段流水线
| 阶段 | 委派 | 产物 | 红线（不过不进） |
|---|---|---|---|
| 1 选题 | what-if-generator | premise.md | 可视化+会做错事的主角+天然困境 |
| 2 角色 | character-architect | characters.md | Want 可拍/Need 可感/对抗+有创伤 |
| 3 世界观 | world-builder | world.md | 1条核心假设+规则推动剧情+代价红线 |
| 4 结构 | story-spine | beats.md | 因果链/无巧合解围/非被动主角 |
| 5 情绪 | emotion-engineer | emotion.md | 泪点回收锚点+双层编码 |
| 6 评审 | braintrust-critic ↔ reviser | braintrust_log.md | 4镜头全部成立 |
| 7 分镜 | storyboard-director | storyboard.md | 静音可懂+一镜一任务 |
| 8 生产 | production-spec | production/（含 checklist.md） | 锁参考图后逐镜生成 + 出生产执行清单 |
| 9 质检 | continuity-checker | review.md | 一致性OK+结尾回收主题 |

## 脑信任循环（阶段 6，必跑；其他阶段用户要求时也可跑）
```
repeat:
    委派 braintrust-critic 评审 beats.md + emotion.md → 写 braintrust_log.md
    若全部"成立" → break
    委派 reviser 按病灶清单优先级重写
    （每轮只验证最关键 2-3 个病灶）
until 收敛 或 用户叫停
```
进入阶段 7 之前必须过一次完整脑信任。

## 效率调度协议（省 token / 防超时，见 CLAUDE.md「效率与 token 纪律」）
- **大体量阶段按单元委派**：分镜（阶段7）按单集委派——给 storyboard-director 一张"浓缩上下文卡 + 承接上一集的物件状态基准"，让它**只回单集表格块**，你用 Edit 拼进 storyboard.md。**不要让子代理反复读写正在增长的 storyboard.md**，也不要一次让它产 4 集（会超时）。
- **生产阶段（8）体量大时同样给浓缩卡**，不必让 production-spec 硬读 storyboard 全文。
- **只读评审代理（critic/checker）只回报，你来落盘**：它们没有写权，把诊断写进 `braintrust_log.md` / `review.md` 是你的活。
- **跨文件命名统一**：同物件/角色全程同一英文名；质检阶段扫一遍命名漂移。
- **风险处置写成工具指令**：质检发现的执行层风险，修订到 storyboard/edit 时用 `[GENERATE: …]` / `[POST-EDIT REDLINE: …]` 这类明确指令，不动剧情。

## 规则
- 子代理上下文隔离：靠 `productions/<剧名>/` 的文件传递上下文，不把长稿堆进对话。
- 你只汇报摘要与红线结论，把细节留在文件里。
- 导演（用户）保留每关的最终决定权。
