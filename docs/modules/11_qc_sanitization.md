# MODULE 11: QUALITY CONTROL & SANITIZATION

**Form Name**: `Quality_Control`  
**Form Link Name**: `Quality_Control`  
**Default Report**: `All_Quality_Controls`  
**Purpose**: Medical device safety verification, electrical tests, and sanitization tracking. Assets cannot be deployed to customers unless they hold a "Passed & Certified" QC certification!

---

## 1. Field Specifications

1. **Testing ID / QC ID** `[Auto-Number]` (Primary Key, e.g. `QC-2026-0245`) -> `Testing_ID`
2. **Asset ID** `[Lookup -> Devices, displaying Serial_Number]` (Mandatory) -> `Asset_ID`
3. **Lot Number** `[Lookup -> Lots, Read-Only]` (Auto-populated from Asset) -> `Lot_Number`
4. **Asset Type** `[Single Line - Read Only]` (Auto-filled: "Control Unit" or "Overlay") -> `Asset_Type`
5. **Inspector / Tester** `[User Lookup]` (Logged-in QA engineer) -> `Inspector`
6. **Test Date & Time** `[Date-Time]` (Mandatory) -> `Test_Date_Time`
7. **QC Stage** `[Dropdown]` (`"Post-Assembly"`, `"Post-Rental Return"`, `"Preventive Maintenance"`) -> `QC_Stage`

### Tests for Control Unit (Shown only if Asset Type == "Control Unit"):
8. **Pump Pressure Output (mmHg)** `[Decimal]` (Acceptance criteria: 30 - 80 mmHg) -> `Pump_Pressure_Output`
9. **Cycle Timing Check (Min)** `[Decimal]` (Acceptance criteria: 10 / 12 minutes alternating) -> `Cycle_Timing_Check`
10. **Audible / Visual Alarm Test** `[Dropdown]` (`"Pass"`, `"Fail"`) -> `Alarm_Test`
11. **Electrical Safety Test** `[Dropdown]` (`"Pass"`, `"Fail"`) -> `Electrical_Safety_Test`

### Tests for Overlay (Shown only if Asset Type == "Overlay"):
12. **Cell Airtightness 2-Hour Drop** `[Dropdown]` (`"Pass"`, `"Fail"`) -> `Cell_Airtightness_Drop`
13. **CPR Quick Deflate Valve Test** `[Dropdown]` (`"Pass"`, `"Fail"`) -> `CPR_Deflate_Test`
14. **Cover Waterproof Inspection** `[Dropdown]` (`"Intact / Clean"`, `"Damaged / Needs Replacement"`) -> `Cover_Waterproof_Check`

### Common Safety & Sanitization:
15. **Sanitization Status** `[Dropdown]` (`"Medical Grade Disinfection Complete"`, `"Not Sanitized"`) -> `Sanitization_Status`
16. **Overall QC Verdict** `[Dropdown]` (`"Passed & Certified for Clinical Use"`, `"Failed - Needs Repair"`, `"Scrap"`) -> `Overall_QC_Verdict`
17. **Next QC Due Date** `[Date]` (Default: +180 days from test date) -> `Next_QC_Due_Date`
18. **Remarks / Repair Notes** `[Multi-Line]` -> `Remarks`

---

## 2. Workflows & State Updates (On Success)

- **If Overall QC Verdict == "Passed & Certified for Clinical Use"**:
  - Update `Devices`:
    - `Current_Custody = "Internal Office"`
    - `Rental_State = "Inactive"`
    - `Device_Stage = "6 - CU Ready Condition"` (or `"6 - OV Ready Condition"`)
- **If Overall QC Verdict == "Failed - Needs Repair"**:
  - Update `Devices`:
    - `Current_Custody = "Internal Office"`
    - `Rental_State = "Inactive"`
    - `Device_Stage = "CU Repair Needed"` (or `"OV Repair Needed"`)
- **Parent Lot Rollup**:
  - Triggers `fn_recalculate_lot_metrics` to update Rent-Ready and In-Progress counters on the parent `Lots` record.
