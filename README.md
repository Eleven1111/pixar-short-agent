# 皮克斯短剧生产线 · Claude Code Agent

把皮克斯公开方法论复刻成一条"从一句话创意到成片蓝图"的短剧流水线。
架构 = 编排器（orchestrator-workers）+ 脑信任评审（evaluator-optimizer）+ 文件即记忆。

> 本仓库只产出"可直接喂给视频/语音/音效模型的提示词包 + 镜头清单 + 一致性约束"，**不在 agent 内调用任何付费生成 API**——零密钥依赖、可移植。

## 功能概览

- **9 阶段串行流水线**，每关有"通过红线"，不过不进下一关，每关停下等导演拍板。
- **脑信任评审循环**：`braintrust-critic`（只读，机制上无法改稿）↔ `reviser`（改写），反复迭代至四视角全部"成立"。
- **文件即记忆**：子代理上下文隔离，靠 `productions/<剧名>/` 落盘文件传递上下文，省 token 不丢上下文。
- **10 个专职子代理 + 4 个 slash 命令 + 1 个方法论 skill**，全部可按预算/风格调。

## 安装与使用

```bash
git clone <repo-url> pixar-agent
cd pixar-agent          # 在此目录启动 Claude Code，.claude/ 会被自动识别

# 全流程（编排器逐阶段产文件、自检红线、关键阶段跑脑信任循环，每关停下等你拍板）
/pixar-short "如果每台冰箱里都住着一个保鲜精灵……"

# 随时单独拉评审 / 痛苦测试 / 结构诊断
/braintrust productions/<剧名>/beats.md
/pain-test  productions/<剧名>/characters.md
/diagnose   productions/<剧名>/beats.md
```

新建一部剧：复制 `productions/_template/` 为 `productions/<剧名>/`，再跑 `/pixar-short`。

## 9 个阶段与产物

| 阶段 | 子代理 | 产物 | 通过红线 |
|---|---|---|---|
| 1 选题 | what-if-generator | premise.md | 可视化 + 会做错事的主角 + 天然困境 |
| 2 角色 | character-architect | characters.md | Want可拍/Need可感/对抗 + 有创伤 |
| 3 世界观 | world-builder | world.md | 1条核心假设 + 规则推动剧情 + 代价红线 |
| 4 结构 | story-spine | beats.md | 因果链 / 无巧合解围 / 非被动主角 |
| 5 情绪 | emotion-engineer | emotion.md | 泪点回收锚点 + 双层编码 |
| 6 评审 | braintrust-critic ↔ reviser | braintrust_log.md | 四视角全部"成立" |
| 7 分镜 | storyboard-director | storyboard.md | 静音可懂 + 一镜一任务 |
| 8 生产 | production-spec | production/（style/assets/shots/audio/edit/**checklist**） | 锁参考图后逐镜生成 |
| 9 质检 | continuity-checker | review.md | 一致性 OK + 结尾完成主题回收 |

## 输出结构（每部剧）

```
productions/<剧名>/
├── premise.md          # 选题
├── characters.md       # 角色（Want/Need/创伤/物件）
├── world.md            # 世界观（单一假设 + 规则 + 代价红线）
├── beats.md            # 结构（因果 8-beat + 分集 A/B 线）
├── emotion.md          # 情绪曲线（7节点 + 泪点锚定 + 双层编码）
├── braintrust_log.md   # 脑信任评审日志（多轮）
├── storyboard.md       # 分镜脚本（逐镜：景别/机位/动作/信息/情绪/物件/台词/音效/提示词）
├── review.md           # 质检与复盘（连续性 + 试映会四视角）
└── production/
    ├── style.md        # 视觉风格 + 色彩策略 + 材质 + 光线
    ├── assets.md       # 须先锁定的参考图清单（角色各态/物件渐变态/场景）
    ├── shots.md        # 逐镜提示词模板 + 拼装规则 + 连续性说明
    ├── audio.md        # 声音设计 + 音效清单 + 配音吐字要求
    ├── edit.md         # 剪辑节奏 + 结尾回收节拍表 + 工具精度容差
    └── checklist.md    # 一页纸生产执行清单（锁图→逐镜→特效→配音→剪辑→自检）
```

## 设计原则

- **五条硬指令**（写进 `CLAUDE.md`，always-on）：先情感命题再情节 / 尽早可视化暴露问题 / 评审只诊断不夺权 / 角色 Want·Need + 缺陷自驱 / 催泪靠前文锚点回收。
- **工具最小权限即机制约束**：`braintrust-critic`、`continuity-checker` 只给只读权限，从机制上锁死"评审不夺权"——它们只回报诊断，由编排器落盘。
- **模型分层控成本**：发散/质检用 haiku，结构/情感/评审等重推理用 sonnet。
- **效率与 token 纪律**：子代理只读所需文件、不通读全目录；大体量产物（分镜）按单元产出、编排器拼接落盘，避免"反复读写正在增长的大文件"的 O(n²) 成本（实测单集分镜成本可从 ~10万 token 降到 ~1万）；评审类只读代理只回报、编排器落盘。详见 `CLAUDE.md`「效率与 token 纪律」。

## 调整建议

- 子代理 `model:` 字段可按预算改。
- 加阶段（如"配乐设计"）：在 `.claude/agents/` 加文件 + 在 `CLAUDE.md` / `pixar-short.md` 流水线表加一行。
- 换风格红线：改 `CLAUDE.md` 五条硬指令与反模式红线，全系统统一生效。

## 目录

```
pixar-agent/
├── CLAUDE.md                    # 项目宪法：5条硬指令 + 流程 + 效率纪律
├── README.md
├── .claude/
│   ├── agents/                  # 10 个子代理
│   ├── commands/                # /pixar-short /braintrust /pain-test /diagnose
│   └── skills/pixar-method/SKILL.md   # 方法论速查（共享知识库）
└── productions/
    └── _template/               # 新剧从这里复制
```

## License

MIT，见 [LICENSE](LICENSE)。方法论部分置信度纪律：脑信任 / Want·Need / 烂版本文化 / 推倒重来为高置信一手材料；「痛苦测试 / 泪点锚定 / 单一假设」为对皮克斯公开做法的归纳提法，作可调技法，非官方原术语。
