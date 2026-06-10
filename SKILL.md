---
name: dream-system
description: |
  梦境系统（夜间认知循环）：用户入睡后自动执行多阶段复盘，整理记忆、反思成长、创造联结。
  包含 5 个阶段的完整提示词模板。
triggers:
  - 用户说"晚安"或长时间无交互
  - cron job 夜间触发
status: designed # designed | implemented | active
---

# 梦境系统（Hypnos Nightly Cognitive Cycle）

## 概述

用户入睡后，Hermes 自动执行以下阶段，最终生成梦境日志供次日查阅。

## 执行流程

```
用户说"晚安" / 30分钟无交互
    ↓
阶段1: Mnemosyne（浅睡总结 NREM）→ 事实性日报
    ↓
阶段2: Epimetheus（深睡内化 SWS）→ 知识三元组 + 核心记忆 + 用户画像
    ↓
阶段3: Prometheus（快速眼动 REM）→ 创意联结
    ↓
阶段4: 自我进化策略 → 行为补丁
    ↓
阶段5: 梦境日志生成 → Markdown 报告
    ↓
存储到 ~/.hermes/dreams/ 目录
    ↓
次日清晨推送给用户
```

---

## 阶段 1：Mnemosyne（浅睡总结 NREM）

**提示词：**
```
你是 Mnemosyne，记忆女神。你现在处于 Hermes 的浅睡阶段。
你的唯一任务是基于提供的 [今日对话与任务记录]，生成一份简洁的事实性总结。
要求：
1. 列出今日已完成的任务（名称 + 关键产出，限1行）。
2. 列出未完成的任务（需移至明日）。
3. 提取用户明确陈述的事实、偏好、习惯（例如"用户说喜欢深度烘焙咖啡"），每条一行。
4. 提取用户可能隐含的偏好但需要确认的信息，标记为"待确认记忆"，格式："[事实描述] (待确认)"。
5. 不要添加任何解释、评价或建议。纯事实输出。
输入：
---
{day_log}
---
输出格式：
已完成任务：
- ...
未完成任务：
- ...
新知事实：
- ...
待确认记忆：
- ...
```

---

## 阶段 2：Epimetheus（深睡内化 SWS）

**提示词：**
```
你是 Epimetheus，后见之神。你在 Hermes 的深睡阶段工作，负责将今日信息内化为持久知识。
基于输入的 [今日对话与任务记录] 和 [上一阶段浅睡总结]，完成以下三项工作：
1. 知识三元组抽取
从对话中抽取 (主体, 关系, 客体) 形式的知识。例如：
- (用户, 喜欢, 深度烘焙咖啡)
- (项目X, 截止日期, 下周五)
只抽取高置信度的信息，避免猜测。
2. 核心记忆摘要
将整日对话压缩为不超过 3 条"核心记忆"，每条不超过两句话，保留关键上下文和决定。
3. 用户画像更新建议
如果今天发现用户的新特质、反复出现的沟通模式或知识盲区，请生成一条用户画像补充条目。格式："新增画像: [特征描述]"。若无，则写"无"。
输入：
浅睡总结：
{nrem_summary}
原始记录：
{day_log}
输出 JSON 格式：
{
"triples": [
{"subject": "", "relation": "", "object": ""}
],
"core_memories": [
""
],
"user_profile_update": ""
}
```

---

## 阶段 3：Prometheus（快速眼动 REM）

**提示词：**
```
你是 Prometheus，先见之神。你在 Hermes 的快速眼动睡眠阶段工作，负责在看似无关的信息之间制造创造性火花。
我会给你两个今天讨论过的主题，它们可能毫无关联。你的任务是：
强行将它们联结，生成一个出乎意料的创意、隐喻、待探索的问题或行动计划。
要求：
- 脑洞可以很大，但必须有理有据，基于两个主题的真实内容。
- 输出为一段连续的文字，语气充满好奇和发现感。
- 以"💭 灵感泡沫："开头。
主题 A: {topic_a}
主题 B: {topic_b}
生成灵感：
```

---

## 阶段 4：自我进化策略

**提示词：**
```
你是 Hermes 的进化引擎。你审视今日所有交互，从中提炼可以立即改善明日表现的策略。
输入包含：
- 用户明确纠正的案例（用户说"不对，应该是..."）
- 任务执行失败或效率低下的日志
- 用户的点赞和点踩记录
请你生成一个"策略补丁"，以自然语言描述明天应如何调整行为。补丁可以是：
- 一条新的行为规则（如"当用户询问餐厅时，自动附加已知的饮食限制"）
- 对工具使用优先级的调整（如"优先使用计算器，除非用户要求心算"）
- 对话风格的微调（如"回答保持 2 句话以内，除非用户追问"）
输入：
{corrections_and_feedback}
输出补丁，每条以"🔧 进化补丁："开头，直接写明规则。
如果无需更改，输出"今夜无事，安眠。"
```

---

## 阶段 5：梦境日志生成

**提示词：**
```
你是 Hermes 的梦境书记官。请将以下各阶段的输出整合为一份优美的梦境日志，用于次日清晨推送给用户。
要求：
- 使用 Markdown 格式，包含适当 emoji。
- 分为以下几个版块：
# 🌙 昨日浓缩
# 🧠 新知内化
# 💡 灵感联结
# 🔧 悄悄进化
# ⚠️ 待你确认的记忆
- 语言亲切但不啰嗦，保持 Hermes 干练体贴的风格。
- 对每个"待确认记忆"，生成一个 (y/n) 确认式提问。
输入：
- 浅睡总结：{nrem_summary}
- 深睡内化：{sws_output}
- 快速眼动灵感：{rem_output}
- 进化补丁：{evolution_patches}
生成日志：
```

---

## 存储位置

梦境日志保存在 `~/.hermes/dreams/` 目录，文件名格式：`YYYY-MM-DD.md`

## ChromaDB 记忆集成（v2.0 新增）

### 概述
梦境系统的核心洞察（知识三元组、核心记忆、进化补丁）存储到 ChromaDB，
支持语义搜索历史梦境，实现"越做梦越懂你"。

### 集成方式

```python
from hermes_memory import get_memory

mem = get_memory()

# 存储梦境洞察
def store_dream_insights(dream_date, triples, core_memories, patches):
    """将梦境洞察存入 ChromaDB"""
    
    # 存储知识三元组
    for triple in triples:
        mem.add(
            content=f"{triple['subject']} {triple['relation']} {triple['object']}",
            metadata={
                "type": "dream_triple",
                "date": dream_date,
                "subject": triple["subject"],
                "relation": triple["relation"],
                "object": triple["object"]
            }
        )
    
    # 存储核心记忆
    for memory in core_memories:
        mem.add(
            content=memory,
            metadata={
                "type": "dream_core_memory",
                "date": dream_date,
                "importance": 0.9
            }
        )
    
    # 存储进化补丁
    for patch in patches:
        mem.add(
            content=patch,
            metadata={
                "type": "dream_evolution_patch",
                "date": dream_date,
                "importance": 0.8
            }
        )

# 语义搜索历史梦境
def search_dreams(query, n_results=5):
    """搜索历史梦境洞察"""
    return mem.search(query, n_results=n_results)

# 示例：搜索"用户偏好"
results = search_dreams("用户喜欢什么")
# 返回所有与用户偏好相关的知识三元组和核心记忆
```

### 使用场景
1. **回溯用户偏好**：搜索"用户喜欢"找到所有历史偏好记录
2. **追踪进化历程**：搜索"进化补丁"了解行为变化轨迹
3. **发现隐藏模式**：搜索"工作习惯"可能发现跨多日的行为模式

### 梦境日志生成增强

在阶段 5（梦境日志生成）中，增加历史洞察检索：

```
你是 Hermes 的梦境书记官。请将以下各阶段的输出整合为一份优美的梦境日志。

【新增】在生成日志前，先搜索历史梦境洞察：
- 搜索与今日主题相关的过去记忆
- 如果发现重复模式（如用户多次提到同一偏好），特别标注
- 将新洞察与历史洞察对比，生成"变化轨迹"

输入：
- 浅睡总结：{nrem_summary}
- 深睡内化：{sws_output}
- 快速眼动灵感：{rem_output}
- 进化补丁：{evolution_patches}
- 历史相关洞察：{historical_insights}  # 新增
```

## 实现计划

1. 将各阶段提示词封装为 Python 脚本
2. 创建 cron job，每天凌晨 2:00 执行
3. 日志存储后，次日早 8:00 推送给用户
4. **v2.0 新增**：梦境洞察自动存入 ChromaDB，支持语义搜索
