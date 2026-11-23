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
4. **Product Model** `[Lookup -> Product_Models, displaying Product_Model_Name]` (Auto-derived hardware model lookup: `Altern4`, `Eturnal One`, etc.)
   - Field Link Name: `Product_Model`
   - Target Module: `Product_Models` (`Product_Model_Name`)
   - Form Behavior: Visible & Enabled on Add (auto-selected from `Product_Models` via serial prefix, user-editable); Visible & Locked in Edit mode.
5. **Lot** `[Lookup -> Lots, displaying Lot_ID]` (Manufacturing Lot Number)
   - Field Link Name: `Lot`
   - Target Module: `Lots` (`Lot_ID`)
   - Form Behavior: Visible & Enabled on Add (auto-populates with the latest lot record ID from `Lots`, user-editable); Visible & Enabled in Edit mode (technicians can reassign if needed).

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
9. **Device State** `[Dropdown - Hidden on Add]` (Commercial & deployment state)
   - Field Link Name: `Device_State`
   - Options: `"Rentally Active"`, `"Active in Demo"`, `"Idle with Customer"`, `"Available in Stock"`
   - Form Behavior: Hidden on Add (defaults to "Available in Stock" in background); Visible & Locked in Edit mode; updated during active customer deployment, returns, or stock intake.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/05_devices/1_Devices_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable `Serial_Number`, `Product_Model`, and `Lot` as interactive fields. Dynamically query `Lots` to auto-select the latest lot created by default. Keep `Device_Type` disabled to provide clear context without user editing. Hide lifecycle fields (`Current_Customer`, `Current_Office`, `Device_Stage`, `Device_State`).
  - In EDIT Mode: Lock `Serial_Number` and `Product_Model`. Keep `Lot` enabled so technicians can reassign lots if necessary. Display and lock context and lifecycle fields (`Device_State`, `Device_Type`, `Current_Customer`, `Current_Office`). Dynamically filter `Device_Stage` by clearing static choices via `clear Device_Stage;` and populating only the relevant stages for that hardware (`Control Unit` vs `Overlay`), re-asserting the current saved stage, and enabling `Device_Stage` for progressive stage updates.

### Workflow 2: On User Input (Step 1: Serial Number)
- **File**: `scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Serial_Number -> On User Input
- **Plain English Logic**:
  - Step 1: If cleared, reset derived fields (`Device_Type`, `Product_Model`, `Device_Stage`, `Device_State`).
  - Step 2: Duplicate check: Check if `Serial_Number` exists in `Devices`. If found, alert user and clear field.
  - Step 3: Detect `Device_Type`: If contains `"CU"` -> `"Control Unit"`; if `"OV"` -> `"Overlay"`; else alert and clear.
  - Step 4: Auto-initialize `Device_Stage`: If Control Unit -> `"0 - CU Mfg Not Yet Started"`; if Overlay -> `"0 - OV Mfg Not Yet Started"`.
  - Step 5: Detect `Product_Model`: Matches serial prefix to determine target model (`ALT` -> `"Altern4"`, `ET` -> `"Eturnal One"`), queries `Product_Models` to retrieve the matching record ID, and assigns `Product_Model` lookup, while allowing user to edit/reassign (e.g. choose `"Eturnal One with foam"`).
  - Step 6: Initialize `Device_State = "Available in Stock"`.

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
  - Step 5: Safety fallback: Ensure `Device_Type`, `Product_Model` lookup (queries `Product_Models` for matching model ID if unassigned), `Lot` lookup (fallback to latest created lot from `Lots`), initial `Device_Stage` (`"0 - CU Mfg Not Yet Started"` or `"0 - OV Mfg Not Yet Started"`), and `Device_State` ("Available in Stock") are populated.
  - Step 6: Bulk Edit & Compatibility Guard: Prevents assigning Overlay stages to Control Units or Control Unit stages to Overlays, protecting against mismatched stage assignments during report bulk edits.

### Workflow 4: On Form Submission Success
- **File**: `scripts/05_devices/4_Devices_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Redirect cleanly to `All_Devices` report view (`openUrl("#Report:All_Devices", "same window");`), returning user to master asset register.

---

## 4. Stateless Bulk Action Utilities & Report Custom Actions

To provide high-speed, user-friendly batch processing without relying on Creator's slow native report bulk edit, two dedicated stateless utility forms are implemented. They can be accessed directly from the navigation menu (for manual barcode scanning) or launched automatically as pre-filled pop-up modals from the `All_Devices` report via Custom Actions.

### A. Utility 1: Update Device Stage (`Update_Device_Stage`)
- **Form Type**: Stateless Form (No record storage; executes Deluge on submit)
- **Form Link Name**: `Update_Device_Stage`
- **Fields**:
  1. **Select Devices** `[Lookup -> Devices, displaying Serial_Number - Multi-Select]`
     - Field Link Name: `Select_Devices`
     - Behavior: Interactive field populated by user or pre-filled from `All_Devices` report custom action.
  2. **New Device Stage** `[Dropdown]`
     - Field Link Name: `New_Device_Stage`
     - Choices: All standard Control Unit and Overlay manufacturing and maintenance stages.
- **Automated Harmonization Principle**:
  - By default, if the batch contains mixed hardware types (Control Units and Overlays), the submit workflow automatically maps the selected stage to the counterpart stage for the respective hardware (e.g. selecting `5 - CU Assembly Completed & Not Tested` automatically applies `5 - OV Assembly Completed & Not Tested` to any Overlays in the batch).
- **Workflows**:
  - **On Load**: `scripts/05_devices/1_Update_Device_Stage_onload.deluge`
    - Evaluates pre-filled selection if passed from report.
  - **On Click of Submit**: `scripts/05_devices/5_1_Update_Device_Stage_onclick_submit.deluge`
    - Validates selection and stage. Fetches target device records in a single batch query, iterates and updates `Device_Stage` with automatic hardware-stage mapping, alerts user of success count, and closes/refreshes view.

### B. Utility 2: Update Device Custody (`Update_Device_Custody`)
- **Form Type**: Stateless Form (No record storage; executes Deluge on submit)
- **Form Link Name**: `Update_Device_Custody`
- **Fields**:
  1. **Select Devices** `[Lookup -> Devices, displaying Serial_Number - Multi-Select]`
     - Field Link Name: `Select_Devices`
     - Behavior: Interactive field populated by user or pre-filled from `All_Devices` report custom action.
  2. **Destination Type** `[Radio / Dropdown]`
     - Field Link Name: `Destination_Type`
     - Choices: `"Internal Office Transfer"`, `"Customer Deployment"`
  3. **Target Office** `[Lookup -> Offices, displaying Office_Name]`
     - Field Link Name: `Target_Office`
     - Behavior: Dynamically displayed when `Destination_Type == "Internal Office Transfer"`.
  4. **Target Customer** `[Lookup -> Customers, displaying Customer_Name]`
     - Field Link Name: `Target_Customer`
     - Behavior: Dynamically displayed when `Destination_Type == "Customer Deployment"`.
- **Workflows**:
  - **On Load**: `scripts/05_devices/1_Update_Device_Custody_onload.deluge`
    - Hides `Target_Office` and `Target_Customer` on initial load.
  - **On User Input (Destination Type)**: `scripts/05_devices/2_1_Update_Device_Custody_onuserinput_destinationtype.deluge`
    - Shows `Target_Office` and hides `Target_Customer` when `"Internal Office Transfer"` is chosen; shows `Target_Customer` and hides `Target_Office` when `"Customer Deployment"` is chosen.
  - **On Click of Submit**: `scripts/05_devices/5_1_Update_Device_Custody_onclick_submit.deluge`
    - Validates selections. Fetches target device records in a single batch query. For office transfer: updates `Current_Office`, clears `Current_Customer`, and sets `Device_State = "Available in Stock"`. For customer deployment: updates `Current_Customer`, clears `Current_Office`, and sets `Device_State = "Rentally Active"`. Alerts user of success count and closes/refreshes view.

### C. Report Custom Action Integration on `All_Devices`
- **Action 1**: `Update Stage` -> Opens `Update_Device_Stage` in a pop-up window, mapping selected record IDs into `Select_Devices`.
- **Action 2**: `Update Custody` -> Opens `Update_Device_Custody` in a pop-up window, mapping selected record IDs into `Select_Devices`.
