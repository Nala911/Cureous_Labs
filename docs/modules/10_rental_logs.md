# MODULE 10: RENTAL LOGS

**Form Name**: `Rental Logs`  
**Form Link Name**: `Rental_Logs`  
**Default Report**: `All_Rental_Logs`  
**Purpose**: Primary commercial and operational billing ledger recording patient usage periods across customer bed stations. Captures physical device pairings (Control Unit and Overlay serials), customer facility/bed space, exact calendar periods, and calculates taxable amounts, 5% GST, and total invoice values. Bridges operational activity to Zoho Books invoices.

---

## 1. Commercial Rental Modes Supported

### 1. Pay-Per-Use (Daily):
- Operator selects `Bed_Station`; system auto-fetches linked Agreement, Customer, Rental Item, Device Model, Daily Rate, and active Control Unit / Overlay device lookups.
- User enters `Usage_Start_Date` and `Usage_End_Date`.
- System calculates `Number_of_Days = (Usage_End_Date - Usage_Start_Date) + 1` (inclusive).
- Computes:
  - `Taxable_Value = Number_of_Days * Agreed_Rate`
  - `GST_Amount (5%) = Taxable_Value * 0.05`
  - `Invoice_Amount = Taxable_Value + GST_Amount`

### 2. Fixed Monthly Rental:
- Generated for bed stations under a monthly recurring agreement.
- Attaches the assigned Bed Station and active Control Unit / Overlay device lookups.
- Computes flat monthly fee:
  - `Taxable_Value = Agreed_Rate`
  - `GST_Amount (5%) = Taxable_Value * 0.05`
  - `Invoice_Amount = Taxable_Value + GST_Amount`

---

## 2. Field Specifications

### A. User Interactive Fields:
1. **Bed Station** `[Lookup -> Bed_Stations, displaying Bed_Station_Title]` (Mandatory)
   - Field Link Name: `Bed_Station`
   - Form Behavior: Enabled on Add. Primary relational anchor that automatically populates contract context and hardware pairings. Locked in Edit mode once billing is confirmed.
2. **Control Unit** `[Lookup -> Devices, displaying Serial_Number]` (Mandatory)
   - Field Link Name: `Control_Unit`
   - Form Behavior: Auto-populated from `Bed_Station.Control_Unit`. Enabled on Add and Edit to accommodate mid-period hardware exchanges.
3. **Overlay** `[Lookup -> Devices, displaying Serial_Number]` (Mandatory)
   - Field Link Name: `Overlay`
   - Form Behavior: Auto-populated from `Bed_Station.Overlay_Asset`. Enabled on Add and Edit to accommodate mid-period hardware exchanges.
4. **Usage Start Date** `[Date]` (Mandatory)
   - Field Link Name: `Usage_Start_Date`
   - Form Behavior: Enabled on Add and Edit. First calendar day of patient usage.
5. **Usage End Date** `[Date]` (Mandatory)
   - Field Link Name: `Usage_End_Date`
   - Form Behavior: Enabled on Add and Edit. Last calendar day of patient usage.
6. **Agreed Rate** `[Currency]` (Mandatory)
   - Field Link Name: `Agreed_Rate`
   - Form Behavior: Auto-populated from `Rental_Agreement.Agreed_Rate`. Enabled on Add and Edit for authorized adjustments.
7. **Number of Days** `[Number]` (Auto-computed, Editable)
   - Field Link Name: `Number_of_Days`
   - Form Behavior: Auto-calculated when dates are entered (`(End - Start) + 1`). Editable if manual operational adjustments are required.

### B. Auto-Derived Context Fields (Disabled / System-Managed):
8. **Rental Log ID** `[Auto-Number - Native Auto-Hidden on Add]` (Primary Key, Prefix: RL-, e.g. `RL-2026-0045`)
   - Field Link Name: `Rental_Log_ID`
   - Form Behavior: Auto-Hidden on Add (PK; Natively locked in Edit).
9. **Customer** `[Lookup -> Customers, displaying Customer_Name]`
   - Field Link Name: `Customer`
   - Form Behavior: Visible & Disabled on Add and Edit. Read-only context snapshot from the Bed Station.
10. **Rental Agreement** `[Lookup -> Rental_Agreements, displaying Agreement_Title]`
    - Field Link Name: `Rental_Agreement`
    - Form Behavior: Visible & Disabled on Add and Edit. Read-only contract terms anchor.
11. **Rental Item** `[Lookup -> Rental_Items, displaying Item_Name]`
    - Field Link Name: `Rental_Item`
    - Form Behavior: Visible & Disabled on Add and Edit. Line item rental item mapping.
12. **Device Model** `[Single Line]`
    - Field Link Name: `Device_Model`
    - Form Behavior: Visible & Disabled on Add and Edit. Hardware family description inherited from `Bed_Station.Rental_Item` (with fallback to `Rental_Items`).
13. **Rental Mode** `[Single Line / Dropdown]`
    - Field Link Name: `Rental_Mode`
    - Form Behavior: Visible & Disabled on Add and Edit. Value: `"Pay Per Use (Daily)"` or `"Fixed Monthly Rental"`.
14. **Taxable Value** `[Currency]`
    - Field Link Name: `Taxable_Value`
    - Form Behavior: Visible & Disabled on Add and Edit. Computed commercial amount before statutory taxes.
15. **GST Rate (%)** `[Percentage]`
    - Field Link Name: `GST_Rate`
    - Form Behavior: Visible & Disabled on Add and Edit. Fixed statutory rate: `5%` (assigned value: `5`).
16. **GST Amount** `[Currency]`
    - Field Link Name: `GST_Amount`
    - Form Behavior: Visible & Disabled on Add and Edit. Calculated statutory 5% GST value (`Taxable_Value * 0.05`).
17. **Invoice Amount** `[Currency]`
    - Field Link Name: `Invoice_Amount`
    - Form Behavior: Visible & Disabled on Add and Edit. Grand total invoice value (`Taxable_Value + GST_Amount`).
18. **Billing Status** `[Dropdown]`
    - Field Link Name: `Billing_Status`
    - Options: `"Pending Invoice"`, `"Invoiced"`, `"Cancelled / Void"` (Default: `"Pending Invoice"`)
    - Form Behavior: Visible & Disabled on Add (system-managed; locked to `"Pending Invoice"` on initial creation).

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/10_rental_logs/1_Rental_Logs_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable interactive user entry fields (`Bed_Station`, `Control_Unit`, `Overlay`, `Usage_Start_Date`, `Usage_End_Date`, `Agreed_Rate`, `Number_of_Days`). Disable context fields (`Customer`, `Rental_Agreement`, `Rental_Item`, `Device_Model`, `Rental_Mode`, `Taxable_Value`, `GST_Rate`, `GST_Amount`, `Invoice_Amount`, `Billing_Status`). Initialize default values: `GST_Rate = 5.00` and `Billing_Status = "Pending Invoice"`.
  - In EDIT Mode: Lock primary relational references (`Bed_Station`, `Customer`, `Rental_Agreement`, `Rental_Item`, `Device_Model`, `Rental_Mode`, `Taxable_Value`, `GST_Rate`, `GST_Amount`, `Invoice_Amount`, `Billing_Status`). If `Billing_Status == "Pending Invoice"`, allow adjustments to dates, rate, duration, and hardware lookups.

### Workflow 2: On User Input of Bed Station
- **File**: `scripts/10_rental_logs/2_1_Rental_Logs_onuserinput_bedstation.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Bed_Station -> On User Input
- **Plain English Logic**:
  - If `Bed_Station` is cleared, reset all dependent fields (`Customer`, `Rental_Agreement`, `Rental_Item`, `Device_Model`, `Rental_Mode`, `Control_Unit`, `Overlay`, `Agreed_Rate`, `Taxable_Value`, `GST_Amount`, `Invoice_Amount`).
  - Query selected `Bed_Stations` record.
  - Set `Customer = bed.Customer`, `Rental_Agreement = bed.Rental_Agreement`, `Control_Unit = bed.Control_Unit`, and `Overlay = bed.Overlay_Asset`.
  - If `bed.Rental_Agreement` exists, query the linked `Rental_Agreements` record:
    - Set `Rental_Item = agmt.Rental_Item`, `Device_Model = bed.Rental_Item` (with fallback to `Rental_Items`), and `Agreed_Rate = agmt.Agreed_Rate`.
    - If `agmt.Billing_Unit == "Day"`, set `Rental_Mode = "Pay Per Use (Daily)"`; else `"Fixed Monthly Rental"`.
  - If usage dates are already present, trigger recalculation of `Number_of_Days`, `Taxable_Value`, `GST_Amount`, and `Invoice_Amount`.

### Workflow 3: On User Input of Usage Start Date & Usage End Date
- **File**: `scripts/10_rental_logs/2_2_Rental_Logs_onuserinput_dates.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Usage_Start_Date / Usage_End_Date -> On User Input
- **Plain English Logic**:
  - When both start and end dates are present:
    - Verify `Usage_End_Date >= Usage_Start_Date`. If invalid, alert operator and clear `Usage_End_Date`.
    - Calculate `Number_of_Days = (Usage_End_Date - Usage_Start_Date) + 1`.
    - If `Rental_Mode == "Pay Per Use (Daily)"`: `Taxable_Value = Number_of_Days * Agreed_Rate`.
    - If `Rental_Mode == "Fixed Monthly Rental"`: `Taxable_Value = Agreed_Rate`.
    - Calculate `GST_Amount = Taxable_Value * 0.05`.
    - Calculate `Invoice_Amount = Taxable_Value + GST_Amount`.

### Workflow 4: On User Input of Agreed Rate
- **File**: `scripts/10_rental_logs/2_3_Rental_Logs_onuserinput_agreedrate.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Agreed_Rate -> On User Input
- **Plain English Logic**:
  - Recalculates `Taxable_Value` based on `Rental_Mode` (Daily vs Monthly).
  - Recalculates `GST_Amount = Taxable_Value * 0.05` and `Invoice_Amount = Taxable_Value + GST_Amount`.

### Workflow 5: On Validate (Form Submission Guards)
- **File**: `scripts/10_rental_logs/3_Rental_Logs_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Enforce mandatory fields: `Bed_Station`, `Control_Unit`, `Overlay`, `Usage_Start_Date`, `Usage_End_Date`, `Agreed_Rate`.
  - Date sanity check: Verify `Usage_End_Date >= Usage_Start_Date`.
  - Rate check: Verify `Agreed_Rate > 0`.
  - **Date Overlap Guard**: Query existing uncancelled `Rental_Logs` records for the same `Bed_Station` where date ranges collide (`Usage_Start_Date <= input.Usage_End_Date && Usage_End_Date >= input.Usage_Start_Date && Billing_Status != "Cancelled / Void"`). If an overlap is detected, cancel submission and alert operator to prevent double-billing.

---

## 4. Phase 2 Zoho Books Integration (Deferred)

The following fields and workflows are defined for subsequent implementation once core logging and ledger commissioning are live:
- **Zoho Books Invoice ID** `[Single Line]`
- **Zoho Books Invoice No** `[Single Line]`
- **Invoice Date** `[Date]`
- **Invoice URL** `[URL]`
- **Billing Notes** `[Multi-Line]`
- Custom Record Action: `[ Push to Zoho Books ]`
- Report Action: `[ Push Consolidated Invoice to Books ]`
