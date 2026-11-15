# MODULE 10: RENTAL LOGS

**Form Name**: `Rental Logs`  
**Form Link Name**: `Rental_Logs`  
**Default Report**: `All_Rental_Logs`  
**Purpose**: Primary commercial and operational billing ledger recording patient usage periods across customer bed stations. Captures physical device pairings (Control Unit and Overlay serials), exact calendar periods, and calculates taxable amounts, 5% GST, and total invoice values via native formula fields. Bridges operational activity to Zoho Books invoices.

---

## 1. Unified Usage & Billing Timeline Ledger

`Rental_Logs` serves as the single authoritative operational and commercial ledger across all customer bed stations:

### 1. Pay-Per-Use (Daily):
- Operator selects `Bed_Station`; system auto-fetches linked active Control Unit, Overlay, and Daily Agreed Rate.
- User enters `Usage_Start_Date` and `Usage_End_Date`.
- System calculates `Number_of_Days = (Usage_End_Date - Usage_Start_Date) + 1` (inclusive).
- Computes:
  - `Taxable_Value = Number_of_Days * Agreed_Rate`
  - `GST_Amount` = Native Formula (`Taxable_Value * 0.05`)
  - `Invoice_Amount` = Native Formula (`Taxable_Value + GST_Amount`)
- **Mid-Month Hardware Swaps**: If a component (e.g. Control Unit) was replaced during the month, staff records separate entries for each device-pairing interval (e.g. Jan 1–14 with CU-01, Jan 15–31 with replacement CU-02). This provides complete, audit-proof clinical tracking and exact billing line items.

### 2. Fixed Monthly Rental:
- Generated at month-end for bed stations under a monthly recurring agreement.
- Attaches the assigned Bed Station, active Control Unit, and Overlay device lookups.
- Operator enters billing period (e.g., 1st to end of month).
- Computes flat monthly fee regardless of days:
  - `Taxable_Value = Agreed_Rate`
  - `GST_Amount` = Native Formula (`Taxable_Value * 0.05`)
  - `Invoice_Amount` = Native Formula (`Taxable_Value + GST_Amount`)

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
   - Form Behavior: Auto-populated from linked `Rental_Agreement.Agreed_Rate`. Enabled on Add and Edit for authorized adjustments.
7. **Number of Days** `[Number]` (Auto-computed, Editable)
   - Field Link Name: `Number_of_Days`
   - Form Behavior: Auto-calculated when dates are entered (`(End - Start) + 1`). Editable if manual operational adjustments are required.

### B. Auto-Derived & Formula Fields:
8. **Rental Log ID** `[Auto-Number - Native Auto-Hidden on Add]` (Primary Key, Prefix: RL-, e.g. `RL-0001`)
   - Field Link Name: `Rental_Log_ID`
   - Form Behavior: Auto-Hidden on Add (PK; Natively locked in Edit).
9. **Taxable Value** `[Currency]`
   - Field Link Name: `Taxable_Value`
   - Form Behavior: Visible & Disabled on Add and Edit. Computed commercial amount before statutory taxes.
10. **GST Amount** `[Formula Field]`
    - Field Link Name: `GST_Amount`
    - Expression: `Taxable_Value * 0.05`
    - Form Behavior: Native Formula Field; automatically computed by Zoho Creator in browser.
11. **Invoice Amount** `[Formula Field]`
    - Field Link Name: `Invoice_Amount`
    - Expression: `Taxable_Value + GST_Amount`
    - Form Behavior: Native Formula Field; automatically computed by Zoho Creator in browser.
12. **Billing Status** `[Dropdown]`
    - Field Link Name: `Billing_Status`
    - Options: `"Pending Invoice"`, `"Invoiced"`, `"Cancelled / Void"` (Default: `"Pending Invoice"`)
    - Form Behavior: Hidden on Add (defaults to `"Pending Invoice"`); Visible & Disabled in Edit mode.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/10_rental_logs/1_Rental_Logs_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable interactive user entry fields (`Bed_Station`, `Control_Unit`, `Overlay`, `Usage_Start_Date`, `Usage_End_Date`, `Agreed_Rate`, `Number_of_Days`). Disable `Taxable_Value`. Hide `Billing_Status` and initialize default `Billing_Status = "Pending Invoice"`. (Note: `GST_Amount` and `Invoice_Amount` are native formula fields and are automatically read-only).
  - In EDIT Mode: Show and lock `Billing_Status`. Lock `Bed_Station` and `Taxable_Value`. If `Billing_Status == "Pending Invoice"`, allow adjustments to dates, rate, duration, and hardware lookups.

### Workflow 2: On User Input of Bed Station
- **File**: `scripts/10_rental_logs/2_1_Rental_Logs_onuserinput_bedstation.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Bed_Station -> On User Input
- **Plain English Logic**:
  - If `Bed_Station` is cleared, reset dependent fields (`Control_Unit`, `Overlay`, `Agreed_Rate`, `Taxable_Value`).
  - Query selected `Bed_Stations` record.
  - Set `Control_Unit = bed_rec.Control_Unit` and `Overlay = bed_rec.Overlay_Asset`.
  - If `bed_rec.Rental_Agreement` exists, query `Rental_Agreements` to fetch `Agreed_Rate`.
  - If usage dates are already present, trigger calculation of `Number_of_Days` and `Taxable_Value`.

### Workflow 3: On User Input of Usage Dates
- **File**: `scripts/10_rental_logs/2_2_Rental_Logs_onuserinput_dates.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Usage_Start_Date / Usage_End_Date -> On User Input
- **Plain English Logic**:
  - When both start and end dates are present:
    - Verify `Usage_End_Date >= Usage_Start_Date`. If invalid, alert operator and clear `Usage_End_Date`.
    - Compute inclusive duration: `Number_of_Days = (diff_days + 1)`.
    - Calculate `Taxable_Value = Number_of_Days * input.Agreed_Rate` (or flat `Agreed_Rate` if agreement is monthly). Formula fields update `GST_Amount` and `Invoice_Amount` automatically in the UI.

### Workflow 4: On User Input of Agreed Rate
- **File**: `scripts/10_rental_logs/2_3_Rental_Logs_onuserinput_agreedrate.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Agreed_Rate -> On User Input
- **Plain English Logic**:
  - Recalculates `Taxable_Value` based on active duration. Formula fields update `GST_Amount` and `Invoice_Amount` automatically.

### Workflow 5: On Form Validation & Safety Guard
- **File**: `scripts/10_rental_logs/3_Rental_Logs_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Validates mandatory fields (`Bed_Station`, `Control_Unit`, `Overlay`, `Usage_Start_Date`, `Usage_End_Date`, `Agreed_Rate`).
  - Verifies `Usage_End_Date >= Usage_Start_Date` and `Taxable_Value > 0`.
  - Ensures default `Billing_Status = "Pending Invoice"`.
