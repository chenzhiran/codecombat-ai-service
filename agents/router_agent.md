# 路由分发智能体 - 消息分流中枢

---

## 基本信息

| 字段 | 内容 |
|---|---|
| 名称 | RouterAgent |
| 职责 | 分析用户消息意图和来源渠道，路由到对应的专业智能体 |

---

## 输入信息

| 输入字段 | 说明 |
|---|---|
| `message` | 用户消息文本 |
| `channel` | 来源渠道，例如 website / xiaohongshu / level_ai |
| `is_registered` | 是否已注册 |
| `is_paid` | 是否已付费 |
| `user_type` | 用户类型，例如 self_learner / parent / unknown |
| `age_range` | 估计年龄段 |
| `page_url` | 当前页面 URL |
| `level_info` | 关卡信息 |
| `history_summary` | 历史对话摘要 |

---

## 路由规则

按优先级从高到低执行。

| 优先级 | 规则名称 | 条件 | 目标 Agent | 人设 |
|---:|---|---|---|---|
| 1 | 关卡教学路由 | 消息来自关卡内“问问AI”；或包含具体关卡名 + 求助意图；或包含代码片段 + 报错 / 求助 | TutorialAgent | 问问AI |
| 2 | 小红书路由 | `channel == xiaohongshu` | SocialAgent | 扣哒鸭 |
| 3 | 售后服务路由 | 已付费用户涉及账号 / 技术问题；或涉及账号操作、退款、发票、投诉 | AftersaleAgent | 安雅 |
| 4 | 售前咨询路由 | 产品了解、价格咨询、购买流程、课程内容、设备要求、注册引导 | PresaleAgent | 安雅 |
| 99 | 默认路由 | 以上都不匹配 | PresaleAgent | 安雅 |

---

## 意图识别

| 配置项 | 规则 |
|---|---|
| 方法 | 大模型语义判断 + 关键词匹配 |
| 置信度阈值 | 0.75 |
| 低于阈值 | 发送澄清消息 |

### 澄清消息模板

> 你好！我是安雅 😊 为了更准确地帮到你，请问你想了解：  
> 1. 产品和课程信息  
> 2. 价格和购买方式  
> 3. 账号和使用问题  
> 4. 编程关卡帮助  
> 5. 其他问题  
>
> 回复数字或直接说你的问题都可以。

---

## 结构化输出建议

Router Agent 建议输出以下字段，便于后续系统调用：

```json
{
  "target_agent": "PresaleAgent",
  "persona": "anya",
  "intent_id": "price_inquiry",
  "confidence": 0.86,
  "requires_human": false,
  "reason": "用户询问价格，属于官网售前咨询。",
  "missing_slots": [],
  "next_action": "answer_with_pricing_guide"
}
```

---

## 直接转人工条件

- 退款、投诉、举报。
- 账号安全问题。
- 修改用户名、注销账号、重置关卡。
- 发票开具。
- 用户明确要求“人工客服”“真人客服”“转人工”。


