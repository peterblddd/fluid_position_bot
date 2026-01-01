# 自动监控功能说明

## 功能概述

Fluid Position Monitor Bot 现在支持自动监控功能！Bot 会每 30 分钟自动检查你监控的地址，如果 Health Factor 低于设定的阈值，会自动发送 Telegram 提醒。

---

## 核心功能

### ✅ 自动监控
- **检查频率**: 每 30 分钟
- **监控范围**: 所有链（ETH, Base, Arbitrum, Polygon, Plasma）
- **智能提醒**: 每个 Position 每小时最多提醒一次（避免骚扰）

### ✅ 可自定义阈值
- **Alert 阈值** (🟠): 默认 HF < 1.15
- **Critical 阈值** (🔴): 默认 HF < 1.05
- 可以为每个地址设置不同的阈值

### ✅ 实时 Telegram 通知
- 自动发送格式化的提醒消息
- 包含完整的 Position 信息
- 提供操作建议

---

## 使用方法

### 1. 开始监控地址

**命令格式:**
```
/monitor <address> [alert_threshold] [critical_threshold]
```

**示例:**

**使用默认阈值:**
```
/monitor 0x1247739ac8e238D21574D18dEAce064675546cfC
```
- Alert: HF < 1.15
- Critical: HF < 1.05

**自定义阈值:**
```
/monitor 0x1247739ac8e238D21574D18dEAce064675546cfC 1.20 1.10
```
- Alert: HF < 1.20
- Critical: HF < 1.10

**成功响应:**
```
✅ Monitoring Started

📍 Address: 0x1247...6cfC
🟠 Alert Threshold: HF < 1.15
🔴 Critical Threshold: HF < 1.05

⏱️ Checks every 30 minutes
🔔 You'll receive alerts via Telegram
```

---

### 2. 查看监控列表

**命令:**
```
/mymonitors
```

**响应示例:**
```
📡 Your Monitored Addresses

1. 0x1247...6cfC
   🟠 Alert: HF < 1.15
   🔴 Critical: HF < 1.05

2. 0x478E...1d61
   🟠 Alert: HF < 1.20
   🔴 Critical: HF < 1.10

Use /unmonitor <address> to stop monitoring.
```

---

### 3. 停止监控

**命令格式:**
```
/unmonitor <address>
```

**示例:**
```
/unmonitor 0x1247739ac8e238D21574D18dEAce064675546cfC
```

**成功响应:**
```
✅ Monitoring Stopped

📍 Address: 0x1247...6cfC

You will no longer receive alerts for this address.
```

---

## 提醒消息格式

### 🟠 WARNING Alert 示例

当 HF 在 1.05 - 1.15 之间时：

```
🟠 WARNING ALERT!

📊 Position #9540
🔗 Chain: Ethereum
━━━━━━━━━━━━━━━━━━━━━

⚠️ Health Factor: 1.072619

📈 Risk Metrics:
   Collateral Ratio: 85.77%
   Liquidation Threshold: 92.00%
   
💰 Collateral: 3,495.4938 syrupUSDC
   💵 $4,001.02

💳 Debt: 3,431.7302 GHO
   💵 $3,431.73

━━━━━━━━━━━━━━━━━━━━━

⚠️ Action Recommended
Your position is approaching liquidation risk.
Monitor closely or adjust your position.
```

### 🔴 CRITICAL Alert 示例

当 HF < 1.05 时：

```
🔴 CRITICAL ALERT!

📊 Position #9540
🔗 Chain: Ethereum
━━━━━━━━━━━━━━━━━━━━━

⚠️ Health Factor: 1.042619

📈 Risk Metrics:
   Collateral Ratio: 88.50%
   Liquidation Threshold: 92.00%
   
💰 Collateral: 3,495.4938 syrupUSDC
   💵 $4,001.02

💳 Debt: 3,431.7302 GHO
   💵 $3,431.73

━━━━━━━━━━━━━━━━━━━━━

🚨 IMMEDIATE ACTION REQUIRED!
Your position is at high risk of liquidation.
Consider:
• Adding more collateral
• Repaying some debt
```

---

## 工作原理

### 监控流程

```
每 30 分钟
    ↓
获取所有监控地址
    ↓
对每个地址:
    ├─ 在所有链上查询 Positions
    ├─ 检查每个 Position 的 HF
    ├─ 如果 HF < 阈值
    │   ├─ 记录到数据库
    │   ├─ 检查是否在冷却期（1小时）
    │   └─ 发送 Telegram 提醒
    └─ 继续下一个
```

### 防骚扰机制

- **冷却期**: 同一个 Position 每小时最多提醒一次
- **智能判断**: 只在 HF 低于阈值时才提醒
- **状态追踪**: 记录所有提醒历史

---

## 数据库结构

### monitored_addresses 表
存储监控的地址和阈值设置

| 字段 | 类型 | 说明 |
|------|------|------|
| id | INTEGER | 主键 |
| user_id | INTEGER | Telegram 用户 ID |
| address | TEXT | 监控的地址 |
| alert_threshold | REAL | Alert 阈值 (默认 1.15) |
| critical_threshold | REAL | Critical 阈值 (默认 1.05) |
| created_at | TIMESTAMP | 创建时间 |

### alert_history 表
记录所有发送的提醒

| 字段 | 类型 | 说明 |
|------|------|------|
| id | INTEGER | 主键 |
| user_id | INTEGER | Telegram 用户 ID |
| position_id | INTEGER | Position ID |
| health_factor | REAL | 当时的 HF |
| alert_type | TEXT | WARNING 或 CRITICAL |
| message | TEXT | 提醒消息 |
| created_at | TIMESTAMP | 提醒时间 |

### position_snapshots 表
记录 Position 的历史快照

| 字段 | 类型 | 说明 |
|------|------|------|
| id | INTEGER | 主键 |
| position_id | INTEGER | Position ID |
| owner_address | TEXT | 所有者地址 |
| health_factor | REAL | HF |
| ratio | REAL | 抵押率 |
| supply_usd | REAL | 抵押品价值 (USD) |
| borrow_usd | REAL | 借款价值 (USD) |
| created_at | TIMESTAMP | 快照时间 |

---

## 技术实现

### 核心模块

**monitor.py**
- `PositionMonitor` 类：监控核心逻辑
- `check_all_positions()`: 检查所有监控的地址
- `check_address_positions()`: 检查单个地址的所有 Position
- `check_position_health()`: 检查单个 Position 的健康状态
- `send_alert()`: 发送 Telegram 提醒

**database.py**
- `add_monitored_address()`: 添加监控地址
- `remove_monitored_address()`: 删除监控地址
- `get_monitored_addresses()`: 获取用户的监控列表
- `get_all_monitored_addresses()`: 获取所有监控地址
- `add_alert()`: 记录提醒历史
- `add_position_snapshot()`: 记录 Position 快照

**bot.py**
- `/monitor` 命令处理
- `/unmonitor` 命令处理
- `/mymonitors` 命令处理
- 启动监控任务

---

## 部署说明

### 新增依赖

无需额外依赖，所有功能使用现有的包。

### 环境变量

```bash
BOT_TOKEN=你的Telegram_Bot_Token
```

### 启动 Bot

```bash
python3 bot.py
```

Bot 启动后会自动：
1. 初始化数据库
2. 启动监控任务（后台运行）
3. 开始接收 Telegram 消息

### 日志输出

```
2026-01-01 12:00:00 - root - INFO - Starting Fluid Position Monitor Bot...
2026-01-01 12:00:01 - root - INFO - Bot started and waiting for messages...
2026-01-01 12:00:01 - root - INFO - Position monitor will start shortly...
2026-01-01 12:00:10 - monitor - INFO - Starting position monitor (check interval: 1800s)
2026-01-01 12:00:10 - monitor - INFO - Starting position check cycle...
2026-01-01 12:00:10 - monitor - INFO - Checking 2 monitored address(es)
```

---

## 常见问题

### Q: 监控多少个地址有限制吗？

**A:** 没有硬性限制，但建议每个用户监控不超过 10 个地址，以确保性能。

### Q: 可以监控其他人的地址吗？

**A:** 可以！你可以监控任何地址，不需要是你自己的。

### Q: 提醒会不会太频繁？

**A:** 不会。每个 Position 每小时最多提醒一次，即使 HF 持续低于阈值。

### Q: 如果 Bot 重启，监控会丢失吗？

**A:** 不会。所有监控配置都存储在数据库中，Bot 重启后会自动恢复。

### Q: 可以修改检查频率吗？

**A:** 可以。在 `bot.py` 中修改 `check_interval` 参数（单位：秒）：
```python
monitor = PositionMonitor(bot, db, check_interval=1800)  # 30 分钟
```

### Q: 监控会消耗查询次数吗？

**A:** 不会！监控是后台自动运行的，不计入每日 10 次查询限制。

### Q: 如何查看历史提醒？

**A:** 目前暂不支持查看历史提醒，但所有提醒都记录在数据库的 `alert_history` 表中。

---

## 使用建议

### 推荐阈值设置

| 风险偏好 | Alert 阈值 | Critical 阈值 |
|---------|-----------|--------------|
| 保守型 | 1.25 | 1.15 |
| 平衡型 | 1.15 | 1.05 |
| 激进型 | 1.10 | 1.03 |

### 最佳实践

1. **监控重要地址**
   - 监控你的主要借贷地址
   - 监控高价值 Position

2. **设置合理阈值**
   - 根据市场波动性调整
   - 保守设置更安全

3. **及时响应提醒**
   - 收到 WARNING 时关注
   - 收到 CRITICAL 时立即行动

4. **定期检查**
   - 使用 `/mymonitors` 查看监控列表
   - 删除不需要的监控

---

## 更新日志

### v2.0 - 2026-01-01

**新增功能:**
- ✅ 自动监控系统
- ✅ 可自定义阈值
- ✅ Telegram 实时提醒
- ✅ 防骚扰机制
- ✅ 历史记录追踪

**新增命令:**
- `/monitor` - 开始监控地址
- `/unmonitor` - 停止监控地址
- `/mymonitors` - 查看监控列表

**技术改进:**
- 数据库持久化存储
- 后台异步监控任务
- 多链并行查询
- 智能提醒去重

---

## 支持与反馈

如有问题或建议，请联系开发者。

**祝你使用愉快！** 🎉
