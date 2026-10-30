# Multifamily Underwriting Skill

A free Claude skill for the back-of-envelope underwriting of small and midsize multifamily deals (2 to 50 units). Give it a listing, rent comps, and today's interest rate, and it rebuilds the income and expenses, sizes the debt, and returns a go or no-go verdict. If the deal fails at the asking price, it solves backward for the highest price where the deal works, so you have a number to bring to the broker.

This is the same first-pass process I used personally on every multifamily I'm weighing (as an individual investor); and some of what we've baked into as quick option in Operator, the flagship product at [Galleon](https://galleon.io). 

It is meant to decide whether a deal deserves more of your time. It does not replace full due diligence.

## What the skill handles

The skill covers the first screen of a deal, before you order inspections or a full rent roll audit:

- Rebuilds income in two scenarios: in-place (current rents and occupancy) and stabilized (comp-median rents at 5% vacancy)
- Rebuilds expenses from fixed assumptions instead of trusting the broker's numbers, since broker NOI is treated as unreliable by default
- Sizes the debt and calculates annual debt service
- Calculates DSCR, cash-on-cash return, stabilized yield, and the spread over the market cap rate
- Gives a GO, CONDITIONAL GO, or NO-GO verdict against fixed thresholds
- When a deal fails, finds the binding constraint, solves for the maximum workable price, and lists what else would need to be true for the deal to work
- Flags qualitative risks such as master-metered utilities, cash-only listings, long days on market, deferred capital expenditures, and broker rents that run above the comps

## Compatibility

| Tool | Status |
|---|---|
| Claude (claude.ai web and desktop apps) | Tested |
| Claude Code | Uses the same skill format; not yet tested by us |
| Other tools that read `SKILL.md` files (GitHub Copilot, Codex, and others) | Not tested |

The skill is a single `SKILL.md` file of instructions with no scripts. In Claude, it relies on code execution to do the math, so make sure that setting is on.

## Installation

Publishing the skill here makes the file available, but each AI tool has its own way of installing and loading skills. Use the steps for the tool you use.

### Claude (claude.ai web or desktop)

1. Download `multifamily-underwriting.skill` from the [latest release](https://github.com/OWNER/REPO/releases/latest).
2. In Claude, go to **Settings**, then **Capabilities**, and turn on **Code execution and file creation**.
3. Under **Skills**, click **Add**, upload the `.skill` file, and toggle it on.

Use the file from the release page. GitHub's green **Code > Download ZIP** button wraps the files in an extra folder, and Claude will not accept that upload.

Once it is on, the skill loads on its own whenever you paste in a deal. Saying "run the numbers on this" also triggers it.

### Claude Code

Copy the `multifamily-underwriting` folder (the one that contains `SKILL.md`) into one of these locations:

- `~/.claude/skills/multifamily-underwriting/` to use it in every project
- `.claude/skills/multifamily-underwriting/` inside a project to use it only there

Restart Claude Code if it is already running.

### Other tools

If your tool supports `SKILL.md` skills, check its documentation for where skill folders go. For example, GitHub Copilot reads project skills from `.github/skills/`, `.claude/skills/`, or `.agents/skills/`. We have not tested the skill outside of Claude.

## Required inputs

The skill asks for three things and estimates everything else.

| Input | What works | Notes |
|---|---|---|
| Listing or offering memorandum | PDF, screenshot, URL, or just the address, unit count, and asking price | If it discloses actual taxes and insurance, the skill uses them. If not, it estimates them conservatively and flags the gap. A rent roll is used for in-place rents if included. |
| Rent comps | Rentometer, Zillow, Apartments.com, CoStar, or broker comps | The comp median sets stabilized rents. Any broker rent claim more than 5% above the median is flagged. |
| Today's interest rate | The rate you are seeing from lenders | The skill will not assume a rate. If you are unsure, the 10-year Treasury plus 150 to 250 bps is a reasonable starting point, but confirm with your lender before you make an offer. |

Optional inputs: a renovation budget, and any changes to the default assumptions below. The skill does not ask for a cap rate. It estimates the market cap rate on its own and states the figure and its basis so you can correct it.

## Outputs

Every run returns the following, in this order:

1. **Deal summary**: address, unit count, asking price, and asset class
2. **In-place model**: income, expenses, NOI, DSCR, and cash-on-cash
3. **Stabilized model**: income, expenses, NOI, DSCR, cash-on-cash, stabilized yield, and spread over the market cap rate
4. **Rate sensitivity**: DSCR and cash-on-cash at your rate plus and minus 50 bps, with a note if that move would change the verdict
5. **Verdict**: GO, CONDITIONAL GO, or NO-GO with a one-paragraph rationale
6. **Workable price** (NO-GO only): the maximum price, the discount to asking, the price per unit, and what else would need to be true
7. **Qualitative flags**: a short bulleted list

## Example

The deal below is fictional and exists only to show the format.

**Prompt**

> Run the numbers on this. 12 units at 100 Sample Street, Exampleton, asking $950,000. Current rents average $950, with 11 of 12 units occupied. The OM lists taxes at $18,000 and insurance at $9,600, and the broker's pro forma assumes $1,200 rents. My Rentometer pull shows a median of $1,100. Lenders are quoting me 6.75%.

**Expected output (condensed)**

The skill estimates a 7.0% market cap rate for this example and states that assumption.

| | In-place | Stabilized |
|---|---|---|
| Gross potential rent | $136,800 | $158,400 |
| Vacancy | ($11,400), 1 unit vacant | ($7,920), 5% |
| Effective gross revenue | $125,400 | $150,480 |
| Repairs (7.5%) | ($9,405) | ($11,880) |
| CapEx reserve (7.5%) | ($9,405) | ($11,880) |
| Management (10%) | ($12,540) | ($15,840) |
| Taxes | ($18,000) | ($18,000) |
| Insurance | ($9,600) | ($9,600) |
| **NOI** | **$66,450** | **$83,280** |
| DSCR | 1.12 | 1.41 |
| Cash-on-cash | 2.9% | 9.6% |
| Stabilized yield | | 8.64% |
| Spread over 7.0% cap | | 164 bps |

Capital stack at asking: $712,500 of debt (75% LTV), $251,750 of equity including $14,250 in closing costs, and $59,073 in annual debt service on a 25-year amortization.

Rate sensitivity: at 6.25%, in-place DSCR is 1.18 and stabilized cash-on-cash is 10.7%. At 7.25%, they are 1.08 and 8.5%. A 50 bps move does not change the verdict, because the spread test fails regardless of rate.

**Verdict: NO-GO at asking.** Stabilized cash-on-cash clears 8%, but in-place DSCR of 1.12 is below 1.25, and the 164 bps spread is below the 200 bps target.

**Workable price:** This deal works at $854,900 or below, a $95,100 discount to asking ($71,242 per unit). In-place DSCR is the binding constraint. Other things that would move the answer:

- Leasing the vacant unit at $950 before closing raises in-place DSCR to about 1.27 at asking, but the spread would still fall short
- Clearing the spread test at asking would take stabilized rents of about $1,135, which is above the comp median
- Taxes may be reassessed at sale, so confirm the post-sale assessment before treating $854,900 as final

**Flags:** The broker's $1,200 pro forma rent is 9% above the comp median.

## Default assumptions and how to change them

These are our buy box numbers. Change anything that does not match how you buy by saying so in plain English at the start of the conversation, for example "use 80% LTV and a 30-year amortization" or "drop management to zero." The skill recalculates around your inputs.

| Assumption | Default |
|---|---|
| Loan-to-value | 75% of purchase price |
| Amortization | 25 years |
| Closing costs | 1.5% of purchase price |
| Vacancy | Actual for in-place; 5% for stabilized |
| Repairs and maintenance | 7.5% of rent |
| CapEx reserve | 7.5% of rent |
| Property management | 10% of rent |
| Renovations | Funded with equity, not debt |

| Gate | Threshold |
|---|---|
| In-place DSCR | 1.25x or higher |
| Stabilized cash-on-cash | 8% or higher |
| Spread of stabilized yield over market cap rate | 200 bps or more |

A CONDITIONAL GO means the stabilized tests pass but in-place DSCR falls between 1.15 and 1.25.

If you self-manage, we would still underwrite with management in, because your time is not free. The one thing we would not relax is letting the skill rebuild expenses from scratch.

## Limitations and what needs human review

- **It is a screen.** The model uses percentage-based expense loads and does not replace a T-12 review, inspections, or a lender's underwriting.
- **Estimated inputs need checking.** When taxes, insurance, or the market cap rate are estimated, the skill says so. Replace those estimates with real figures before you make an offer.
- **Taxes can change at sale.** Many jurisdictions reassess at purchase, so the taxes in the OM may not be the taxes you pay.
- **Comps drive the stabilized case.** Results are only as good as the comps you provide. In tight urban submarkets, use a radius of 0.5 to 1 mile.
- **Returns are simple, first-year metrics.** The skill does not model IRR, appreciation, refinancing, rent growth over time, or taxes on income.
- **Check the math.** AI models can make arithmetic mistakes. Spot-check the debt service and the workable price before relying on them.

This skill is a tool for screening deals and is not investment, legal, or tax advice.

## License

Released under the [MIT License](LICENSE). You may use, modify, and redistribute it, including commercially, as long as you keep the copyright notice.

## Feedback

Send the first deal you run through it to [amanda@galleon.io](mailto:amanda@galleon.io). We would like to hear what it gets right and what it misses.
