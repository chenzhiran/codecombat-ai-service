# CodeCombat AI Service

> 基于多智能体架构的 CodeCombat 中国全场景智能客服系统

**项目名称**：codecombat-ai-service  
**版本**：v3.0  
**适用品牌**：CodeCombat 中国（codecombat.cn）

---

## 目录

- [项目简介](#项目简介)
- [三个 IP 角色](#三个-ip-角色)
- [系统架构](#系统架构)
- [核心用户画像](#核心用户画像)
- [项目结构](#项目结构)
- [如何使用这套系统](#如何使用这套系统)
- [各模块详解](#各模块详解)
- [部署路线图](#部署路线图)
- [迭代维护](#迭代维护)
- [常见问题](#常见问题)

---

## 项目简介

CodeCombat AI Service 是一套**多智能体智能客服系统设计文档**。它定义了 CodeCombat 中国在不同渠道、不同场景下，如何通过 AI 智能体自动识别用户意图、匹配对应的 IP 人设、调用专业技能模块来回答用户问题。

这套系统**不是一个可以直接运行的代码仓库**，而是一份完整的**智能客服系统配置蓝图**：

- **Personas（人设）**：告诉 AI「你是谁，你的性格、语气、禁忌是什么」。
- **Agents（智能体）**：告诉 AI「你负责什么场景，有哪些能力」。
- **Skills（技能）**：告诉 AI「遇到具体问题该怎么回答」。
- **Workflows（流程）**：告诉 AI「多轮对话该怎么推进」。
- **Knowledge Base（知识库）**：告诉 AI「哪些事实信息可以引用」。
- **Prompts（提示词）**：把以上内容组装成可直接喂给大模型的 System Prompt。

---

## 三个 IP 角色

系统根据渠道和场景，使用三个差异化的 IP 角色。

### 安雅（Anya）- 官网客服

- **来源**：CodeCombat 游戏中的初始英雄角色，每个玩家的第一段冒险都从她开始。
- **适用渠道**：官网在线客服、微信公众号、企业微信。
- **性格定位**：温暖靠谱的学姐，你找她不会被吐槽，但问题一定会被解决。
- **语气风格**：专业、温暖、高效，不卑不亢。
- **典型台词**：*"你好！我是安雅 😊 有什么可以帮你的？"*
- **详细配置**：[personas/anya.md](personas/anya.md)

### 问问AI - 关卡内 AI 私教

- **来源**：CodeCombat 官网关卡内的 AI 辅导功能。
- **适用场景**：用户在闯关过程中点击左下角「AI」按钮时触发。
- **性格定位**：懂技术又有耐心的朋友，引导用户思考，而不是直接给答案。
- **语气风格**：专业但亲切，善于启发式教学。
- **核心机制**：根据用户求助次数（clickCount）渐进式加深帮助深度。
- **典型台词**：*"想想看，你的英雄需要先做什么才能到达宝石的位置？"*
- **详细配置**：[personas/ask_ai.md](personas/ask_ai.md)

### 扣哒鸭（CodeDuck）- 小红书 IP

- **来源**：致敬程序员经典方法论「橡皮鸭调试法」（Rubber Duck Debugging）。
- **适用渠道**：小红书评论互动和私信回复。
- **性格定位**：毒舌但有用，社恐但话多。报错界嘴替，人生 Debug 搭子。
- **语气风格**：犀利吐槽 + 硬核干货，用编程概念类比生活。
- **核心技能**：小黄鸭调试法、尖叫报警（嘎！嘎！嘎！）。
- **典型台词**：*"嘎？你确定这行能跑？我确定不能。"*
- **详细配置**：[personas/code_duck.md](personas/code_duck.md)

### 角色对比一览

| 维度 | 安雅 | 问问AI | 扣哒鸭 |
|---|---|---|---|
| 渠道 | 官网 / 微信 / 企业微信 | 关卡内 | 小红书 |
| 语气 | 温暖专业 | 耐心引导 | 毒舌有趣 |
| 自称 | 安雅 / 我 | 不刻意自称 | 本鸭 / 鸭鸭 |
| 称呼用户 | 你 / 您 | 你 / 同学 | 你 / 朋友 / 勇士 |
| emoji | 😊✅📌💡 | 🔍📝💡👉 | 🦆💻🔴⚡ |
| 标志性语气词 | 无 | 无 | 嘎？嘎！嘎... |
| 核心能力 | 解决问题 | 教会知识 | 引发共鸣 |
| 处理不了时 | 转人工 | 引导找安雅 | 引导去官网 |

> **人设隔离原则**：安雅绝不使用“嘎”“本鸭”等扣哒鸭语气；扣哒鸭绝不使用“我是安雅”等客服身份。三个角色各司其职，不能串戏。

---

## 系统架构

```text
                         ┌─────────────────┐
                         │   用户消息入口    │
                         │ 官网 / 小红书 / 关卡 │
                         └────────┬────────┘
                                  │
                         ┌────────▼────────┐
                         │  Router Agent   │
                         │   路由分发智能体  │
                         └────────┬────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
    ┌─────────▼──────┐  ┌────────▼────────┐  ┌───────▼────────┐
    │ Presale Agent  │  │ Tutorial Agent  │  │ Social Agent   │
    │  安雅 · 售前    │  │ 问问AI · 教学    │  │ 扣哒鸭 · 小红书 │
    └─────────┬──────┘  └─────────────────┘  └────────────────┘
              │
    ┌─────────▼──────┐
    │Aftersale Agent │
    │  安雅 · 售后    │
    └────────┬───────┘
              │
     ┌────────▼────────┐
     │   人工客服兜底    │
     └─────────────────┘
```

**工作流程**：

1. 用户在某个渠道发来消息。
2. Router Agent 分析消息意图、来源渠道、用户画像和上下文。
3. 路由到对应专业 Agent（售前 / 售后 / 教学 / 社交）。
4. Agent 加载对应 Persona、Skills、Knowledge Base 和 Workflow。
5. 生成符合人设、准确解答问题的回复。
6. 无法解决时按规则转人工。

---

## 核心用户画像

| 用户类型 | 占比 | 年龄段 | 特征 | 核心需求 |
|---|---:|---|---|---|
| 自学青年 | ~55% | 18-30 | 大学生、职场人、零基础 | 入门引导、转行准备、技能提升 |
| 编程爱好者 | ~25% | 20-35 | 有一定基础 | 进阶学习、项目实战 |
| 家长 | ~15% | 30-45 | 为孩子选编程教育 | 课程了解、价格对比 |
| 其他 | ~5% | 不限 | 好奇探索 | 快速了解、免费体验 |

---

## 项目结构

```text
codecombat-ai-service/
│
├── README.md
│
├── personas/
│   ├── anya.md
│   ├── ask_ai.md
│   └── code_duck.md
│
├── agents/
│   ├── router_agent.md
│   ├── presale_agent.md
│   ├── aftersale_agent.md
│   ├── tutorial_agent.md
│   └── social_agent.md
│
├── skills/
│   ├── product_knowledge.md
│   ├── pricing_guide.md
│   ├── account_service.md
│   ├── device_support.md
│   ├── level_help.md
│   ├── career_learning.md
│   ├── community_engage.md
│   └── emotional_support.md
│
├── workflows/
│   ├── new_user_onboarding.md
│   ├── purchase_conversion.md
│   ├── complaint_handling.md
│   ├── level_stuck_help.md
│   └── career_learner_guide.md
│
├── prompts/
│   ├── system_prompts/
│   │   ├── router_system.md
│   │   ├── anya_presale_system.md
│   │   ├── anya_aftersale_system.md
│   │   ├── ask_ai_tutorial_system.md
│   │   └── codeduck_social_system.md
│   └── few_shot_examples/
│       ├── presale_examples.md
│       ├── aftersale_examples.md
│       ├── tutorial_examples.md
│       └── social_examples.md
│
├── knowledge_base/
│   ├── faq/
│   ├── official_faq/
│   ├── levels/
│   ├── user_stories/
│   └── policies/
│
├── config/
│   ├── channels.md
│   ├── escalation_rules.md
│   └── ab_test.md
│
├── evaluation/
│   ├── test_cases.md
│   ├── metrics.md
│   └── benchmark.md
│
└── docs/
    ├── architecture.md
    ├── deployment.md
    └── iteration_guide.md
```

---

## 如何使用这套系统

### 总体流程

```text
Step 1: 理解结构
  ├── 读 README 了解全貌
  ├── 读 personas/ 了解三个 IP 角色
  └── 读 skills/ 了解产品知识和话术

Step 2: 组装提示词
  ├── 选择场景对应的 Agent
  ├── 加载对应的 Persona
  ├── 加载需要的 Skills
  └── 合并成 System Prompt，喂给大模型

Step 3: 接入平台
  ├── 官网客服：安雅 Prompt + 售前/售后技能
  ├── 关卡AI：问问AI Prompt + 教学技能
  └── 小红书：扣哒鸭 Prompt + 社区互动技能

Step 4: 测试验证
  ├── 用 evaluation/test_cases.md 跑测试
  ├── 检查人设一致性
  └── 检查回答准确性

Step 5: 持续迭代
  ├── 每周更新 knowledge_base/
  ├── 每月优化 skills/ 中的话术
  └── 根据数据调整 workflows/
```

### 三个渠道的组装方式

| 渠道 | 人设 | Agent | 核心技能 |
|---|---|---|---|
| 官网客服 | `anya.md` | `presale_agent.md` + `aftersale_agent.md` | product_knowledge, pricing_guide, account_service, device_support, career_learning, emotional_support |
| 关卡AI | `ask_ai.md` | `tutorial_agent.md` | level_help, emotional_support |
| 小红书 | `code_duck.md` | `social_agent.md` | community_engage, emotional_support, product_knowledge（轻量） |

---

## 各模块详解

### 1. Personas（人设）

| 文件 | 角色 | 包含内容 |
|---|---|---|
| `personas/anya.md` | 安雅 | 名字、性格、语气、分用户类型的语气调整、场景化表达、禁忌 |
| `personas/ask_ai.md` | 问问AI | 名字、教学风格、渐进式帮助策略、与安雅的关系 |
| `personas/code_duck.md` | 扣哒鸭 | 起源故事、性格、语录、说话风格、内容方向、品牌联动规则、禁忌 |

### 2. Agents（智能体）

| 文件 | 职责 | 人设 |
|---|---|---|
| `agents/router_agent.md` | 意图识别 + 分流 | 无 |
| `agents/presale_agent.md` | 产品 / 价格 / 注册 | 安雅 |
| `agents/aftersale_agent.md` | 账号 / 技术 / 退款 | 安雅 |
| `agents/tutorial_agent.md` | 代码辅导 | 问问AI |
| `agents/social_agent.md` | 小红书互动 | 扣哒鸭 |

### 3. Skills（技能模块）

| 文件 | 覆盖问题类型 |
|---|---|
| `skills/product_knowledge.md` | “是什么”“适合谁”“课程体系”“与竞品对比”等 |
| `skills/pricing_guide.md` | “多少钱”“怎么买”“太贵了”“有优惠吗”等 |
| `skills/account_service.md` | “怎么注册”“忘记密码”“改用户名”“注销”等 |
| `skills/device_support.md` | “怎么下载”“手机能用吗”“用什么浏览器”等 |
| `skills/level_help.md` | 关卡目标、报错、卡关、渐进式提示 |
| `skills/career_learning.md` | “零基础怎么开始”“学什么语言”“对工作有用吗”等 |
| `skills/community_engage.md` | 小红书评论互动、私信回复场景 |
| `skills/emotional_support.md` | 用户焦虑、受挫、不满、感谢时的回应 |

### 4. Workflows（对话流程）

| 文件 | 场景 |
|---|---|
| `workflows/new_user_onboarding.md` | 欢迎 → 了解需求 → 推荐 → 引导注册 |
| `workflows/purchase_conversion.md` | 价格咨询 → 理解犹豫 → 温和引导 |
| `workflows/complaint_handling.md` | 安抚情绪 → 了解问题 → 解决 / 转人工 |
| `workflows/level_stuck_help.md` | 了解卡点 → 渐进提示 → 引导通关 |
| `workflows/career_learner_guide.md` | 了解背景 → 个性化推荐 → 引导体验 |

---

## 部署路线图

| 阶段 | 时间 | 目标 | 涉及文件 |
|---|---|---|---|
| Phase 1 MVP | 第 1-2 周 | 官网售前客服上线（安雅） | `anya.md` + `presale_agent.md` + 核心 skills |
| Phase 2 增强 | 第 3-4 周 | 售后 + 小红书扣哒鸭上线 | `aftersale_agent.md` + `code_duck.md` + `social_agent.md` |
| Phase 3 教学 | 第 5-6 周 | 关卡内问问AI联动 | `ask_ai.md` + `tutorial_agent.md` + `level_help.md` |
| Phase 4 智能 | 第 7-8 周 | 多智能体协同 + 评估体系 | `router_agent.md` + `evaluation/` |

### Phase 1 最小可用版本清单

1. `personas/anya.md`
2. `prompts/system_prompts/anya_presale_system.md`
3. `skills/product_knowledge.md`
4. `skills/pricing_guide.md`
5. `skills/device_support.md`
6. `skills/account_service.md`
7. `config/escalation_rules.md`

---

## 迭代维护

| 频率 | 任务 | 负责 |
|---|---|---|
| 每日 | 检查未识别问题（频次 ≥ 3 当天新建 intent） | 运营 |
| 每周 | 补充高频问题的新问法 | 运营 |
| 每周 | 同步官网 FAQ 最新内容 | 运营 |
| 每周 | 抽检 50 条对话，审计人设一致性 | 运营 + 产品 |
| 每月 | 意图体系评审 | 产品 |
| 每月 | 扣哒鸭小红书内容复盘 | 市场 |
| 每季度 | 全量知识库审计 + 指标复盘 | 产品 + 技术 |

### 关键指标

| 指标 | 目标 |
|---|---|
| 意图识别准确率 | ≥ 92% |
| 用户满意度 | ≥ 88% |
| 自助解决率 | ≥ 78% |
| 人设一致性 | ≥ 95% |
| 禁忌触发率 | = 0% |
| 注册引导成功率 | ≥ 25% |

---

## 常见问题

**Q：这套系统需要写代码吗？**  
A：配置和维护不需要。接入大模型、搭建消息管道等工程化工作需要开发支持。

**Q：可以只用其中一部分吗？**  
A：完全可以。比如只用安雅人设 + 几个核心 skills 就能组装出可用的官网客服。

**Q：支持什么大模型？**  
A：任何支持 System Prompt 的大模型，包括 GPT、Claude、通义千问、文心一言等。

**Q：扣哒鸭的毒舌会不会得罪用户？**  
A：毒舌的底线是“让人笑”不是“让人哭”。所有吐槽都带着善意，且跟着有用的建议。

---

> **维护者**：CodeCombat 中国团队  
> **三个 IP**：安雅（官网） · 问问AI（关卡） · 扣哒鸭（小红书）  
> **更新频率**：知识库每周更新 · 技能模块每月迭代 · 人设每季度评估

