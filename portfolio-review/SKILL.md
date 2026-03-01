---
name: portfolio-review
description: |
  Parse Indian HNI/family portfolio statements into a consolidated Excel workbook with distribution tracking, XIRR/CAGR, and quarterly performance.
  Inputs: CAMS, KFintech, Zerodha, HDFC Securities, IIFL Securities, PMS, AIF, Schwab CSV, ICICI Direct, Groww/Kuvera, NSDL/CDSL CAS.
  Family portfolios (self, spouse, HUF, parents, trusts). Google Drive + Gmail integration for distribution advices.
  XIRR/CAGR: portfolio-level and per-holding XIRR with intermediate cashflows and distributions. Quarterly rolling XIRR from locked baseline. CAGR when no intermediate flows.
  Distribution + P&L tracking: dividends, email advices, HDFC P&L summaries, AIF income, realized gains.
  Triggers: "portfolio", "holdings", "consolidate", "distributions", "XIRR", "CAGR", "quarterly return", "performance", or upload of financial PDFs/Excel/CSV.
  Multi-currency (USD→INR). Cross-platform overlap detection.
---

# Family Portfolio Review Skill (v7)

Parses investment statements for one or more family members and generates a professional
consolidated Excel workbook with family-level and member-level views, **including distribution
income tracking, realized P&L, and true XIRR with all cashflows**.

## Family Structure

Multiple **family members** ("heads"), each with any combination of platforms:

- **Self** — Primary investor
- **HUF** — Hindu Undivided Family — separate tax entity
- **Spouse**, **Parent**, **Child / Minor**, **Trust / LLP**

## Supported Input Formats (per member)

| Source | File Type | What to Extract |
|--------|-----------|----------------|
| CAMS Consolidated Statement | PDF | MF schemes, cost, market value, units, NAV, folio numbers |
| KFintech Statement | PDF | Same as CAMS for KFintech-registered AMCs |
| Zerodha Holdings | XLSX or CSV | Demat holdings — equities, ETFs, G-Sec, SGBs, REITs, InvITs |
| Zerodha Dividend Files | XLSX | Dividend/distribution history by FY for REIT/InvIT/equity |
| **HDFC Securities Equity Summary** | XLS | **Current holdings with cost, MV, realized P&L, unrealized P&L** |
| **HDFC Securities P&L Summary** | XLS | **Calendar-year realized P&L + dividend/interest income per stock** |
| **IIFL Securities Portfolio** | XLSX | **Current holdings with cost, MV, dividends, XIRR per stock** |
| **IIFL Securities Dividends** | XLSX | **Dividend payout history with component breakdown** |
| **Demat Statement (any broker)** | XLS | **Transaction history — credits/debits, qty changes, NO cost basis** |
| PMS Statements (any provider) | PDF | Stock-level holdings, cost, market value, cash, performance |
| AIF Statements (any provider) | PDF | Capital committed/called, NAV, market value, distributions, drawdowns |
| Charles Schwab Positions | CSV | US ETFs, stocks — prices in USD, convert to INR |
| ICICI Direct Portfolio | PDF | Bonds, debentures, SGBs, tax-free bonds, debt ETFs, REITs, InvITs |
| Groww / Kuvera / MFCentral | PDF/CSV | MF holdings similar to CAMS |
| NSDL/CDSL CAS | PDF | Demat + MF consolidated |
| **Gmail Distribution Advices** | Email | REIT/InvIT quarterly distribution advices from KFintech/LinkIntime |

## Workflow

### Step 0: Locate Files (Uploads OR Google Drive OR Gmail)

**Option A — Direct uploads:** Read `/mnt/user-data/uploads/`.

**Option B — Google Drive folder:** Use GDrive MCP tools to list/fetch files.

**Option C — Gmail for distribution advices:** Search email for REIT/InvIT/AIF distribution advices.
See `references/parsing-guide.md` Section 9.

### Step 1: Identify Family Members & Files
Determine **family member**, **platform source**, **format type** per file.
Read `references/parsing-guide.md` for format-specific extraction rules.

**File identification heuristics:**
- `holdings.csv` with columns Instrument/Qty/Avg.cost/LTP → Zerodha
- `Equity_Summary_Details*.xls` → HDFC Securities current holdings
- `*Profit_And_Loss_Summary*.xls` → HDFC Securities calendar-year P&L
- `CAS_*.pdf` with CAMS/KFintech headers → CAMS Consolidated Account Summary
- `Report.xlsx` with Scrip Name/Qty/Purchase Price columns → IIFL Securities
- `*dividend*.xlsx` with Scrip Name/Date/Amount columns → IIFL Securities dividends
- `Demat_Statement.xls` with Client Id/Date/Name/Quantity → Broker demat transaction history (NO cost basis)
- Account numbers in filenames (e.g., `1518693`) → map to HDFC Securities client ID

### Step 2: Parse Each Statement
Tag every holding with: `member_name`, `platform`, `asset_category`.
Verify parsed totals against statement totals.

### Step 3: Classify Holdings

**Mutual Funds:** Arbitrage, Equity-Index, Equity-FlexiCap, Equity-Value/Contra, Equity-Sectoral,
Hybrid-BAF/MultiAsset, Hybrid-LongShort, Debt-BankingPSU, Debt-Gilt, Debt-TargetMaturity, Liquid, International

**Demat:** Direct Equity, ETFs-Debt, ETFs-Gold/Commodity, ETFs-Equity, G-Sec, SGB, REITs, InvITs, Tax-Free Bonds, NCDs

**AIF:** AIF-Credit/Debt, AIF-Equity, AIF-RealEstate, AIF-InfraDebt

**US Brokerage:** US Equity ETFs, US International/China ETFs, US Fixed Income ETFs, US Crypto ETFs, US Cash

**PMS:** PMS Equity (by provider name), PMS Cash

### Step 4: Collect Distribution Income AND Realized P&L

**Critical step.** Collect from ALL sources:

| Source | How to Get | Data Available |
|--------|-----------|----------------|
| Zerodha REIT/InvIT dividends | Uploaded dividend XLSX files | Symbol, ISIN, ex-date, qty, DPS, net amount |
| HDFC Securities dividends/interest | **HDFC P&L Summary XLS** (DIVIDEND/INTEREST column) | Per-stock annual dividend/interest by calendar year |
| HDFC Securities realized gains | **HDFC P&L Summary XLS** (PROFIT/LOSS column) | Per-stock buy/sell/qty/realized P&L by calendar year |
| ICICI Direct bond/SGB coupons | ICICI Direct PDF (income section) | Security, date, amount |
| REIT/InvIT distributions (any demat) | **Gmail search** for KFintech/LinkIntime emails | Quarter, DPU, component breakdown, gross, TDS, net |
| AIF distributions | AIF statement PDF (transaction table) | Date, type, amount, TDS |

**HDFC P&L Summary files are especially valuable** — they provide BOTH realized P&L AND dividend/interest income per stock per calendar year. Multiple years of P&L files build a complete trading + income history.

See `references/parsing-guide.md` Sections 8, 9, and 11 for parsing details.

### Step 5: Calculate XIRR / CAGR

See `references/parsing-guide.md` Section 15 for full computation methodology.

**Portfolio-Level XIRR (Quarterly Tracking)**

The family runs quarterly XIRR tracking from a locked baseline date.
Each quarter, compute rolling XIRR from baseline using all intermediate cashflows.

- **Baseline**: locked start date + starting portfolio value (negative cashflow = investment)
- **End**: current quarter-end date + current portfolio value (positive cashflow = redemption)
- **Intermediate cashflows**: any net new money IN (negative) or withdrawals OUT (positive) between quarters
- If no intermediate cashflows, XIRR = CAGR (they are identical for 2-point calculations)

**Per-Holding XIRR (with distributions)**

Calculate three scenarios per holding or member:
1. **MV-only XIRR** — just cost→MV, no distributions
2. **+ Distributions** — add that member's dividends/interest as positive cashflows on their dates
3. **+ Realized P&L** — include realized gains/losses for sold positions

Use **gross** amounts (pre-TDS) for economic return.

**XIRR vs CAGR**
- CAGR = (End/Start)^(1/years) - 1. Only valid with no intermediate cashflows.
- XIRR = IRR with irregular cashflows. Required when money moves in/out at different dates.
- For a closed portfolio (no additions/withdrawals), XIRR = CAGR.
- Always compute XIRR; report CAGR only when identical (no intermediate flows).

### Step 6: Build Excel Workbook
Read `references/excel-template.md` for formatting rules.

**Required Sheets:**
1. **Family Summary** — Grand total, member-wise, asset allocation, XIRR & distributions, observations
2. **[Member] — [Platform]** — Detail sheets per member per platform
3. **Distributions** — Comprehensive income sheet with per-member sections
4. **Action Items** — Family-wide prioritized recommendations

**For members with HDFC P&L data:** include a "Realized P&L & Distribution Income" subsection on their HDFC sheet showing year-by-year trading activity + dividends per security.

### Step 7: Observations & Action Items

**FAMILY level:**
- Net worth, member %, total distributions, annualized yield, XIRR with/without distributions
- Cross-member overlap (same instrument held by multiple members — e.g., IndiGrid across platforms)
- Cross-member strategy overlap, allocation gaps
- **Active trading assessment** — are members trading REIT/InvIT for both income AND capital gains?

**MEMBER level:**
- Platform concentration, strategy overlap
- **DPU trends**, **quantity changes** affecting future income
- **Realized P&L analysis** — profitable trades vs loss-making (e.g., Capital Infra losses vs distributions)
- **Total economic return** = unrealized gain + realized P&L + distributions
- Tax harvesting, return of capital tracking

### Step 8: Save and Present
Save to `/mnt/user-data/outputs/` → `[Surname]_Family_Portfolio_Review_[Date].xlsx`

## Important Notes

- All amounts in INR (₹). Indian notation in observations (Cr/L), raw numbers in Excel
- **USD conversion**: Search for live USD/INR rate before processing
- **HUF is separate tax entity**: Own member, never merged with individual
- **Distribution tracking is critical**: MV-only return understates true economic return for yield instruments
- **HDFC Securities P&L files**: One per calendar year. Contains realized P&L + dividend/interest per stock. Multiple years build complete history. These are XLS (old Excel format) — use `xlrd` or `pd.read_excel`.
- **Zerodha holdings.csv**: Simple CSV with Instrument/Qty/Avg.cost/LTP/Invested/Cur.val/P&L columns. May contain just 1 holding (e.g., LIQUIDCASE as cash equivalent).
- **Active REIT/InvIT traders**: Some family members actively buy/sell REITs/InvITs for both distributions AND capital gains. Track BOTH income streams. Flag when realized losses exceed distributions (net negative outcome).
- **Gmail for distributions**: REIT/InvIT distribution advices from KFintech/LinkIntime. Quarterly.
- **IIFL Securities**: Report.xlsx provides holdings with XIRR per stock. Dividend file has per-payout detail but components are unlabeled — infer from amount patterns.
- **Demat statements**: Only show qty transactions (credits/debits), NOT cost basis. Flag cost as "estimated" and add action item to get contract notes.
- **Cross-platform same security**: A single person may hold the same ISIN across multiple brokers (e.g., Brookfield REIT in HDFC + IIFL). Track separately, report combined totals.
- Use openpyxl. Always run recalc script. Navy/gold theme.
- Cross-member holdings: flag for awareness, not consolidation.
