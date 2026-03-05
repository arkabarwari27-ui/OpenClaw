已经上传到 https://clawhub.ai/huangchen2005/btceth-dulwin-engine


# 双币赢分析 Skill 开发总结



## 日期
2026-03-05

## 项目概述
创建了一个专业的BTC/ETH期权双币赢策略分析工具，用户可通过AI助手获取交易建议。

## 架构

```
用户 → AI (Skill) → API服务 → 数据采集 → Deribit/Binance
```

### 核心文件
- **服务端API**: `/btceth-dualwin-engine/scripts/api.py`
- **数据脚本**: `/btceth-dualwin-engine/scripts/fetch_data.py`
- **Skill定义**: `/skills/btceth-dualwin-engine/SKILL.md`
- **Skill执行**: `/skills/btceth-dualwin-engine/run.py`

## 解决的问题

### 1. Skill调用方式
- 最初尝试用Python脚本调用 → 改为直接用web_fetch访问API
- 使用时间戳参数避免OpenClaw缓存

### 2. API缓存问题
- web_fetch有严重缓存
- 解决方案：URL加时间戳参数 + API返回no-cache头

### 3. 胜率计算错误
- **问题**: 固定查表导致多个策略胜率相同
- **修复**: 改用正态分布公式 P = N(k) × 100

### 4. 胜率修正逻辑
- **原逻辑**: RSI>70时胜率×0.9，导致最低值被拉平
- **修复**: 改为减法：RSI>70 减6%，同时调整下限

### 5. 趋势过滤规则
- **原逻辑**: RSI>65禁止卖CALL
- **新逻辑**: 
  - RSI>70 只允许卖PUT（禁止卖CALL）
  - RSI<30 只允许卖CALL（禁止卖PUT）

## 待完成

### 1. 付费功能
- 已实现skillpay.me接口，但暂时关闭
- 需要：客户端配置User ID、支付验证

### 2. Skill分发
- 需要确认OpenClaw的skills目录位置
- 需要发布到ClawHub

## 核心参数

| 参数 | 说明 |
|------|------|
| DVOL | Deribit波动率指数 |
| IVP | IV Percentile，历史分位 |
| RSI | 4小时RSI(14) |
| Skew | Delta 25 Skew |
| σ | 波动率标准差 |

## 胜率公式

```
基础胜率 = N(k) × 100
k = 0.75, 1.0, 1.25, 1.5

最终胜率 = 基础胜率 - 时间惩罚 - RSI惩罚
- 时间惩罚: 激进3%/天，稳健2%/天
- RSI惩罚: RSI>70 或 RSI<30 减6%
```

## 趋势过滤规则

| RSI区间 | 允许方向 |
|---------|----------|
| >70 | 只允许卖PUT |
| <30 | 只允许卖CALL |
| 30-70 | 正常判断 |



## 后续计划
1. 完善付费功能
2. 测试Skill分发



