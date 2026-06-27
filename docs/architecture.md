# 系统架构说明

## 总览

CodeCombat AI Service 采用多智能体架构。Router Agent 负责识别用户意图和渠道，业务 Agent 负责具体回答，Persona 控制语气，Skills 和 Knowledge Base 提供事实和话术，Workflows 控制多轮对话推进。

## 核心组件

| 组件 | 作用 |
|---|---|
| Router Agent | 判断渠道、意图、用户画像和目标智能体 |
| Presale Agent | 官网售前咨询、产品介绍、价格和注册引导 |
| Aftersale Agent | 账号、技术、订单、退款和投诉处理 |
| Tutorial Agent | 关卡内代码辅导和渐进式提示 |
| Social Agent | 小红书评论、私信和社区互动 |
| Personas | 控制身份、语气、禁忌和表达风格 |
| Skills | 按意图组织标准回答和处理策略 |
| Workflows | 处理多轮对话和状态推进 |
| Evaluation | 回归测试和质量评估 |

## 2026 推荐运行架构

```text
用户消息
  │
  ▼
输入标准化：channel / page / user_profile / history_summary
  │
  ▼
Router Agent：结构化输出 target_agent + intent + confidence
  │
  ▼
检索：Persona + Skills + Workflow + Knowledge Base
  │
  ▼
业务 Agent 生成回复
  │
  ▼
安全和质量检查：禁忌 / 转人工 / 事实来源
  │
  ▼
发送回复 + 记录日志 + 更新评估数据
```

## 结构化路由输出建议

```json
{
  "target_agent": "PresaleAgent",
  "persona": "anya",
  "intent_id": "price_inquiry",
  "confidence": 0.91,
  "requires_human": false,
  "missing_slots": [],
  "next_action": "answer_with_pricing_guide"
}
```

## 设计原则

- Prompt 短而稳定，知识通过检索注入。
- 事实、价格、政策集中维护，避免散落在多个提示词中。
- 高风险问题优先转人工，不让模型自由发挥。
- 所有 Agent 都要受全局禁忌和渠道规则约束。



