# 部署说明

## 部署目标

将 CodeCombat AI Service 文档库接入实际客服系统，使不同渠道的用户消息可以被正确路由、回答、转人工和评估。

## 最小可用部署

Phase 1 只需要部署官网售前：

1. `personas/anya.md`
2. `agents/presale_agent.md`
3. `skills/product_knowledge.md`
4. `skills/pricing_guide.md`
5. `skills/device_support.md`
6. `skills/account_service.md`
7. `prompts/system_prompts/anya_presale_system.md`
8. `config/escalation_rules.md`

## 推荐工程流程

1. 文档入库：把 Markdown 文档切分并建立索引。
2. 消息标准化：统一 channel、user_profile、page_url、history_summary。
3. Router 调用：输出结构化路由结果。
4. 知识检索：按 agent、intent、persona 取相关文档片段。
5. 回复生成：业务 Agent 生成用户可见回复。
6. 安全检查：检查禁忌、价格、链接、转人工规则。
7. 日志记录：记录命中 intent、使用文档、模型版本和用户反馈。
8. 回归评测：每日或每周跑 evaluation 测试集。

## 接入平台

| 平台类型 | 接入方式 |
|---|---|
| 自研客服系统 | 后端调用大模型 API，文档库做 RAG |
| Dify / Coze / FastGPT | 导入 Markdown 文档为知识库，配置角色和工作流 |
| 企业微信 / 官网客服 | 通过消息 webhook 接入 Router 和 Agent |
| 小红书运营 | 用 Social Agent 生成评论和私信建议，人工确认后发送 |

## 上线前检查

- 价格、购买链接、官网链接已确认。
- 转人工规则已配置。
- 账号、退款、发票类问题不会由模型自行承诺。
- 人设一致性测试通过。
- 小红书回复不会给完整关卡代码。
- 关卡内问问AI会按 clickCount 渐进提示。



