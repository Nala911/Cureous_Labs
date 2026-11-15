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
2. **Lot** `[Lookup -> Lots, displaying Lot_ID]` (Mandatory parent lot traceability)
   - Field Link Name: `Lot`
   - Form Behavior: Visible & Enabled on Add (auto-filled with latest lot on serial number entry, editable by user); Locked in Edit mode.

### B. Auto-Derived Context Fields (Disabled - Provide Visual Context without Editing):
*Note: These fields are auto-derived from the Serial Number syntax and are disabled on both Add and Edit to provide immediate visual feedback to the technician.*
3. **Device Type** `[Dropdown - Disabled Context]` (Auto-derived from serial: `"Control Unit"`, `"Overlay"`)
   - Field Link Name: `Device_Type`
   - Form Behavior: Visible & Disabled on Add and Edit.
4. **Product Model** `[Lookup -> Product_Model, displaying Product_Model_Name - Disabled Context]` (Auto-derived hardware model lookup: Altern4, Altern8, EturnalOne, Eturnal One with foam)
   - Field Link Name: `Product_Model`
   - Form Behavior: Visible & Disabled on Add and Edit.

### C. Metric & Lifecycle Fields (Hidden on Add - For Reports, Blueprints & Stock Transfers):
*Note: These are operational status, custody, and lifecycle metric fields. They make no sense during initial device intake and are completely hidden on Add mode.*
5. **Current Office** `[Lookup -> Offices, displaying Office_Name - Hidden on Add]` (Auto-initialized to Primary Hub; updated by Stock Transfers)
   - Field Link Name: `Current_Office`
   - Form Behavior: Hidden on Add; Auto-initialized in background to Primary Hub; Visible & Locked in Edit mode; shown in reports.
6. **Current Customer** `[Lookup -> Customers, displaying Customer_Name - Hidden on Add]` (Updated by Stock Transfers on customer delivery / collection)
   - Field Link Name: `Current_Customer`
   - Form Behavior: Hidden on Add; Visible & Locked in Edit mode; shown in reports.
7. **Device Stage** `[Dropdown - Hidden on Add]` (Manufacturing, Logistics & Maintenance stage; editable in Edit mode)
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
     - `0 - OV Mfg Not Yet Started`
     - `1 - OV in Screen printing/Welding`
     - `2 - OV in Stitching`
     - `3 - OV Yet to Assemble`
     - `4 - OV Assembly In Progress`
     - `5 - OV Assembly Completed & Not Tested`
     - `6 - OV Ready Condition`
     - `OV Retesting Needed`
     - `OV Repair Needed`
     - `OV Disinfection Needed`
8. **Rental State** `[Dropdown - Hidden on Add]` (Commercial deployment state; renamed from Rental Stage)
   - Field Link Name: `Rental_State`
   - Options: `"Active"`, `"Inactive"`
   - Form Behavior: Hidden on Add (defaults to "Inactive" in background); Visible & Locked in Edit mode; updated via Bed Stations (`Active` on patient bed deployment, `Inactive` when closed/buffer/warehouse).
9. **Current Custody** `[Lookup -> Stock_Transfers, displaying Transfer_Status - Hidden on Add]` (Physical custody tracking)
   - Field Link Name: `Current_Custody`
   - Form Behavior: Hidden on Add (defaults to blank/null on initial intake); Visible & Locked in Edit mode; automatically updated by Module 7 (`Stock_Transfers`) to point to the latest transfer record.
*(Note: Legacy 'Rental status' field has been completely removed).*

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/05_devices/1_Devices_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable `Serial_Number` and `Lot` as the only user interactive fields. Keep `Device_Type` and `Product_Model` disabled to provide clear context without user editing. Hide custody and lifecycle fields (`Current_Customer`, `Current_Office`, `Device_Stage`, `Rental_State`, `Current_Custody`).
  - In EDIT Mode: Lock `Serial_Number` and `Lot`. Display and lock all context and custody fields (`Rental_State`, `Current_Custody`, `Device_Type`, `Product_Model`, `Current_Office`, `Current_Customer`). Dynamically filter `Device_Stage` by clearing static choices via `clear Device_Stage;` and populating only the relevant stages for that hardware (`Control Unit` vs `Overlay`), re-asserting the current saved stage, and enabling `Device_Stage` for progressive stage updates.

### Workflow 2: On User Input (Step 1: Serial Number)
- **File**: `scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Serial_Number -> On User Input
- **Plain English Logic**:
  - Step 1: If cleared, reset derived fields (`Device_Type`, `Product_Model`, `Device_Stage`, `Rental_State`, `Current_Custody`, `Current_Office`, `Lot`).
  - Step 2: Duplicate check: Check if `Serial_Number` exists in `Devices`. If found, alert user and clear field.
  - Step 3: Detect `Device_Type`: If contains `"CU"` -> `"Control Unit"`; if `"OV"` -> `"Overlay"`; else alert and clear.
  - Step 4: Auto-initialize `Device_Stage`: If Control Unit -> `"0 - CU Mfg Not Yet Started"`; if Overlay -> `"0 - OV Mfg Not Yet Started"`.
  - Step 5: Detect `Product_Model`: Matches serial prefix to determine target model (`Altern4`, `Altern8`, `EturnalOne`, `Eturnal One with foam`), queries `Product_Model` and assigns lookup ID; else alerts and clears.
  - Step 6: Initialize `Rental_State = "Inactive"` and leave `Current_Custody = null` (blank on intake; stamped on first transfer).
  - Step 7: Query `Offices` where `Is_Primary == true` and auto-populate `Current_Office` with the Primary Hub.
  - Step 8: Auto-populate `Lot` with the latest manufacturing lot from `Lots` (using optimized query `sort by Added_Time desc range from 1 to 1`), while keeping `Lot` fully enabled so the technician can manually change the lot if required.

### Workflow 2.2: On Update of Device Stage (Auto-Harmonization Guard)
- **File**: `scripts/05_devices/2_2_Devices_onupdate_devicestage.deluge`
- **Trigger**: Edited -> Field Actions -> Device_Stage -> On Update
- **Plain English Logic**:
  - Step 1: Detect hardware classification (`Control Unit` vs `Overlay`).
  - Step 2: Compare updated `Device_Stage` against hardware type.
  - Step 3: If an Overlay receives a Control Unit stage (e.g. during report Bulk Edit), automatically harmonize and convert it to the exact equivalent Overlay stage (e.g., `5 - CU Assembly Completed & Not Tested` -> `5 - OV Assembly Completed & Not Tested`).
  - Step 4: If a Control Unit receives an Overlay stage, automatically convert it to the exact equivalent Control Unit stage.

### Workflow 3: On Form Validation & Safety Guard
- **File**: `scripts/05_devices/3_Devices_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory check: Ensure `Serial_Number` is entered and `Lot` is selected.
  - Step 2: Sanitize `Serial_Number` (trim & uppercase).
  - Step 3: Duplicate check: Query `Devices` to prevent duplicate serial numbers.
  - Step 4: Verify syntax: Serial number must contain "CU" or "OV", and begin with valid prefix.
  - Step 5: Safety fallback: Ensure `Device_Type`, `Product_Model` lookup, initial `Device_Stage` (`"0 - CU Mfg Not Yet Started"` or `"0 - OV Mfg Not Yet Started"`), and `Rental_State` ("Inactive") are populated. Verify `Current_Office` is initialized to primary hub if blank.
  - Step 6: Bulk Edit & Compatibility Guard: Prevents assigning Overlay stages to Control Units or Control Unit stages to Overlays, protecting against mismatched stage assignments during report bulk edits.

### Workflow 4: On Form Submission Success
- **File**: `scripts/05_devices/4_Devices_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Redirect cleanly to `All_Devices` report view (`openUrl("#Report:All_Devices", "same window");`), returning user to master asset register.
