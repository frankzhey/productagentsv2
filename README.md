# ProductPortfolio — AI Agent 协作工作流

> BCChina 产品研发一体化 Agent 框架  
> 更新时间：2026-05-02

---

## 仓库定位

本仓库管理 BCChina 从**产品规划 → UX 设计 → 工程评审 → 任务拆分 → Wiki 发布**的全流程 AI Agent 协作体系。所有 Agent 遵循 `.github/copilot-instructions.md` 全局规则，通过 handoff 机制串联。

---

## Agent 工作流总览

```
                    ┌─────────────────────┐
                    │  Knowledge Retriever │  ← Epic Kickoff 时调用一次
                    │  搜索 ADO Wiki 历史   │
                    │  生成 context-memo.md │
                    └────────┬────────────┘
                             │ handoff（memo 文件共享）
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼
   Product Planner    UX Prototyper       Eng Reviewer
          │
          ▼
┌──────────────────────────────────────────────────┐
│  Product Planner  (v2.4.0)                       │
│                                                  │
│  1. Read SKILL §1-§5 + Rules/{project}-rules.md │
│  2. 写 AC → PRD 落盘 PRD/{project}/{epic}.md    │
│  3. FCS > 10? ──→ Story Splitter (sub-agent)    │
│     │              Read SKILL → 写 AC            │
│     │              返回 Stories                   │
│     ▼                                            │
│  4. 整合 Story                                   │
│  5. Quality Gate 10.5（SKILL §1/§2/§3 自检）     │
│  6. Step 11 规则沉淀 → Rules/{project}-rules.md │
└──┬───────────────┬───────────────────────────────┘
   │ handoff       │ handoff
   ▼               ▼
┌────────────┐  ┌──────────────────────────────────┐
│ UX Proto-  │  │  Eng Reviewer  (v2.1.0)          │
│ typer      │  │                                  │
│ (v2.0.0)   │  │  Read SKILL §1/§2/§3             │
│            │  │  Section 17.0 AC 合规校验（阻塞） │
│ HTML +     │  │  输出 Engineering Review          │
│ UX.md      │  │                                  │
└──┬─────┬───┘  └──┬───────────────┬───────────────┘
   │     │         │ handoff       │ handoff
   │     │         ▼               ▼
   │     │  ┌────────────┐  ┌──────────────┐
   │     │  │ Task       │  │              │
   │     │  │ Planner    │  │              │
   │     │  │ 任务拆分    │  │              │
   │     │  └─────┬──────┘  │              │
   │     │        │         │              │
   ▼     ▼        ▼         ▼              │
┌─────────────────────────────────────┐    │
│        Wiki Publisher  (v2.0.0)     │◄───┘
│                                     │
│  ADO Wiki 路径：                     │
│  /{epic-name}/                      │  ← PRD
│  /{epic-name}/ui-prototype          │  ← UX
│  /{epic-name}/engineering-review    │  ← Eng Review
│  /{epic-name}/task-planning         │  ← Task Plan
└─────────────────────────────────────┘
```

---

## Agent 清单

| Agent | 版本 | 职责 | 子 Agent | Handoff 目标 |
|-------|------|------|----------|-------------|
| **Knowledge Retriever** | — | Epic Kickoff 时检索 ADO Wiki 历史，生成 `context-memo.md` | — | Product Planner / UX Prototyper / Eng Reviewer |
| **Product Planner** | v2.4.0 | PRD 编写、AC 生成、PRD 落盘、规则沉淀 | Story Splitter | UX Prototyper / Eng Reviewer / Wiki Publisher |
| **Story Splitter** | v2.1.0 | Feature 复杂度评估 (FCS)、Story 拆分与 AC 补全 | — | （返回 Product Planner） |
| **UX Prototyper** | v2.0.0 | UX 文档 + HTML 原型生成 | — | Eng Reviewer / Wiki Publisher |
| **Eng Reviewer** | v2.1.0 | 工程评审、AC 合规校验 (§17.0)、技术方案 | — | Task Planner / Wiki Publisher |
| **Task Planner** | — | 任务拆分、估算、依赖识别 | — | Wiki Publisher |
| **Wiki Publisher** | v2.0.0 | 文档类型识别、Wiki 路径生成、ADO Wiki 发布 | — | — |

---

## 核心设计决策

| # | 决策 | 说明 |
|---|------|------|
| 1 | AC 规则单一来源 | `skills/ac-writing-spec/SKILL.md` 为唯一权威；Product Planner / Story Splitter / Eng Reviewer 均通过强制 Read 引用 |
| 2 | instructions 与 agent 职责分离 | `product.instructions.md` 只定义文件级 contract（PRD 合格标准）；agent 定义行为流程 |
| 3 | 编号体系 | A/B/C 跨章节锚点 + ①②③④⑤ 章节内序号混合 |
| 4 | Eng Reviewer AC 强校验 | Section 17.0 强制阻塞性校验，不合规 → flag 到 Risks |
| 5 | 强制依赖加载 | 三个 agent 均在 Step 0 前置 Read SKILL，确保 Copilot 加载链路确定性 |
| 6 | context-memo 共享 | Knowledge Retriever 仅 Epic 启动调用一次，后续 Agent 读文件而非重复查询 ADO |

---

## 文件结构

```
.github/
  copilot-instructions.md          ← 全局规则（最高优先级）
  agents/
    product-planner.agent.md       ← PRD 生成
    story-splitter.agent.md        ← Story 拆分（Product Planner 子 Agent）
    ux-prototyper.agent.md         ← UX/UI 设计 + HTML 原型
    eng-reviewer.agent.md          ← 工程评审 + AC 合规
    task-planner.agent.md          ← 任务拆分与估算
    wiki-publisher.agent.md        ← ADO Wiki 发布
    knowledge-retriever.agent.md   ← Epic Kickoff 历史知识检索
  instructions/
    product.instructions.md        ← PRD 文件级 contract
    engineering.instructions.md    ← 工程设计规范
    frontend.instructions.md       ← 前端/UI 规范
skills/
  ac-writing-spec/SKILL.md         ← AC 写作规范（唯一权威）
  claud-frontend/SKILL.md          ← 前端 UI 生成能力
  premium-frontend-ui/SKILL.md     ← 设计规范能力
PRD/                               ← PRD 落盘目录
  {project}/{epic-slug}.md
Rules/                             ← 规则沉淀目录
  {project}-rules.md
UI/                                ← HTML 原型文件
docs/                              ← UX / Engineering Review 等文档
context-memo.md                    ← Epic 级历史知识缓存（Knowledge Retriever 生成）
```

---

## 指令优先级

1. 用户当前明确要求
2. `.github/copilot-instructions.md`（全局规则）
3. `.github/instructions/*.instructions.md`（局部规则）
4. `.github/agents/*.agent.md`（角色规则）
5. `skills/` 内部说明

---

## Wiki 发布路径

| 文档类型 | Wiki 路径 | 发布 Agent |
|---------|----------|-----------|
| PRD | `/{epic-name}/` | Wiki Publisher |
| UX 文档 | `/{epic-name}/ui-prototype` | Wiki Publisher |
| Engineering Review | `/{epic-name}/engineering-review` | Wiki Publisher |
| Task Planning | `/{epic-name}/task-planning` | Wiki Publisher |

---

## 版本记录

| 日期 | 变更 |
|------|------|
| 2026-05-02 | README 重写为工作流架构文档，反映 V2 agent 体系实际状态 |
| 2026-04-28 | V2 重构：AC 规则抽象到 SKILL、instructions 瘦身、Eng Reviewer 增加 §17.0 合规校验 |
| 2026-04-16 | 初始 Agent 体系建立（Product Planner / UX Prototyper / Eng Reviewer / Wiki Publisher） |
- 「不在本文件范围」对照表

---

## 五、Review 步骤建议

1. **先读 SKILL.md**（这是新增最重要的文件）
2. **再读 product.instructions.md**（确认瘦身后是否还能 cover 文件级要求）
3. **diff product-planner.agent.md** 与原文件，确认删除部分是否都已移到 SKILL，新增的 reference 是否到位
4. **diff story-splitter.agent.md** 与原文件，特别注意 B-3 修复
5. **diff eng-reviewer.agent.md** 与原文件，主要看 Section 17.0 的合理性

---

## 六、应用到 Live 文件的步骤

Review 通过后告诉我，我会执行：

```
1. cp V2/skills/ac-writing-spec/SKILL.md → PM agents/skills/ac-writing-spec/SKILL.md
2. cp V2/instructions/product.instructions.md → PM agents/instructions/product.instructions.md
3. cp V2/product-planner.agent.md → PM agents/product-planner.agent.md
4. cp V2/story-splitter.agent.md → PM agents/story-splitter.agent.md
5. cp V2/eng-reviewer.agent.md → PM agents/eng-reviewer.agent.md
6. （可选）archive V2/ folder 或保留作为版本快照
```

---

## 七、需要你确认的事项

请逐项 review 后告诉我：

- [ ] SKILL.md 内容是否完整？编号体系是否接受？
- [ ] product.instructions.md 瘦身是否过度？还有哪些必须留？
- [ ] product-planner.agent.md 删除项是否都接受？还有哪些不该删？
- [ ] story-splitter.agent.md B-3 修复是否符合预期？
- [ ] eng-reviewer.agent.md Section 17.0 AC 合规校验是否要这么严格（阻塞性）？还是仅 warning？
- [ ] 5 个决策默认值是否需要调整？

确认后我执行最终覆盖。
