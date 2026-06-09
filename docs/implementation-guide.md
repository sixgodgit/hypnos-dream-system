# 📚 部署指南

## 前置条件

- [x] Hermes Agent 已部署并运行
- [x] Cron job 功能可用
- [x] Lark/飞书 已连接
- [x] session_search 可用（FTS5）

## 部署步骤

### Step 1: 创建目录

```bash
mkdir -p ~/.hermes/dreams
```

### Step 2: 配置 Cron Jobs

#### 夜间执行（凌晨 2:00）

在 Hermes 中执行：

```python
cronjob(
    action="create",
    name="梦境系统-夜间执行",
    schedule="0 2 * * *",
    deliver="local",
    enabled_toolsets=["terminal", "file", "session_search"],
    prompt="[使用 prompts/01-05 的完整提示词]"
)
```

#### 清晨推送（早 8:00）

```python
cronjob(
    action="create",
    name="梦境日志-清晨推送",
    schedule="0 8 * * *",
    deliver="feishu",
    enabled_toolsets=["file", "terminal"],
    prompt="[读取昨日日志并推送]"
)
```

### Step 3: 验证

```bash
# 检查目录
ls -la ~/.hermes/dreams/

# 手动触发测试
cronjob(action="run", job_id="xxx")
```

## 调优参数

| 参数 | 默认值 | 说明 |
|:----:|:------:|:----:|
| 空闲触发延迟 | 30分钟 | 无交互后等待时间 |
| 深夜判断时间 | 22:00-06:00 | 只在深夜触发空闲检测 |
| 核心记忆上限 | 3条 | 每日最多保留3条核心记忆 |
| 进化补丁上限 | 3条 | 每日最多生成3条补丁 |
| 日志保留天数 | 30天 | 超过30天的日志自动清理 |

## 故障排除

### 问题：session_search 无结果
**原因**：当天无对话记录
**解决**：输出"今夜无事，安眠"，跳过后续阶段

### 问题：日志文件写入失败
**原因**：权限问题或磁盘满
**解决**：检查 `~/.hermes/dreams/` 目录权限，清理旧日志

### 问题：Lark 推送失败
**原因**：Token 过期或网络问题
**解决**：检查 Lark 连接状态，重新配置 Token
