---
name: tiger-option-profit-analysis
description: Analyze a TigerOpen brokerage account's option strategy profits using the user's Wheel, short put, covered call, assignment, and put-rollover rules, then update the Notion page named 期权收益统计. Use when the user asks to update option profit stats, analyze TigerOpen option trades, Wheel收益, 卖Put收益, covered call收益, 展期链, 期权收益统计, or write the results to Notion.
metadata:
  short-description: TigerOpen option/Wheel profit analysis for Notion
---

# Tiger Option Profit Analysis

Use this skill when the user asks to analyze TigerOpen option profits or update the Notion page `期权收益统计`.

## Required Sources

Use the `tigeropen` skill first when available.

Query these read-only TigerOpen data sets:

- Account assets, including `currency_assets` forex rates.
- Account total net liquidation value for full-account contribution calculations.
- Current stock positions.
- Current option positions.
- Trade order history visible through the API.

Also search Notion for the page titled `期权收益统计`; update that page unless the user specifies another target.

Record these data cutoffs in the output:

- Account/position snapshot time.
- Latest order time and earliest order time returned by the API.
- Forex rate used for HKD to USD conversion.
- Account net liquidation value converted to HKD.
- Visible strategy day count used for the full-account annualized contribution.

## Core Accounting Rules

### Annualized Return Basis

When calculating annualized returns, assume the user does not use leverage. Use full cash-secured capital, not broker margin, buying power, or reduced margin requirement.

- Short put capital = strike * contract multiplier.
- Assigned Wheel capital = the full cash needed to take assignment at the put strike, carried through the Wheel until shares are called away or sold.
- Covered call on assigned shares does not add new capital; it uses the assigned stock capital already counted in the Wheel.
- Annualized return = profit / full capital / holding days * 365.

### Final Reporting Currency

Use HKD as the final reporting currency for all top-level totals, dashboards, symbol summaries, and Notion conclusions.

- Keep native currency columns in detail tables.
- Convert every non-HKD amount to HKD before summing.
- If the account asset query is based in USD and `currency_assets["HKD"].forex_rate` means `1 HKD = X USD`, convert USD to HKD with `USD / X`.
- Do not present USD as the final total currency. USD equivalents may be shown only as supporting detail when useful.

### Full Account Capital Contribution

Calculate how much the option strategy contributes to the whole brokerage account's capital return.

- Account capital denominator = current account `net_liquidation` converted to HKD.
- Strategy profit numerator = final option strategy total profit in HKD, including realized items and current estimates for incomplete flows.
- Visible strategy days = days from the earliest visible filled option trade used in the analysis to the account/position snapshot time. If the user provides a different start date, use the user's date and state it.
- Simple full-account contribution = strategy profit HKD / account net liquidation HKD.
- Annualized full-account contribution = simple full-account contribution / visible strategy days * 365.
- Label this metric as the option strategy's contribution to the whole account, not the account's total return.
- If API history is incomplete, mark the contribution as based on the visible API range.

### Short Put Not Assigned

If the user sold a put and it expires worthless, count the option premium as realized profit.

If the user buys back the put and does not open a related replacement put, count the close result as realized PnL.

### Put Assignment Enters Wheel

If a short put is assigned, treat the full cycle as one Wheel flow.

Wheel PnL = short put premium + stock PnL after assignment + covered call premium/PnL.

Include cash dividends received or receivable while holding assigned shares as part of the stock-holding component of the Wheel.

The Wheel is complete only when the assigned shares are called away or otherwise sold as part of the same cycle.

For incomplete Wheel flows:

Current estimate = realized put PnL + current stock unrealized PnL + covered call unrealized PnL + dividends attributable to the assigned shares.

Also report the strategy-adjusted holding cost per share for every incomplete Wheel flow that already has assigned shares. Prefer a concise column named `策略成本/股` in the `未完成：Wheel 流程` table.

Strategy-adjusted holding cost per share =

`(stock cost basis - realized put PnL - realized covered call PnL - current unrealized covered call PnL - dividends attributable to the assigned shares) / assigned share quantity`

Use the current TigerOpen stock cost basis (`average_cost * share quantity`) when available, because it includes assignment-related costs reflected by the broker. If the stock position cost basis is unavailable, use the assignment strike times assigned shares and state that the value excludes broker cost-basis adjustments. Do not subtract current stock unrealized PnL from the holding cost; stock unrealized PnL belongs in the current estimate, not in the cost-basis adjustment. If useful, add a note with the realized-only cost per share that excludes open covered call unrealized PnL.

Important mapping example: Tiger/Tencent option contracts may appear as `TCH.HK`, while the assigned stock position may appear as `00700`. Treat `TCH.HK` options and `00700` stock as the same Tencent Wheel flow when the order/position sequence matches.

Known user-specific dividend adjustment to preserve unless newer data contradicts it:

- Tencent/`00700` assigned shares include a `500 HKD` dividend in the Wheel stock-holding result.

### Put Rollovers

If a put is bought back and the user opens a same-underlying replacement put with a later expiry and/or different strike, treat the pair as a rollover chain.

Do not classify the buyback loss as a standalone completed loss.

Rollover chain estimate = realized close PnL from the old put + current unrealized PnL of the new put.

Known user-specific rollover patterns to preserve unless newer data contradicts them:

- `TCH.HK 260429 P500` buyback rolls into `TCH.HK 260528 P500`.
- `TCH.HK 260528 P450` buyback rolls into `TCH.HK 260629 P450`.

### Covered Calls

If a covered call is written against stock acquired via put assignment, include that call in the corresponding Wheel.

Do not count that call as an isolated single-leg option.

Example: `TCH.HK` covered calls should be grouped with the `00700` stock Wheel when the shares came from Tencent put assignment.

### Unmatched Single Legs

If an open option cannot be matched to a Wheel or rollover chain, classify it as `未完成：其他单腿期权` and use current unrealized PnL.

### Existing Notion Strategy Precedence

Treat the existing Notion page as the source of truth for strategy grouping when updating an already-maintained report.

- Always fetch the current Notion page before analyzing TigerOpen orders.
- Parse the existing report's strategy rows, strategy names, Wheel flows, rollover chains, completed items, and unmatched single-leg classifications.
- If a strategy or grouping already exists in Notion, keep that grouping and update only its dynamic numbers: realized PnL, unrealized PnL, HKD conversion, subtotals, totals, account contribution, snapshot times, and observations.
- Do not re-compose, split, merge, or reclassify existing Notion strategies solely from the latest order history.
- Use TigerOpen order history to refresh numbers for existing strategies and to identify genuinely new strategies not yet recorded on the Notion page.
- Add a new strategy row only when the visible API data contains a filled option position/order that cannot be matched to any existing Notion strategy, contract, underlying, Wheel, rollover chain, or single-leg row.
- If current TigerOpen data appears to contradict an existing Notion grouping, keep the Notion grouping and mark the row as needing review instead of silently changing the grouping.
- Preserve user-written strategy notes, special mappings, dividends, and manual adjustments from the existing Notion page unless newer user instructions explicitly override them.

## Required Output Structure

Write the Notion page with these sections:

1. `一句话总结`
2. `收益看板`
3. `可视化结构` with a Mermaid flowchart
4. `统计规则`
5. `已完成：Put 未行权/到期`
6. `已完成：完整 Wheel`
7. `未完成：Wheel 流程`
8. `未完成：Put 展期链`
9. `未完成：其他单腿期权`
10. `按标的汇总`
11. `关键观察`

Use concise tables. Keep native currency columns and add an HKD-converted summary. Do not sum HKD and USD directly without converting. The final dashboard, final strategy total, symbol summary, and conclusion must use HKD as the base currency.

In `未完成：Wheel 流程`, include the strategy-adjusted holding cost per share for each assigned-stock Wheel, using the stock's native currency. Use `N/A` only when the Wheel has no assigned stock yet or when the assigned share quantity cannot be determined.

Add a `全账户本金贡献` section before the final observations. It must include:

- Option strategy profit in HKD.
- Current account net liquidation in HKD.
- Visible strategy day count.
- Strategy profit / account net liquidation.
- Annualized full-account contribution.

## Notion Update Rules

- Search Notion for `期权收益统计`.
- Fetch the page before updating and use its existing strategy structure as the baseline.
- If it is the intended page, update the full refreshed report while preserving existing strategy groupings and user-written notes.
- After updating, fetch it again to verify the content saved.

## Safety

Use only read-only TigerOpen calls for this analysis.

Never place, modify, or cancel orders while using this skill.

If data is incomplete, say exactly what API range or position data was visible and mark affected rows as estimates.
