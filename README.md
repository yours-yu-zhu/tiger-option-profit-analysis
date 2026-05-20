# Tiger Option Profit Analysis

Codex plugin for analyzing TigerOpen option strategy profits and updating the Notion page `期权收益统计`.

## What It Does

This plugin contributes the `tiger-option-profit-analysis` skill. It helps Codex:

- Pull read-only TigerOpen account, position, option position, and order-history data.
- Classify short puts, assignments, covered calls, Wheel flows, and put rollover chains.
- Preserve user-specific Tencent/TCH.HK accounting rules and dividend adjustments.
- Convert HKD/USD summaries without mixing native currencies directly.
- Refresh the Notion page named `期权收益统计` with a structured report.

## Plugin Layout

```text
.codex-plugin/plugin.json
skills/tiger-option-profit-analysis/SKILL.md
```

## Safety

The skill is read-only for TigerOpen. It must never place, modify, or cancel orders.

---

# Tiger Option Profit Analysis 中文说明

这是一个 Codex 插件，用于分析 TigerOpen 账户中的期权策略收益，并更新 Notion 页面 `期权收益统计`。

## 功能

本插件提供 `tiger-option-profit-analysis` skill，帮助 Codex：

- 只读获取 TigerOpen 账户资产、股票持仓、期权持仓和订单历史。
- 识别并分类卖 Put、行权接股、Covered Call、Wheel 流程和 Put 展期链。
- 保留用户特定的腾讯/TCH.HK 统计规则和股息调整。
- 在汇总 HKD/USD 收益时先做汇率换算，避免直接混加不同币种。
- 将结构化收益报告刷新到 Notion 页面 `期权收益统计`。

## 插件结构

```text
.codex-plugin/plugin.json
skills/tiger-option-profit-analysis/SKILL.md
```

## 安全规则

该 skill 对 TigerOpen 只执行只读查询。使用本插件时，绝不能下单、改单、撤单或执行任何交易操作。
