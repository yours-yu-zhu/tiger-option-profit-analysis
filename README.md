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
