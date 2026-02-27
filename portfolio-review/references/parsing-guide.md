# Statement Parsing Guide

## 1. CAMS Consolidated Account Statement (PDF)

### Structure
- Page 1-2: Scheme-wise holdings table
- Header: Client name, email, statement date
- Registrar split: CAMS schemes listed first, then KFINTECH schemes

### Extraction Strategy
Read visually from the PDF pages:

**For each scheme row, extract:**
- AMC / Fund House name, Scheme name (with plan type — Direct/Regular, Growth/IDCW)
- Folio number, ISIN, Registrar (CAMS or KFINTECH)
- Cost value, Market/Current value, Units, NAV and NAV date

**Validation**: Sum all scheme market values — should match "Total" row at bottom.

### Common CAMS Gotchas
- Same scheme may appear with multiple folios (demat vs non-demat)
- "Advisor: DIRECT" means direct plan
- Some schemes show as "** Registrar: KFINTECH"
- Cost value labels vary: "Amount", "Cost", "Investment Value"

---

## 2. Zerodha Holdings Export (XLSX)

### Structure
- Rows 1-12: Header info (blank rows, client ID)
- Row 13-17: Summary (Invested Value, Present Value, P&L)
- Row 22 (approx): Column headers
- Row 23+: Individual holdings

### Column Mapping (0-indexed from column B)
```
Col B (1): Symbol    Col C (2): ISIN      Col D (3): Sector
Col E (4): Qty       Col F (5): Qty Discrepant   Col G (6): Qty Long Term
Col H (7): Pledged   Col I (8): Pledged Loan     Col J (9): Average Price
Col K (10): LTP      Col L (11): Unrealized P&L  Col M (12): P&L %
```

### Zerodha Symbol Classification
| Pattern | Asset Type |
|---------|-----------|
| `*GS20*-GS` | Government Security |
| `BBETF*`, `EBBETF*` | Bharat Bond ETF |
| `*GOLDBEES*`, `*GOLDETF*` | Gold ETF |
| `LIQUIDBEES*`, `LIQUIDCASE` | Liquid ETF |
| `*METALIETF*`, `*SILVETF*` | Commodity ETF |
| `*-RR` | REIT |
| `*-IV` | InvIT |
| `SGB*-GB`, `SGBD*-GB` | Sovereign Gold Bond |
| Everything else | Direct Equity or NCD |

---

## 2b. Zerodha Holdings CSV (holdings.csv)

### Structure
A simpler format than the XLSX — often downloaded from the Zerodha Console/Kite web app.

### Column Mapping
```
Instrument, Qty., Avg. cost, LTP, Invested, Cur. val, P&L, Net chg., Day chg.
```

### Extraction Code
```python
import pandas as pd
df = pd.read_csv('holdings.csv')
for _, row in df.iterrows():
    symbol = row['Instrument']
    qty = row['Qty.']
    avg_cost = row['Avg. cost']
    ltp = row['LTP']
    invested = row['Invested']
    cur_val = row['Cur. val']
    pnl = row['P&L']
```

### Notes
- May contain very few holdings (e.g., just LIQUIDCASE as cash equivalent)
- LIQUIDCASE = Liquid ETF used as cash equivalent (low yield ~5-6%)
- `Net chg.` and `Day chg.` are intraday metrics — ignore for portfolio review
- Apply same symbol classification rules as XLSX format

---

## 3. PMS Statements (PDF — Various Providers)

### Capitalmind Wealth
- Stock-level holdings: INSTRUMENT, QTY, COST, LTP, VALUE, P&L, P&L%
- Values in Lakhs (₹ X.XXL) — multiply by 100,000
- Cash may be "LIQUIDCASE"

### Negen Capital
- Portfolio Appraisal: Security, Quantity, Unit Cost, Cost, Price, Market Value, Gain/Loss
- Performance Appraisal — period returns vs BSE 500 TRI

### General PMS Rules
1. Separate equity holdings from cash component
2. Total = Equity MV + Cash — verify against stated total
3. Note "as on" date (monthly, not daily)

---

## 4. AIF Statements (PDF — Various Providers)

### Key Fields to Extract
- Fund name, Folio, Investor name/PAN (→ member), Tax status
- Capital Commitment, Called/Contributed, Net Contribution
- NAV per unit, Balance Units, Current Market Value, XIRR
- Distributions: Interest, Dividend, Premium, TDS, Net
- Drawdown History: Date, Amount, NAV, Units

### ICICI Prudential AIF Format
- "Summary of Capital Contributions and Distributions" table
- Transaction types: "Drawdown", "Net Contribution", "Interest FY XX-XX", "TDS on Interest", "Additional Return"

### Nippon India AIF Format
- "Summary of Capital Contribution and Distribution" table
- Narration: "Distribution - Nth Payout FY XXXX-XX - Revenue", "TDS" lines

### AIF Cross-Validation
- Units × NAV ≈ Market Value
- Sum of drawdowns ≈ Capital Called
- Sum of Revenue lines ≈ Gross Distribution in summary

---

## 5. Charles Schwab Positions Export (CSV)

### Column Mapping
```
Symbol, Description, Qty, Price, Mkt Val, Cost Basis, Gain $, Gain %, Security Type
```
- Second-to-last row: "Cash & Cash Investments"
- Last row: "Account Total"

### Schwab ETF Classification
| Pattern | Asset Type |
|---------|-----------|
| QQQ, SPY, VOO, VTI, RSP | US Equity — Broad |
| CQQQ, EEM, VWO, KWEB | International / China |
| PPA, XLF, XLE, XLK | US Equity — Sector |
| IBIT, FBTC, GBTC | Crypto / Bitcoin |
| SGOV, SHY, BND, TLT | US Fixed Income |

**CRITICAL**: Search for live USD/INR rate BEFORE processing.

---

## 6. ICICI Direct Consolidated Portfolio (PDF)

### Key Pages
- **Page 3**: Portfolio Summary totals
- **Page 7**: Holdings detail — Security, Category, Qty, Current Value, Cost, Maturity
- **Page 14**: Income Statement (bond interest, SGB coupons, REIT/InvIT distributions)

### Classification
| Statement Category | Review Category |
|-------------------|----------------|
| DEBT ETF | Bharat Bond ETF |
| GOLD BONDS | Sovereign Gold Bond |
| TAX FREE BONDS | Tax-Free Bond |
| INFRASTRUCTURE INVESTMENT TRUST | InvIT |
| REAL ESTATE INVESTMENT TRUST | REIT |

---

## 7. NSDL/CDSL CAS

Similar to CAMS but includes demat holdings. MF + demat in one statement.

---

## 8. Zerodha Dividend / Distribution Files (XLSX)

One per FY. Contains all dividend/distribution payouts.

### Column Mapping
```
Symbol / Stock, ISIN, Ex-Date / Record Date, Quantity, Dividend Per Share, Net Amount
```

### REIT/InvIT ISIN Reference
| ISIN | Instrument |
|------|-----------|
| INE0CCU25019 | Mindspace REIT |
| INE219X23014 | IndiGrid InvIT |
| INE1JAR25012 | KreditBee InvIT |
| INE041025011 | Embassy REIT |
| INE872Y25012 | IRB InvIT |
| INE390V25010 | PowerGrid InvIT |
| INE0FDU25010 | Brookfield REIT |
| INE0KS425012 | Brookfield REIT (alt) |
| INE0Z8Z23013 | Capital Infra (National) Trust |
| INE0NHL23019 | Indus Infra Trust |

### Notes
- One file per FY — need all FYs for complete history
- Quantity changes across rows = user bought/sold between ex-dates
- DPS varies quarter to quarter — track trends
- Net amount = qty × DPS (typically net of TDS on interest component)
- Aggregate by FY for subtotals; use ex-date/payment date for XIRR cashflows

---

## 9. Gmail Distribution Advices (Email)

### When to Use
- **Always for non-Zerodha demat** (ICICI Direct, HDFC Securities, etc.)
- **Optionally for Zerodha** to get component breakdown (interest vs dividend vs return of capital)
- **For AIF distributions** if statements are outdated

### Gmail Search Queries

| Instrument | Search Query |
|-----------|-------------|
| Embassy REIT | `from:embassy.reit@kfintech.com` |
| IRB InvIT | `from:support.irbinvit@kfintech.com` |
| PowerGrid InvIT | `from:powergrid.invit@kfintech.com` |
| Brookfield REIT | `from:brookfieldindia.update@linkintime.co.in` |
| Mindspace REIT | `from:mindspacereit@kfintech.com` |
| IndiGrid InvIT | `from:indigrid@linkintime.co.in` |
| Nippon AIF | `from:Updates.NipponAif@kfintech.com` |
| ICICI Pru AIF | `from:ipruaif@camsonline.com` |

**General**: `subject:"distribution advice" OR subject:"distributions advice"`

### KFintech Email Body Structure
```
DISTRIBUTION DETAILS FOR FY XXXX-XX
3. Beneficiary Name    [NAME]           ← identifies member
7. PAN                 [PAN]            ← identifies member
9. Units held          [Number]
10. Date of Payment    [Date]           ← XIRR cashflow date

Component table:
B. For your holding   Interest  Dividend  Repayment  Total    ← GROSS
D. TDS Amount         XX,XXX    XX,XXX    ...        XX,XXX   ← TOTAL TDS
E. Net Distribution   XX,XXX    XX,XXX    ...        XX,XXX   ← NET
```

### Key Fields
| Field | Use |
|-------|-----|
| Payment Date | XIRR cashflow date |
| Gross Amount (Row B Total) | XIRR cashflow amount |
| TDS (Row D Total) | Tax tracking |
| Return of Capital / Repayment | Reduces cost basis for LTCG |

### Component Types by Instrument
| Instrument | Components |
|-----------|-----------|
| Embassy REIT | Interest (10% TDS), Dividend (exempt), SPV Debt Repayment |
| IRB InvIT | Interest (10% TDS), Return of Capital, Exempt Dividend |
| PowerGrid InvIT | Interest (10% TDS), Taxable Dividend (10% TDS), Exempt, Repayment, Treasury |
| Brookfield REIT | Interest (10% TDS), Dividend (exempt), Other Income |
| IndiGrid InvIT | Interest (10% TDS), Dividend (exempt), Return of Capital |

---

## 10. HDFC Securities Equity Summary (XLS)

### Identifier
Filename: `Equity_Summary_Details*.xls`

### Structure
Old-format XLS (requires `xlrd`). Rows are:
- Rows 0-3: Blank / metadata / blank / header
- Row 3: Column headers
- Rows 4+: Holdings
- Last row: TOTAL row

### Column Mapping (from Row 3)
```
Stock Name | ISIN | Sector Name | Quantity | Average Cost Price | Value At Cost |
Current Market Price | Current Market Price % Change | Valuation at Current Market Price |
Unrealized Profit/Loss | Unrealized Profit/Loss % Change | Realized Profit/Loss |
Nearing Long term Quantity (within 30 days)
```

### Extraction Code
```python
import pandas as pd
df = pd.read_excel('Equity_Summary_Details.xls', header=None)
# Header row is row 3 (index 3)
# Data starts at row 4
# TOTAL row is last non-empty row
for i in range(4, len(df)):
    row = df.iloc[i]
    if str(row[0]) == 'nan' or 'TOTAL' in str(row[3]):
        break  # reached total row
    name = str(row[0])
    isin = str(row[1])
    sector = str(row[2])
    qty = int(row[3])
    avg_cost = float(row[4])
    cost_value = float(row[5])
    cmp = float(row[6])
    mv = float(row[8])
    unrealized_pnl = float(row[9])
    unrealized_pct = float(row[10])
    realized_pnl = float(row[11])
```

### Classification by Sector Name
| HDFC Sector | Review Category |
|-------------|----------------|
| REAL ESTATE INVESTMENT TRUSTS | REIT |
| INFRASTRUCTURE INVESTMENT TRUSTS | InvIT |
| (other sector names) | Direct Equity |

### Key Notes
- **Realized Profit/Loss column** includes ALL historical realized P&L for that stock — very useful
- **Value At Cost** = Qty × Avg Cost Price (for current holdings only)
- TOTAL row has aggregate cost, MV, unrealized/realized P&L
- **Install `xlrd`**: `pip install xlrd --break-system-packages`

---

## 11. HDFC Securities Profit & Loss Summary (XLS) — NEW

### Identifier
Filename: `*Profit_And_Loss_Summary_for_ALL_*YYYY.xls` where YYYY = calendar year

### Why This File is Valuable
This is the **only file that provides per-stock, per-year dividend/interest AND realized P&L** in a single table. Multiple years of these files build a complete trading + income history for HDFC Securities holdings.

### Structure
Old-format XLS (requires `xlrd`). Rows:
- Rows 0-3: Blank / metadata ("Equity as on: YYYY/MM/DD HH:MM:SS") / blank / header
- Row 4 (index): Column headers
- Rows 5+: Per-stock data
- Last row: "EQUITY TOTAL" with aggregates

### Column Mapping (from Row 4)
```
STOCK SYMBOL | ISIN | QUANTITY | BUY VALUE | SELL VALUE | PROFIT/LOSS | DIVIDEND/INTEREST | TOTAL INCOME
```

### Extraction Code
```python
import pandas as pd
df = pd.read_excel(fname, header=None)
# Find header row
for i in range(len(df)):
    if str(df.iloc[i][0]).strip() == 'STOCK SYMBOL':
        hdr = i; break
# Parse data rows
for i in range(hdr+1, len(df)):
    row = df.iloc[i]
    sym = str(row[0])
    if 'TOTAL' in sym: break  # total row
    isin = str(row[1])
    qty = float(row[2]) if pd.notna(row[2]) else 0
    buy = float(row[3]) if pd.notna(row[3]) else 0
    sell = float(row[4]) if pd.notna(row[4]) else 0
    pnl = float(row[5]) if pd.notna(row[5]) else 0
    dividend = float(row[6]) if pd.notna(row[6]) else 0
    total = float(row[7]) if pd.notna(row[7]) else 0
```

### Interpretation
| Column | Meaning |
|--------|---------|
| QUANTITY | Units traded (bought+sold) in that calendar year. 0 = no trades, income only |
| BUY VALUE | Total purchase cost for units bought in that year |
| SELL VALUE | Total sale proceeds for units sold in that year |
| PROFIT/LOSS | Realized capital gain/loss = Sell Value - Buy Value |
| DIVIDEND/INTEREST | Distributions/dividends/interest received in that year |
| TOTAL INCOME | PROFIT/LOSS + DIVIDEND/INTEREST |

### Key Analysis Patterns

**Pure income (no trading):** QTY=0, BUY=0, SELL=0, P&L=0, DIVIDEND>0
→ Holding and collecting distributions. Most common for REIT/InvIT holders.

**Active trader:** QTY>0, BUY>0, SELL>0, P&L≠0, DIVIDEND>0
→ Both trading and collecting distributions. Calculate total economic return.

**Loss-making trade:** P&L<0 but DIVIDEND>0
→ Compare: if |P&L| > DIVIDEND, net negative. Flag as lesson learned.

**Multiple years of files:**
Combine all years to build a complete per-stock history:
```python
all_years = {}
for year, fname in year_files.items():
    # parse each file
    for stock in stocks:
        all_years.setdefault(stock, []).append({
            'year': year, 'pnl': pnl, 'div': div, 'total': total
        })
# Now can show per-stock multi-year income history
```

### Cross-Validation
- Sum of all stocks' DIVIDEND/INTEREST should match file's EQUITY TOTAL dividend column
- Sum of PROFIT/LOSS should match EQUITY TOTAL P&L column
- Compare current holdings (from Equity Summary) against trade history — stocks with qty=0 in latest P&L but present in Equity Summary were acquired in the current year

---

## 12. Google Drive Integration

When the user points to a Google Drive folder:
1. Use gdrive_search/gdrive_list to list folder contents
2. Filter for .pdf, .xlsx, .xls, .csv files
3. If subfolders exist, use subfolder name as member identifier
4. Download each file, save to `/home/claude/drive_files/[member]/[filename]`

### Member Assignment Priority
1. Subfolder name → 2. Filename → 3. Statement header (name/PAN) → 4. Ask user

---

## Cross-Validation Checklist

- [ ] Each MF total matches CAMS/KFintech statement total
- [ ] Zerodha (XLSX or CSV) total matches file summary
- [ ] HDFC Securities Equity Summary total matches TOTAL row
- [ ] HDFC P&L Summary year totals match EQUITY TOTAL row
- [ ] ICICI Direct sum matches page 3 total
- [ ] Each PMS equity + cash = stated total
- [ ] Each AIF: units × NAV ≈ market value
- [ ] Schwab total matches "Account Total" row
- [ ] USD/INR rate noted and consistent
- [ ] No duplicate holdings within same member
- [ ] Cross-platform overlaps flagged (same ISIN in Zerodha + HDFC)
- [ ] Cross-member holdings noted (same ISIN across family members)
- [ ] Grand family total = sum of all member totals
- [ ] **Distribution totals**: Zerodha files + HDFC P&L dividends + Gmail emails + AIF statements = family total
- [ ] **XIRR calculated in 3 scenarios**: MV-only, + primary member, + all distributions
- [ ] **DPU trends noted** for REIT/InvIT holdings
- [ ] **Realized P&L** aggregated per member where available (HDFC P&L files)
