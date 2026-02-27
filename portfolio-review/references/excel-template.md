# Excel Output Template

## Visual Design

### Color Palette
```python
NAVY = "1B2A4A"        # Headers, totals, main brand
DARK_BLUE = "2C3E6B"   # Tab colors
GOLD = "C5A55A"        # Subtitles, accents
LIGHT_GOLD = "F5EDD6"  # Section backgrounds, metric labels
WHITE = "FFFFFF"        # Cell backgrounds
LIGHT_GRAY = "F2F2F2"  # Alternating row fill
MED_GRAY = "D9D9D9"    # Borders
GREEN = "1A7A3A"       # Positive P&L
RED = "C0392B"         # Negative P&L, HIGH priority
SECTION_BG = "E8EAF0"  # Subtotal rows
ORANGE = "E67E22"      # MEDIUM priority, Zerodha tab
```

### Font Standards
- **Title**: Arial 16pt Bold, NAVY
- **Subtitle**: Arial 12pt Bold, GOLD
- **Section Header**: Arial 12pt Bold, NAVY (with bottom border)
- **Table Header**: Arial 11pt Bold, WHITE on NAVY background
- **Data**: Arial 10pt
- **Subtotal**: Arial 10pt Bold on SECTION_BG
- **Grand Total**: Arial 11pt Bold, WHITE on NAVY
- **Positive values**: GREEN bold
- **Negative values**: RED bold
- **Notes/Source**: Arial 9pt Italic, gray (#666666)

### Number Formatting
- Currency (INR): `#,##0`
- Currency (INR precise): `#,##0.00` (for DPS, avg cost)
- Currency (USD): `$#,##0.00`
- Percentage: `0.00%`
- NAV: `#,##0.0000` (MF) or `#,##0.00` (AIF/REIT)
- Units: `#,##0.000` (MF) or `#,##0` (equity/REIT)

### Layout Rules
- Column A: Always 3px spacer (empty)
- Alternating row colors (LIGHT_GRAY on even rows)
- All cells have thin MED_GRAY borders
- Subtotal rows: medium NAVY top+bottom border
- Grid lines hidden on all sheets
- Landscape orientation, fit to 1 page width

---

## Sheet Specifications

### Sheet 1: Family Summary
**Tab Color**: NAVY

**Content blocks:**

1. **Title Block** (rows 2-4)
   - Row 2: "[SURNAME] FAMILY — CONSOLIDATED PORTFOLIO REVIEW" (merged B:F)
   - Row 3: "As on [date] | [X] Members ([Y] pending) | [Z] Platforms | XIRR [X.X]% (incl. distributions) | USD/INR: [rate]" (GOLD)
   - Row 4: "Members: [list all with pending status]" (italic gray)

2. **Key Family Metrics** (rows 6-14)
   - Labels in column B (LIGHT_GOLD), values in column C

3. **Member-Wise Breakdown** (rows 17-24+)
   - Table: Member Name | Cost | Market Value | Gain | % of Family
   - FAMILY TOTAL row (navy bar) with SUM formulas

4. **Per-Member Platform Breakdown** (rows 27+)
   - For each member with multiple platforms, show platform subtable

5. **Family Asset Allocation** (rows 46+)
   - 20-25 categories, Source column (italic gray)

6. **Key Observations** (rows 70+)
   - 10-15 numbered observations

7. **Distribution Income & True Return** (rows 85+)
   - Key-value pairs with per-member breakdown:
     - Total Distributions Received (gross)
     - Total TDS Deducted
     - Net Distributions Received
     - [Member 1] Distributions
     - [Member 2] Distributions
     - [Member 3] Distributions (if applicable)
     - Annualized Distribution Income
     - Distribution Yield on Cost
     - XIRR — MV-only (no distributions)
     - **XIRR — With All Distributions** (GREEN bold, highlighted)
     - XIRR Uplift from Distributions

### Sheet: MF Holdings (per member)
**Tab Color**: DARK_BLUE

**Columns**: B=Scheme (42w), C=Cost (17w), D=Market Value (18w), E=Gain (14w), F=Gain% (14w), G=Units (13w), H=NAV (12w)

- Category headers (LIGHT_GOLD, merged)
- Fund rows under each category
- Subtotal per category + Grand Total (navy bar)

### Sheet: Zerodha Holdings (per member)
**Tab Color**: ORANGE

**Columns**: B=Symbol (25w), C=Description (20w), D=Qty (14w), E=Avg Price (14w), F=LTP (14w), G=Invested (16w), H=Present (16w), I=Gain% (14w)

- For CSV imports (holdings.csv): same structure, may have fewer holdings
- LIQUIDCASE should be categorized under "Liquid ETF"
- Category sections + Grand Total

### Sheet: HDFC Securities Holdings (per member)
**Tab Color**: Purple (#8E44AD)

**Section 1: Current Holdings**
**Columns**: B=Security (36w), C=Sector (22w), D=Qty (14w), E=Cost Value (18w), F=Market Value (18w), G=Gain (18w), H=Gain% (14w), I=Type (10w)

- One row per holding. Green/red gain coloring.
- TOTAL row (navy bar)

**Section 2: Realized P&L & Distribution Income** (from P&L Summary files)
- Section header with bottom border
- **Columns**: B=Year, C=Security, D=Qty Traded, E=Buy Value, F=Sell Value, G=Realized P&L, H=Div/Interest, I=Total Income
- Group by calendar year with LIGHT_GOLD year headers
- Year subtotal rows (SECTION_BG) with SUM formulas for Div/Interest and Total Income
- Grand total across all years (navy bar)

**Section 3: Notes**
- Per-stock observations: trading patterns, distributions vs realized P&L, selldown impact
- Flag: if realized loss > distributions received for any stock, highlight as net negative

### Sheet: ICICI Direct (per member)
**Tab Color**: Red (#E74C3C)

### Sheet: PMS Holdings (one per PMS)
**Tab Color**: Unique per PMS

### Sheet: AIF Holdings (per member — combine all AIFs)
**Tab Color**: GOLD (#D4AC0D)

### Sheet: Distributions
**Tab Color**: Green (#2ECC71)

**Title**: "FAMILY DISTRIBUTION INCOME — ALL SOURCES"
**Subtitle**: "[Members list] | [FY range] | Total: ₹X.XX Cr (gross)" (GOLD)

**Section 1: Distribution Summary by Source**
- Table: Source | Member | Payouts | Period | Gross (₹) | TDS (₹) | Net (₹) | Ann. Yield | % of Total
- Member subtotal rows (SECTION_BG)
- FAMILY TOTAL row (navy bar)

**Section 2: XIRR Impact Table**
- Scenario rows: MV-only | + [Member] distributions | + All distributions
- BOLD GREEN for final row

**Section 3: Key Metrics**
- Total Gross/TDS/Net, Annualized Income, Yield on Cost, % of Total Return, XIRR Uplift

**Section 4+: Per-Member Detail Sections**

For each member with distribution data, create a separate section:

**Zerodha dividends** (from dividend XLSX files):
- Organized by FY with year headers
- Columns: Symbol | ISIN | Ex-Date | Qty | DPS (₹) | Net Amt (₹)
- FY subtotals + instrument grand total

**HDFC Securities distributions** (from P&L Summary files):
- Organized by calendar year
- Columns: Year | Security | ISIN | Qty Traded | Realized P&L | Div/Interest | Total Income
- Year subtotals + member grand total
- Include realized P&L alongside distributions for full economic picture

**REIT/InvIT from Gmail** (distribution advices):
- Columns: Quarter | Credit Date | DPU | Interest | Dividend | Repayment | Gross | TDS | Net
- Per-instrument sections with component breakdown
- Instrument totals

**AIF distributions** (from AIF statements):
- Show aggregate per AIF with gross/TDS/net

**Final section: Notes & Observations**
- Largest income generators, DPU trends, quantity changes, HUF strategy, cross-member overlap, tax implications

### Sheet: Schwab US Brokerage (per member)
**Tab Color**: Blue (#2980B9)

### Sheet: Action Items (family-wide)
**Tab Color**: RED

**Columns**: B=# (6w), C=Priority (14w), D=Action (44w), E=Amount (18w), F=Rationale (44w)
- Priority badges: HIGH=red, MEDIUM=orange, LOW=green (white text)
- 38px rows, wrap text. Include member tag: "(CHJ)", "(HUF)", "(AJ)", "(Family)"

---

## Python Code Template

```python
import openpyxl
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side

wb = Workbook()
# ... build sheets per specs above ...
wb.save('/home/claude/portfolio_review.xlsx')

# ALWAYS recalculate after saving:
# python3 /mnt/skills/public/xlsx/scripts/recalc.py portfolio_review.xlsx

# Copy to outputs:
# cp /home/claude/portfolio_review.xlsx /mnt/user-data/outputs/
```

## Validation Checklist

- [ ] All formula cells recalculated (0 errors from recalc script)
- [ ] Member totals in Family Summary match individual sheet totals
- [ ] Family total = sum of member totals
- [ ] Asset allocation percentages sum to ~100%
- [ ] AIF: units × NAV ≈ stated market value
- [ ] Distribution totals in Distributions sheet = sum of all individual payouts
- [ ] XIRR values calculated and consistent
- [ ] Member subtotals in Distributions = sum of that member's instruments
- [ ] HDFC P&L year totals match file EQUITY TOTAL rows
- [ ] No #REF!, #VALUE!, or #DIV/0! errors
- [ ] Positive=green, negative=red throughout
- [ ] All INR cells: `#,##0`. All USD cells: `$#,##0.00`
- [ ] Grid lines hidden on all sheets
- [ ] File in /mnt/user-data/outputs/ and presented via present_files
