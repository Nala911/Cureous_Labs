# MODULE 10: RENTAL LOG (BILLING LEDGER)

**Form Name**: `Rental_Log`  
**Form Link Name**: `Rental_Log`  
**Default Report**: `All_Rental_Logs`  
**Purpose**: Primary commercial and operational billing ledger recording patient usage periods across all customer bed stations. Captures physical device pairings (Control Unit and Overlay serials), customer facility/bed space, exact calendar periods, and calculates taxable amounts, 5% GST, and total invoice values. Bridges operational activity to Zoho Books invoices.

---

## 1. Commercial Rental Modes Supported

### 1. Pay-Per-Use (Daily):
- Operator selects `Bed Station`; system auto-fetches linked Agreement, Customer, Model, Item, Daily Rate, and active CU/Overlay serials.
- User enters `Usage_Start_Date` and `Usage_End_Date`.
- System calculates `Number_of_Days = (Usage_End_Date - Usage_Start_Date) + 1` (inclusive).
- Computes:
  - `Taxable_Value = (Number_of_Days * Agreed_Rate) - Discount_Amount`
  - `GST_Amount (5%) = Taxable_Value * 0.05`
  - `Invoice_Value = Taxable_Value + GST_Amount`

### 2. Fixed Monthly Rental:
- Generated on-demand directly from `Rental_Agreement` via Stateless Wizard.
- Attaches the assigned Bed Station and active CU/Overlay serials.
- Computes flat monthly fee:
  - `Taxable_Value = Agreed_Rate - Discount_Amount`
  - `GST_Amount (5%) = Taxable_Value * 0.05`
  - `Invoice_Value = Taxable_Value + GST_Amount`

---

## 2. Field Specifications

### A. System & Relational Identifiers:
1. **Rental Log ID** `[Auto-Number]` (Primary Key, e.g. `RL-2026-0045`) -> `Rental_Log_ID`
2. **Bed Station** `[Lookup -> Bed_Stations]` (Mandatory) -> `Bed_Station`
3. **Customer** `[Lookup -> Customers - Read Only]` -> `Customer`
4. **Rental Agreement** `[Lookup -> Rental_Agreements - Read Only]` -> `Rental_Agreement`
5. **Rental Item** `[Lookup -> Product_Catalog - Read Only]` -> `Rental_Item`
6. **Product Model** `[Single Line - Read Only]` -> `Product_Model`
7. **Rental Mode** `[Single Line - Read Only]` (`"Pay Per Use (Daily)"` | `"Fixed Monthly Rental"`) -> `Rental_Mode`

### B. Hardware Device Snapshot (Bedside Configuration):
8. **Control Unit Serial No** `[Single Line - Read Only]` -> `Control_Unit_Serial_No`
9. **Overlay Serial No** `[Single Line - Read Only]` -> `Overlay_Serial_No`
10. **Control Unit Asset** `[Lookup -> Devices - Read Only]` -> `Control_Unit_Asset`
11. **Overlay Asset** `[Lookup -> Devices - Read Only]` -> `Overlay_Asset`

### C. Billing Period & Duration:
12. **Billing Month** `[Single Line - Read Only]` (e.g. "October 2026") -> `Billing_Month`
13. **Usage Start Date** `[Date]` (Mandatory) -> `Usage_Start_Date`
14. **Usage End Date** `[Date]` (Mandatory) -> `Usage_End_Date`
15. **Number of Days** `[Number - Read Only]` (Formula: `(End - Start) + 1`) -> `Number_of_Days`

### D. Commercial Computations & Taxes:
16. **Agreed Rate** `[Currency]` (From Agreement; editable if authorized) -> `Agreed_Rate`
17. **Discount Amount** `[Currency]` (Default: 0.00) -> `Discount_Amount`
18. **Taxable Value** `[Currency - Read Only]` -> `Taxable_Value`
19. **GST Rate (%)** `[Decimal - Read Only]` (Fixed at 5.00%) -> `GST_Rate`
20. **GST Amount** `[Currency - Read Only]` (Calculated: `Taxable_Value * 0.05`) -> `GST_Amount`
21. **Invoice Value** `[Currency - Read Only]` (`Taxable_Value + GST_Amount`) -> `Invoice_Value`

### E. Invoicing Lifecycle & Zoho Books Sync:
22. **Billing Status** `[Dropdown]` (Mandatory, Default: `"Pending Invoice"`) -> `Billing_Status`  
    *Options*: `"Pending Invoice"`, `"Invoiced"`, `"Cancelled / Void"`
23. **Zoho Books Invoice ID** `[Single Line - Read Only]` -> `Zoho_Books_Invoice_ID`
24. **Zoho Books Invoice No** `[Single Line - Read Only]` -> `Zoho_Books_Invoice_No`
25. **Invoice Date** `[Date - Read Only]` -> `Invoice_Date`
26. **Invoice URL** `[URL - Read Only]` -> `Invoice_URL`
27. **Billing Notes** `[Multi-Line]` -> `Billing_Notes`

---

## 3. UI Actions & Zoho Books Workflows

### Record Action: `[ Push to Zoho Books ]`
- Available when `Billing_Status == "Pending Invoice"`.
- Pushes draft invoice to Zoho Books with single line item detailing Bed Station, serial numbers, and usage dates.
- Updates status to `"Invoiced"` and stores Books invoice ID, number, and direct URL.

### Report Action: `[ Push Consolidated Invoice to Books ]`
- Multi-select action on report. Accounts selects multiple logs for the same customer (e.g. 6 beds at Apollo Hospital).
- Pushes a single consolidated draft invoice with multiple line items.
- Atomically marks all selected logs as `"Invoiced"`.

### Date Overlap Guard (On User Input of Dates & On Validate):
- Queries existing active logs for the same `Bed_Station`. If dates overlap with an existing uncancelled log, displays warning alert to prevent accidental double-billing.
