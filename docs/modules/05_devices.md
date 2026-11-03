# MODULE 5: DEVICES (ASSET MASTER)

**Form Name**: `Devices`  
**Form Link Name**: `Devices`  
**Default Report**: `All_Devices`  
**Purpose**: Physical inventory tracking, manufacturing stage progress, and complete lifecycle ledger of EVERY individual piece of hardware.

---

## 1. Core Manufacturing & Operational Principles
- **Complete Independence**: Control Units (CU) and Overlays (OV) are manufactured, assembled, and stocked completely independently. They are **never** manufactured in fixed pairs.
- **Dynamic Bedside Pairing**: They are paired dynamically at the patient bedside in Module 8 (`Bed_Station`).
- **Traceability**: Every physical unit is linked to its parent production batch in `Lots`.
- **Independent Progress**: Individual CUs and Overlays inside the same lot progress through manufacturing stages at different paces.

### Serial Number Intelligence
- **Primary Key**: `Serial_Number` (Single Line, Mandatory, Unique, Barcode/QR enabled).
- **Format**: `[ModelPrefix][DeviceType][SequenceNumber]`
  - Device Type: `CU` -> "Control Unit", `OV` -> "Overlay"
  - Model Prefix: `ALT4` / `ALT` -> "Altern4™", `ETO` -> "Eturnal One™"
  - Examples: `ALT4CU053`, `ALTOV017`, `ETOCU065`, `ETOOV064`.

---

## 2. Field Specifications

### A. User Interactive Fields (The ONLY fields user enters on intake):
1. **Serial Number** `[Single Line]` (Mandatory, Unique, Primary Key, Barcode/QR capable)
   - Field Link Name: `Serial_Number`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.
2. **Lot Number** `[Lookup -> Lots, displaying Lot_ID]` (Mandatory parent lot traceability)
   - Field Link Name: `Lot_Number`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.

### B. Auto-Derived Context Fields (Disabled - Provide Visual Context without Editing):
*Note: These fields are auto-derived from the Serial Number syntax and are disabled on both Add and Edit to provide immediate visual feedback to the technician.*
3. **Device Type** `[Dropdown - Disabled Context]` (Auto-derived from serial: `"Control Unit"`, `"Overlay"`)
   - Field Link Name: `Device_Type`
   - Form Behavior: Visible & Disabled on Add and Edit.
4. **Device Model** `[Single Line - Disabled Context]` (Auto-derived hardware model: `"Altern4™"`, `"Eturnal One™"`)
   - Field Link Name: `Device_Model`
   - Form Behavior: Visible & Disabled on Add and Edit.

### C. Metric & Lifecycle Fields (Hidden on Add - For Reports, Blueprints & Stock Transfers):
*Note: These are operational status, custody, and lifecycle metric fields. They make no sense during initial device intake and are completely hidden on Add mode.*
5. **Current Location** `[Lookup -> Locations, displaying Location_s_Name - Hidden on Add]` (Auto-initialized to Primary Hub; updated by Stock Transfers)
   - Field Link Name: `Current_Location`
   - Form Behavior: Hidden on Add; Auto-initialized in background to Primary Hub; Visible & Locked in Edit mode; shown in reports.
6. **Current Customer** `[Lookup -> Customers, displaying Customer_Name - Hidden on Add]` (Updated by Stock Transfers on customer delivery / collection)
   - Field Link Name: `Current_Customer`
   - Form Behavior: Hidden on Add; Visible & Locked in Edit mode; shown in reports.
7. **Total Cumulative Days Used** `[Number - Hidden on Add]` (Default: 0; updated by Rental Log)
   - Field Link Name: `Total_Cumulative_Days_Used`
   - Form Behavior: Hidden on Add; Visible & Locked in Edit mode; shown in reports.
8. **Device Stage** `[Dropdown - Hidden on Add]` (Manufacturing, Logistics & Maintenance stage; editable in Edit mode)
   - Field Link Name: `Device_Stage`
   - Form Behavior: Hidden on Add (auto-initialized to Stage 0 based on serial number); Visible & Enabled in Edit mode with options dynamically filtered by `Device_Type`.
   - **Control Unit Options**:
     - `0 - CU Mfg Not Yet Started`
     - `1 - CU Enclosure Mfg in Progress`
     - `2 - CU Subsystems Assembled`
     - `3 - CU Yet to Assemble`
     - `4 - CU Assembly in Progress`
     - `5 - CU Assembly Completed & Not Tested`
     - `6 - CU Ready Condition`
     - `CU Retesting Needed`
     - `CU Repair Needed`
     - `CU Disinfection Needed`
   - **Overlay Options**:
     - `0 - OV Not Yet Started`
     - `1 - OV in Screen printing/Welding`
     - `2 - OV in Stitching`
     - `3 - OV Yet to Assemble`
     - `4 - OV Assembly In Progress`
     - `5 - OV Assembly Completed & Not Tested`
     - `6 - OV Ready Condition`
     - `OV Retesting Needed`
     - `OV Repair Needed`
     - `OV Disinfection Needed`
9. **Rental State** `[Dropdown - Hidden on Add]` (Commercial deployment state; renamed from Rental Stage)
   - Field Link Name: `Rental_State`
   - Options: `"Active"`, `"Inactive"`
   - Form Behavior: Hidden on Add (defaults to "Inactive" in background); Visible & Locked in Edit mode; updated via Bed Stations (`Active` on patient bed deployment, `Inactive` when closed/buffer/warehouse).
10. **Current Custody** `[Dropdown - Hidden on Add]` (Physical custody location tracking)
    - Field Link Name: `Current_Custody`
    - Options: `"Internal Office"`, `"Dispatched"`, `"In-Transit"`, `"Customer's Site"`
    - Form Behavior: Hidden on Add (defaults to "Internal Office" in background); Visible & Locked in Edit mode; updated via Stock Transfers and Bed Stations.
*(Note: Legacy 'Rental status' field has been completely removed).*

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/05_devices/1_Devices_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable `Serial_Number` and `Lot_Number` as the only user interactive fields. Keep `Device_Type` and `Device_Model` disabled to provide clear context without user editing. Hide metric, custody, and lifecycle fields (`Current_Customer`, `Current_Location`, `Total_Cumulative_Days_Used`, `Device_Stage`, `Rental_State`, `Current_Custody`). Initialize default `Total_Cumulative_Days_Used = 0`.
  - In EDIT Mode: Lock `Serial_Number` and `Lot_Number`. Display and lock all context and metric fields (`Rental_State`, `Current_Custody`, `Device_Type`, `Device_Model`, `Current_Location`, `Current_Customer`, `Total_Cumulative_Days_Used`). Dynamically filter and populate `Device_Stage` dropdown choices based on `Device_Type` (Control Unit vs Overlay), re-asserting the current saved stage, and enable `Device_Stage` for technician progressive stage updates.

### Workflow 2: On User Input (Step 1: Serial Number)
- **File**: `scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Serial_Number -> On User Input
- **Plain English Logic**:
  - Step 1: If cleared, reset derived fields (`Device_Type`, `Device_Model`, `Device_Stage`, `Rental_State`, `Current_Custody`, `Current_Location`).
  - Step 2: Duplicate check: Check if `Serial_Number` exists in `Devices`. If found, alert user and clear field.
  - Step 3: Detect `Device_Type`: If contains `"CU"` -> `"Control Unit"`; if `"OV"` -> `"Overlay"`; else alert and clear.
  - Step 4: Auto-initialize `Device_Stage`: If Control Unit -> `"0 - CU Mfg Not Yet Started"`; if Overlay -> `"0 - OV Not Yet Started"`.
  - Step 5: Detect `Device_Model`: If starts with `"ALT4"` or `"ALT"` -> `"Altern4™"`; if `"ETO"` -> `"Eturnal One™"`; else alert and clear.
  - Step 6: Initialize `Rental_State = "Inactive"` and `Current_Custody = "Internal Office"`.
  - Step 7: Query `Locations` where `Is_Primary == true` and auto-populate `Current_Location` with the Primary Hub.

### Workflow 3: On Form Validation & Safety Guard
- **File**: `scripts/05_devices/3_Devices_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory check: Ensure `Serial_Number` is entered and `Lot_Number` is selected.
  - Step 2: Sanitize `Serial_Number` (trim & uppercase).
  - Step 3: Duplicate check: Query `Devices` to prevent duplicate serial numbers.
  - Step 4: Verify syntax: Serial number must contain "CU" or "OV", and begin with valid prefix.
  - Step 5: Safety fallback: Ensure `Device_Type`, `Device_Model`, initial `Device_Stage`, `Rental_State` ("Inactive"), and `Current_Custody` ("Internal Office") are populated.

### Workflow 4: On Form Submission Success
- **File**: `scripts/05_devices/4_Devices_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Record successfully saved. Relies on Zoho Creator native form properties for post-submission handling (avoiding client-side modal loader lockups).
