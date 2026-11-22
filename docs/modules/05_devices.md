# MODULE 5: DEVICES (ASSET MASTER)

**Form Name**: `Devices`  
**Form Link Name**: `Devices`  
**Default Report**: `All_Devices`  
**Purpose**: Physical inventory tracking, manufacturing stage progress, and complete lifecycle ledger of EVERY individual piece of hardware.

---

## 1. Core Manufacturing & Operational Principles
- **Complete Independence**: Control Units (CU) and Overlays (OV) are manufactured, assembled, and stocked completely independently. They are **never** manufactured in fixed pairs.
- **Dynamic Bedside Pairing**: They are **never** manufactured in fixed pairs. They are paired dynamically at the patient bedside during field deployment.

### Serial Number Intelligence
- **Primary Key**: `Serial_Number` (Single Line, Mandatory, Unique, Barcode/QR enabled).
- **Format**: `[ModelPrefix][DeviceType][SequenceNumber]`
  - Device Type: `CU` -> "Control Unit", `OV` -> "Overlay"
  - Model Prefix: `ALT4` / `ALT` -> "Altern4™", `ETO` -> "Eturnal One™"
  - Examples: `ALT4CU053`, `ALTOV017`, `ETOCU065`, `ETOOV064`.

---

## 2. Field Specifications

### A. User Interactive Field (Entered on intake):
1. **Serial Number** `[Single Line]` (Mandatory, Unique, Primary Key, Barcode/QR capable)
   - Field Link Name: `Serial_Number`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.

### B. Auto-Derived Context & Model Fields:
3. **Device Type** `[Dropdown - Disabled Context]` (Auto-derived from serial: `"Control Unit"`, `"Overlay"`)
   - Field Link Name: `Device_Type`
   - Form Behavior: Visible & Disabled on Add and Edit.
4. **Product Model** `[Dropdown]` (Auto-derived hardware model: `Altern4`, `Eturnal One`, `Eturnal One with foam`)
   - Field Link Name: `Product_Model`
   - Options: `"Altern4"`, `"Eturnal One"`, `"Eturnal One with foam"`
   - Form Behavior: Visible & Enabled on Add (auto-selected on serial input, user-editable); Visible & Locked in Edit mode.
5. **Lot** `[Dropdown]` (Manufacturing Lot Number)
   - Field Link Name: `Lot`
   - Options: Current lot numbers (e.g. `26AB02`, `26FB03`, etc.).
   - Decoding Format: `[YY][M][Seq]` (`26` = 2026, `A` = January, `B` = February, ..., `F` = June; remaining digits = batch order number in that year).
   - Form Behavior: Visible & Enabled on Add (auto-fills latest lot `"26FB03"` on load, user-editable); Visible & Enabled in Edit mode (technicians can reassign if needed).

### C. Metric & Lifecycle Fields (Hidden on Add):
*Note: These are operational status, customer/office assignment, and lifecycle metric fields. They are not entered during initial device intake and are hidden on Add mode.*
6. **Current Customer** `[Lookup -> Customers, displaying Customer_Name - Hidden on Add]` (Customer deployment allocation)
   - Field Link Name: `Current_Customer`
   - Form Behavior: Hidden on Add; Visible & Locked in Edit mode; shown in reports.
7. **Current Office** `[Lookup -> Offices, displaying Office_Name - Hidden on Add]` (Office / Warehouse location)
   - Field Link Name: `Current_Office`
   - Form Behavior: Hidden on Add; Visible & Locked in Edit mode; shown in reports. Updated via stock transfers / intake.
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
9. **Rental State** `[Dropdown - Hidden on Add]` (Commercial deployment state)
   - Field Link Name: `Rental_State`
   - Options: `"Active"`, `"Inactive"`
   - Form Behavior: Hidden on Add (defaults to "Inactive" in background); Visible & Locked in Edit mode; updated during active customer deployment (`Active` on patient deployment, `Inactive` when in warehouse/buffer).

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/05_devices/1_Devices_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable `Serial_Number`, `Product_Model`, and `Lot` as interactive fields. Auto-select the latest lot (`"26FB03"`) by default. Keep `Device_Type` disabled to provide clear context without user editing. Hide lifecycle fields (`Current_Customer`, `Current_Office`, `Device_Stage`, `Rental_State`).
  - In EDIT Mode: Lock `Serial_Number` and `Product_Model`. Keep `Lot` enabled so technicians can reassign lots if necessary. Display and lock context and lifecycle fields (`Rental_State`, `Device_Type`, `Current_Customer`, `Current_Office`). Dynamically filter `Device_Stage` by clearing static choices via `clear Device_Stage;` and populating only the relevant stages for that hardware (`Control Unit` vs `Overlay`), re-asserting the current saved stage, and enabling `Device_Stage` for progressive stage updates.

### Workflow 2: On User Input (Step 1: Serial Number)
- **File**: `scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Serial_Number -> On User Input
- **Plain English Logic**:
  - Step 1: If cleared, reset derived fields (`Device_Type`, `Product_Model`, `Device_Stage`, `Rental_State`).
  - Step 2: Duplicate check: Check if `Serial_Number` exists in `Devices`. If found, alert user and clear field.
  - Step 3: Detect `Device_Type`: If contains `"CU"` -> `"Control Unit"`; if `"OV"` -> `"Overlay"`; else alert and clear.
  - Step 4: Auto-initialize `Device_Stage`: If Control Unit -> `"0 - CU Mfg Not Yet Started"`; if Overlay -> `"0 - OV Mfg Not Yet Started"`.
  - Step 5: Detect `Product_Model`: Matches serial prefix to determine target model (`ALT` -> `"Altern4"`, `ET` -> `"Eturnal One"`) and directly sets `Product_Model` dropdown choice, while allowing user to edit (e.g. choose `"Eturnal One with foam"`).
  - Step 6: Initialize `Rental_State = "Inactive"`.

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
  - Step 1: Mandatory check: Ensure `Serial_Number` is entered.
  - Step 2: Sanitize `Serial_Number` (trim & uppercase).
  - Step 3: Duplicate check: Query `Devices` to prevent duplicate serial numbers.
  - Step 4: Verify syntax: Serial number must contain "CU" or "OV", and begin with valid prefix.
  - Step 5: Safety fallback: Ensure `Device_Type`, `Product_Model` dropdown, initial `Device_Stage` (`"0 - CU Mfg Not Yet Started"` or `"0 - OV Mfg Not Yet Started"`), and `Rental_State` ("Inactive") are populated.
  - Step 6: Bulk Edit & Compatibility Guard: Prevents assigning Overlay stages to Control Units or Control Unit stages to Overlays, protecting against mismatched stage assignments during report bulk edits.

### Workflow 4: On Form Submission Success
- **File**: `scripts/05_devices/4_Devices_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Redirect cleanly to `All_Devices` report view (`openUrl("#Report:All_Devices", "same window");`), returning user to master asset register.
