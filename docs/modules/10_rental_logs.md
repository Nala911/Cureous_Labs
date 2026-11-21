# MODULE 10: RENTAL LOGS

**Form Name**: `Rental Logs`  
**Form Link Name**: `Rental_Logs`  
**Default Report**: `All_Rental_Logs`  
**Purpose**: Primary commercial and operational billing ledger recording patient usage periods across customer bed stations. Captures physical device pairings (Control Unit and Overlay serials), exact calendar periods, billable days used (accounting for inactive downtime), and calculates taxable amounts, 5% GST, and total invoice values via native formula fields. Bridges operational activity to Zoho Books invoices.

---

## 1. Unified Usage & Billing Timeline Ledger

`Rental_Logs` serves as the single authoritative operational and commercial ledger across all customer bed stations:

### 1. Pay-Per-Use (Daily):
- Operator selects `Bed_Station`; system auto-fetches linked `Customer`, `Rental_Agreement`, `Rental_Item`, active `Control_Unit`, active `Overlay`, `Agreed_Rate`, and sets `Billing_Type = "Pay-Per-Use (Daily)"`.
- User enters `Usage_Start_Date` and `Usage_End_Date`.
- System computes inclusive `Days_Used = (Usage_End_Date - Usage_Start_Date) + 1`.
- **Inactive Days Adjustment**: If the bed station was inactive for some days during the period (e.g. patient off the bed), the sales agent adjusts `Days_Used` directly.
- Computes:
  - `Taxable_Value = Days_Used * Agreed_Rate`
  - `GST_Amount` = Native Formula (`Taxable_Value * 0.05`)
  - `Invoice_Amount` = Native Formula (`Taxable_Value + GST_Amount`)

### 2. Fixed Monthly Rental:
- Generated at month-end (or billing cycle) for bed stations under a recurring monthly agreement.
- Attaches the assigned `Bed_Station`, `Customer`, `Rental_Agreement`, `Rental_Item`, and active `Control_Unit` and `Overlay`. Sets `Billing_Type = "Fixed (Monthly)"`.
- Operator enters billing period (`Usage_Start_Date` and `Usage_End_Date`).
- System computes default `Days_Used = (Usage_End_Date - Usage_Start_Date) + 1`.
- Defaults to full month fixed amount: `Taxable_Value = Agreed_Rate`.
- **Proration / Custom Billing**: If the bed station was deployed mid-month or inactive for several days, user can adjust `Days_Used` (which auto-prorates `Taxable_Value`) or directly edit `Taxable_Value` to any custom agreed amount.
- Computes:
  - `GST_Amount` = Native Formula (`Taxable_Value * 0.05`)
  - `Invoice_Amount` = Native Formula (`Taxable_Value + GST_Amount`)

### 3. Mid-Month Hardware Swaps:
- If a device (Control Unit or Overlay) was replaced during the billing period:
  - **Consolidated Monthly Invoice**: User selects the active Bed Station, which pulls the current active hardware pair, bills the full month, and records swap notes in `Notes` (e.g. *"CU swapped on 14-Nov"*).
  - **Split Audit Invoices**: User can log distinct intervals for each hardware pair (e.g. Nov 1–14 with CU-01 and Nov 15–30 with replacement CU-02) adjusting `Taxable_Value` accordingly without double-billing.

---

## 2. Field Specifications

### A. User Interactive Fields:
1. **Bed Station** `[Lookup -> Bed_Stations, displaying Bed_Station_Title]` (Mandatory)
   - Field Link Name: `Bed_Station`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode. Primary relational anchor that automatically populates contract context and hardware pairings.
2. **Control Unit** `[Lookup -> Devices, displaying Serial_Number]` (Mandatory)
   - Field Link Name: `Control_Unit`
   - Form Behavior: Auto-populated from `Bed_Station.Control_Unit`. Enabled on Add and Edit to accommodate mid-period hardware exchanges.
3. **Overlay** `[Lookup -> Devices, displaying Serial_Number]` (Mandatory)
   - Field Link Name: `Overlay`
   - Form Behavior: Auto-populated from `Bed_Station.Overlay_Asset`. Enabled on Add and Edit to accommodate mid-period hardware exchanges.
4. **Usage Start Date** `[Date]` (Mandatory)
   - Field Link Name: `Usage_Start_Date`
   - Form Behavior: Visible & Enabled on Add and Edit. First calendar day of patient usage or billing period.
5. **Usage End Date** `[Date]` (Mandatory)
   - Field Link Name: `Usage_End_Date`
   - Form Behavior: Visible & Enabled on Add and Edit. Last calendar day of patient usage or billing period.
6. **Days Used** `[Number]` (Mandatory, Auto-computed, User Editable)
   - Field Link Name: `Days_Used`
   - Form Behavior: Auto-calculated when dates are entered (`(End - Start) + 1`). Fully user-editable to deduct inactive days or enter actual billable days.
7. **Agreed Rate** `[Currency]` (Mandatory)
   - Field Link Name: `Agreed_Rate`
   - Form Behavior: Auto-populated from linked `Rental_Agreement.Agreed_Rate`. Enabled on Add and Edit for authorized adjustments.
8. **Taxable Value** `[Currency]` (Auto-computed, User Editable)
   - Field Link Name: `Taxable_Value`
   - Form Behavior: Visible & Enabled on Add and Edit. Auto-computed based on billing model, yet user-editable for authorized commercial or prorated adjustments.
9. **Notes / Downtime Details** `[Single Line]` (Optional)
   - Field Link Name: `Notes`
   - Form Behavior: Visible & Enabled on Add and Edit. Documents downtime reasons, patient discharge notes, or hardware swap references.

### B. Relational Context Fields (Disabled):
10. **Customer** `[Lookup -> Customers, displaying Customer_Name]` (Disabled Context)
    - Field Link Name: `Customer`
    - Form Behavior: Visible & Disabled on Add and Edit. Auto-populated from `Bed_Station.Customer`. Essential for customer filtering and future Zoho Books mapping.
11. **Rental Agreement** `[Lookup -> Rental_Agreements, displaying Agreement_Title]` (Disabled Context)
    - Field Link Name: `Rental_Agreement`
    - Form Behavior: Visible & Disabled on Add and Edit. Auto-populated from `Bed_Station.Rental_Agreement`.
12. **Rental Item** `[Lookup -> Rental_Items, displaying Item_Name]` (Disabled Context)
    - Field Link Name: `Rental_Item`
    - Form Behavior: Visible & Disabled on Add and Edit. Auto-populated from `Rental_Agreement.Rental_Item`.
13. **Billing Type** `[Dropdown]` (Disabled Context)
    - Field Link Name: `Billing_Type`
    - Options: `"Pay-Per-Use (Daily)"`, `"Fixed (Monthly)"`
    - Form Behavior: Visible & Disabled on Add and Edit. Auto-derived from `Rental_Item.Billing_Unit`.

### C. Auto-Derived & Formula Fields:
14. **Rental Log ID** `[Auto-Number - Native Auto-Hidden on Add]` (Primary Key, Prefix: RL-, e.g. `RL-0001`)
    - Field Link Name: `Rental_Log_ID`
    - Form Behavior: Auto-Hidden on Add (PK; Natively locked in Edit).
15. **GST Amount** `[Formula Field]`
    - Field Link Name: `GST_Amount`
    - Expression: `Taxable_Value * 0.05`
    - Form Behavior: Native Formula Field; automatically computed by Zoho Creator in browser.
16. **Invoice Amount** `[Formula Field]`
    - Field Link Name: `Invoice_Amount`
    - Expression: `Taxable_Value + GST_Amount`
    - Form Behavior: Native Formula Field; automatically computed by Zoho Creator in browser.
17. **Billing Status** `[Dropdown]`
    - Field Link Name: `Billing_Status`
    - Options: `"Pending Invoice"`, `"Invoiced"`, `"Cancelled / Void"` (Default: `"Pending Invoice"`)
    - Form Behavior: Hidden on Add (defaults to `"Pending Invoice"`); Visible & Disabled in Edit mode.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/10_rental_logs/1_Rental_Logs_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable interactive user entry fields (`Bed_Station`, `Control_Unit`, `Overlay`, `Usage_Start_Date`, `Usage_End_Date`, `Days_Used`, `Agreed_Rate`, `Taxable_Value`, `Notes`). Disable context fields (`Customer`, `Rental_Agreement`, `Rental_Item`, `Billing_Type`). Hide lifecycle field `Billing_Status` and initialize default `Billing_Status = "Pending Invoice"`. (Note: `Rental_Log_ID`, `GST_Amount`, and `Invoice_Amount` are system/formula fields and are never targeted with UI state commands).
  - In EDIT Mode: Show `Billing_Status` and lock `Bed_Station`. Disable context fields. If `Billing_Status == "Pending Invoice"`, allow editing of dates, days used, hardware, notes, agreed rate, and taxable value. If `Billing_Status == "Invoiced"`, lock all user fields to protect audited financial data.

### Workflow 2: On User Input of Bed Station
- **File**: `scripts/10_rental_logs/2_1_Rental_Logs_onuserinput_bedstation.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Bed_Station -> On User Input
- **Plain English Logic**:
  - When `Bed_Station` is cleared, reset all dependent fields (`Customer`, `Rental_Agreement`, `Rental_Item`, `Billing_Type`, `Control_Unit`, `Overlay`, `Agreed_Rate`, `Days_Used`, `Taxable_Value`).
  - When selected, query `Bed_Stations` record:
    - Auto-populate `Customer`, `Rental_Agreement`, `Control_Unit`, and `Overlay`.
    - Query linked `Rental_Agreements` to fetch `Agreed_Rate` and `Rental_Item`.
    - Query linked `Rental_Items` to check `Billing_Unit`:
      - If `"Month"`: set `Billing_Type = "Fixed (Monthly)"`.
      - If `"Day"`: set `Billing_Type = "Pay-Per-Use (Daily)"`.
  - If usage dates are already present, trigger recalculation of days used and taxable value.

### Workflow 3: On User Input of Usage Dates
- **File**: `scripts/10_rental_logs/2_2_Rental_Logs_onuserinput_dates.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Usage_Start_Date / Usage_End_Date -> On User Input
- **Plain English Logic**:
  - When both start and end dates are present:
    - Verify `Usage_End_Date >= Usage_Start_Date`. If invalid, alert operator and clear `Usage_End_Date`, `Days_Used`, and `Taxable_Value`.
    - Compute inclusive `Days_Used = (Usage_End_Date - Usage_Start_Date) + 1`.
    - Recalculate `Taxable_Value`:
      - Fixed Monthly: default `Taxable_Value = Agreed_Rate`.
      - Pay-Per-Use: `Taxable_Value = Days_Used * Agreed_Rate`.

### Workflow 4: On User Input of Days Used
- **File**: `scripts/10_rental_logs/2_3_Rental_Logs_onuserinput_daysused.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Days_Used -> On User Input
- **Plain English Logic**:
  - Ensures `Days_Used > 0`.
  - If `Usage_Start_Date` and `Usage_End_Date` are present, calculate total calendar days `tot_days = (End - Start) + 1`. If `Days_Used > tot_days`, warn operator and reset to `tot_days`.
  - Recalculates `Taxable_Value`:
    - Pay-Per-Use: `Days_Used * Agreed_Rate`.
    - Fixed Monthly: auto-prorates if days were reduced: `round((Agreed_Rate / tot_days) * Days_Used, 2)`.

### Workflow 5: On User Input of Agreed Rate
- **File**: `scripts/10_rental_logs/2_4_Rental_Logs_onuserinput_agreedrate.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Agreed_Rate -> On User Input
- **Plain English Logic**:
  - Recalculates `Taxable_Value` based on `Billing_Type` and `Days_Used`.

### Workflow 6: On Form Validation & Safety Guard
- **File**: `scripts/10_rental_logs/3_Rental_Logs_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Mandatory validation on `Bed_Station`, `Control_Unit`, `Overlay`, `Usage_Start_Date`, `Usage_End_Date`, `Days_Used`, `Agreed_Rate`, and `Taxable_Value`.
  - Verifies `Usage_End_Date >= Usage_Start_Date`.
  - Verifies `Days_Used > 0`.
  - Overlap Guard: Queries `Rental_Logs` to prevent duplicate overlapping active billing periods on the same `Bed_Station` (excluding `Cancelled / Void` logs and current record in Edit mode).
  - Guarantees valid positive `Taxable_Value` and default `Billing_Status = "Pending Invoice"`.

### Workflow 7: On Form Submission Success
- **File**: `scripts/10_rental_logs/4_Rental_Logs_onsuccess.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Success
- **Plain English Logic**:
  - Clean redirect: Redirects to `All_Rental_Logs` report view (`openUrl("#Report:All_Rental_Logs", "same window");`).
