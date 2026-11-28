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
7. **Current Office** `[Lookup -> Offices, displaying Office_Name - Auto-Default on Add]` (Office / Warehouse location)
   - Field Link Name: `Current_Office`
   - Form Behavior: Visible & Enabled on Add (auto-defaults to `"Bangalore Office"` on load, user-editable); Visible & Locked in Edit mode; shown in reports. Updated via stock transfers / intake.
8. **Device Stage** `[Dropdown - Hidden on Add]` (Manufacturing, Logistics & Maintenance stage; editable in Edit mode except while actively deployed)
   - Field Link Name: `Device_Stage`
   - Form Behavior: Hidden on Add (auto-initialized to `"0 - Mfg Not Yet Started"`); Visible & Enabled in Edit mode (locked while `Device_State` is `"Rentally Active"` or `"Active in Demo"`) with shared lifecycle options plus hardware-specific Stages 1–2 dynamically filtered by `Device_Type`.
   - **Shared Lifecycle Options (Both Control Units & Overlays)**:
     - `0 - Mfg Not Yet Started`
     - `3 - Yet to Assemble`
     - `4 - Assembly in Progress`
     - `5 - Assembly Completed & Not Tested`
     - `6 - Ready Condition`
     - `Retesting Needed`
     - `Repair Needed`
     - `Disinfection Needed`
   - **Control Unit-Specific Options (Stages 1–2)**:
     - `1 - CU Enclosure Mfg in Progress`
     - `2 - CU Subsystems Assembled`
   - **Overlay-Specific Options (Stages 1–2)**:
     - `1 - OV in Screen Printing / Welding`
     - `2 - OV in Stitching`
9. **Device State** `[Dropdown - Hidden on Add]` (Commercial & deployment state)
   - Field Link Name: `Device_State`
   - Options: `"Rentally Active"`, `"Active in Demo"`, `"Idle with Customer"`, `"Available in Stock"`
   - Form Behavior: Hidden on Add (defaults to "Available in Stock" in background); Visible & Locked in Edit mode; updated during active customer deployment, returns, or stock intake.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/05_devices/form_devices/1_Devices_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable `Serial_Number`, `Product_Model`, `Lot`, and `Current_Office` as interactive fields. Dynamically query `Lots` to auto-select the latest lot created by default and query `Offices` to auto-default `Current_Office` to `"Bangalore Office"`. Keep `Device_Type` disabled to provide clear context without user editing. Hide remaining lifecycle fields (`Current_Customer`, `Device_Stage`, `Device_State`).
  - In EDIT Mode: Lock `Serial_Number` and `Product_Model`. Keep `Lot` enabled so technicians can reassign lots if necessary. Display and lock context and lifecycle fields (`Device_State`, `Device_Type`, `Current_Customer`, `Current_Office`). Dynamically filter `Device_Stage` by clearing static choices via `clear Device_Stage;` and loading hardware-specific stages from the shared standalone function `scripts/functions/get_device_stages.deluge` (`thisapp.get_device_stages(dev_type)`), re-asserting the current saved stage, and enabling `Device_Stage` unless `Device_State` is `"Rentally Active"` or `"Active in Demo"` (locked while actively deployed).

### Workflow 2: On User Input (Step 1: Serial Number)
- **File**: `scripts/05_devices/form_devices/2_1_Devices_onuserinput_serialnumber.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Serial_Number -> On User Input
- **Plain English Logic**:
  - Step 1: If cleared, reset derived fields (`Device_Type`, `Product_Model`, `Device_Stage`, `Device_State`).
  - Step 2: Calls `scripts/functions/parse_device_serial.deluge` (`thisapp.parse_device_serial`) to sanitize `Serial_Number`, validate `CU`/`OV` and `ALT`/`ET` model prefix, and derive `Device_Type` and target `Product_Model` name.
  - Step 3: Duplicate check: Check if `Serial_Number` exists in `Devices`. If found, alert user and clear dependent fields.
  - Step 4: Auto-initialize `Device_Stage` to `"0 - Mfg Not Yet Started"` for both Control Units and Overlays.
  - Step 5: Queries `Product_Models` for the parsed model name to assign `Product_Model` lookup, while allowing user to edit/reassign (e.g. choose `"Eturnal One with foam"`).
  - Step 6: Initialize `Device_State = "Available in Stock"`.

### Workflow 2.2: On Update of Device Stage (Auto-Harmonization Guard)
- **File**: `scripts/05_devices/form_devices/2_2_Devices_onupdate_devicestage.deluge`
- **Trigger**: Edited -> Field Actions -> Device_Stage -> On Update
- **Plain English Logic**:
  - Step 1: Detect hardware classification (`Control Unit` vs `Overlay`).
  - Step 2: Check whether updated `Device_Stage` is a hardware-specific Stage 1 or Stage 2 option (`1 - CU...` / `2 - CU...` vs `1 - OV...` / `2 - OV...`). Shared stages (`0`, `3`, `4`, `5`, `6 - Ready Condition`, `Retesting Needed`, `Repair Needed`, `Disinfection Needed`) pass through without conversion.
  - Step 3: If an Overlay receives a Control Unit Stage 1 or 2 (e.g. during report Bulk Edit), automatically convert it to the counterpart Overlay Stage 1 or 2 (`1 - OV in Screen Printing / Welding`, `2 - OV in Stitching`).
  - Step 4: If a Control Unit receives an Overlay Stage 1 or 2, automatically convert it to the counterpart Control Unit Stage 1 or 2 (`1 - CU Enclosure Mfg in Progress`, `2 - CU Subsystems Assembled`).

### Workflow 3: On Form Validation & Safety Guard
- **File**: `scripts/05_devices/form_devices/3_Devices_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory check: Ensure `Serial_Number` is entered.
  - Step 2: Sanitize `Serial_Number` (trim & uppercase).
  - Step 3: Duplicate check: Query `Devices` to prevent duplicate serial numbers.
  - Step 4: Verify syntax via shared `scripts/functions/parse_device_serial.deluge` (must contain "CU" or "OV", and begin with valid `ALT`/`ET` prefix).
  - Step 5: Safety fallback: Ensure `Device_Type`, `Product_Model` lookup (queries `Product_Models` for matching model ID if unassigned), `Lot` lookup (fallback to latest created lot from `Lots`), initial `Device_Stage` (`"0 - Mfg Not Yet Started"`), and `Device_State` (`"Available in Stock"`) are populated. Guard active Edit mode submissions so `"Rentally Active"` / `"Active in Demo"` devices remain at `"6 - Ready Condition"`.
  - Step 6: Bulk Edit & Compatibility Guard: Prevents assigning Overlay-specific Stages 1–2 to Control Units or Control Unit-specific Stages 1–2 to Overlays.

### Workflow 4: On Form Submission Success
- **File**: `scripts/05_devices/form_devices/4_Devices_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Redirect cleanly to `All_Devices` report view (`openUrl("#Report:All_Devices", "same window");`), returning user to master asset register.

---

## 4. Stateless Bulk Action Utilities & Report Custom Actions

To provide high-speed, user-friendly batch processing without relying on Creator's slow native report bulk edit, two dedicated stateless utility forms are implemented and launched as pre-filled pop-up modals from `All_Devices`.

### Stateless Wizards & Action Buttons Map

| Action Button Label | Report & Placement | Condition | Launcher Function & Namespace | Target Stateless Form | Script Folder |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Bulk Create Devices** *(Sidebar)* | Sidebar Navigation -> Devices / Utilities | Always | *Direct Form Open (no row selection needed)* | `Bulk_Create_Devices` | `scripts/05_devices/stateless_bulk_create/` |
| **Update Stage** | `All_Devices` -> Multiple Records | Always | `open_update_stage(Devices selected_devices)` *(Namespace: `Default` / `devices_actions`)* | `Update_Device_Stage` | `scripts/05_devices/stateless_update_stage/` |
| **Update Custody** | `All_Devices` -> Multiple Records | Always | `open_update_custody(Devices selected_devices)` *(Namespace: `Default` / `devices_actions`)* | `Update_Device_Custody` | `scripts/05_devices/stateless_update_custody/` |
| **Deploy Devices** | `All_Devices` -> Multiple Records | Always | `open_deploy_devices(Devices selected_devices)` *(Namespace: `Default` / `devices_actions`)* | `Deployments` *(pre-filled popup)* | `scripts/05_devices/report_all_devices/` |

### A. Utility 1: Update Device Stage (`Update_Device_Stage`)
- **Form Type**: Stateless Form (No record storage; executes Deluge on submit)
- **Form Link Name**: `Update_Device_Stage`
- **Fields**:
  1. **Select Devices** `[Lookup -> Devices, displaying Serial_Number - Multi-Select]`
     - Field Link Name: `Select_Devices`
     - Behavior: Interactive field populated by user or pre-filled from `All_Devices` report custom action.
  2. **New Device Stage** `[Dropdown]`
     - Field Link Name: `New_Device_Stage`
     - Choices: Dynamically populated on load via `scripts/functions/get_device_stages.deluge` (`thisapp.get_device_stages("All")` — all 12 unified lifecycle choices: 8 shared stages + CU Stages 1–2 + OV Stages 1–2); no manual Form Builder choices required.
- **Automated Harmonization Principle**:
  - By default, if the batch contains mixed hardware types (Control Units and Overlays), shared stages (`0`, `3`, `4`, `5`, `6 - Ready Condition`, `Retesting Needed`, `Repair Needed`, `Disinfection Needed`) apply identically to all devices, while hardware-specific Stages 1–2 automatically map to the counterpart stage for the respective hardware (e.g. `1 - CU Enclosure Mfg in Progress` $\leftrightarrow$ `1 - OV in Screen Printing / Welding`).
- **Workflows**:
  - **On Load**: `scripts/05_devices/stateless_update_stage/1_Update_Device_Stage_onload.deluge`
    - Dynamically filters `Select_Devices` via Deluge to exclude `"Rentally Active"` and `"Active in Demo"` devices, preserves eligible URL pre-fills and alerts if active units were excluded, and populates `New_Device_Stage` dynamically from `scripts/functions/get_device_stages.deluge`.
  - **On Click of Submit**: `scripts/05_devices/stateless_update_stage/5_1_Update_Device_Stage_onclick_submit.deluge` (delegates to `scripts/functions/handle_update_device_stage.deluge`)
    - Thin wrapper invokes `thisapp.handle_update_device_stage(...)` inside `try-catch` error protection (`format_error_log`). Validates selection and stage, fetches target device records in 1 batch query, blocks execution if any selected device is `"Rentally Active"` or `"Active in Demo"`, iterates and updates `Device_Stage` with automatic hardware-stage mapping, alerts only on validation/runtime error, and redirects silently to `All_Devices` on success.

### B. Utility 2: Update Device Custody (`Update_Device_Custody`)
- **Form Type**: Stateless Form (No record storage; executes Deluge on submit)
- **Form Link Name**: `Update_Device_Custody`
- **Fields**:
  1. **Select Devices** `[Lookup -> Devices, displaying Serial_Number - Multi-Select]`
     - Field Link Name: `Select_Devices`
     - Behavior: Interactive field populated by user or pre-filled from `All_Devices` report custom action; dynamically filtered via Deluge to exclude `"Rentally Active"` and `"Active in Demo"` devices.
  2. **Target Office** `[Lookup -> Offices, displaying Office_Name]`
     - Field Link Name: `Target_Office`
     - Behavior: Displayed side-by-side with `Target_Customer` on load; selecting an office auto-clears `Target_Customer`.
  3. **Target Customer** `[Lookup -> Customers, displaying Customer_Name]`
     - Field Link Name: `Target_Customer`
     - Behavior: Displayed side-by-side with `Target_Office` on load; selecting a customer auto-clears `Target_Office`.
- **Workflows**:
  - **On Load**: `scripts/05_devices/stateless_update_custody/1_Update_Device_Custody_onload.deluge`
    - Ensures both `Target_Office` and `Target_Customer` are visible and enabled side-by-side, and dynamically filters `Select_Devices` via Deluge to exclude `"Rentally Active"` and `"Active in Demo"` devices while preserving eligible pre-fills.
  - **On User Input (Target Office)**: `scripts/05_devices/stateless_update_custody/2_1_Update_Device_Custody_onuserinput_target_office.deluge`
    - Auto-clears `Target_Customer` whenever a `Target_Office` is selected.
  - **On User Input (Target Customer)**: `scripts/05_devices/stateless_update_custody/2_2_Update_Device_Custody_onuserinput_target_customer.deluge`
    - Auto-clears `Target_Office` whenever a `Target_Customer` is selected.
  - **On Click of Submit**: `scripts/05_devices/stateless_update_custody/5_1_Update_Device_Custody_onclick_submit.deluge` (delegates to `scripts/functions/handle_update_device_custody.deluge`)
    - Thin wrapper invokes `thisapp.handle_update_device_custody(...)` inside `try-catch` error protection (`format_error_log`). Validates selections, fetches target devices and checks active states (`"Rentally Active"` / `"Active in Demo"`) plus open `Deployments` (`Deployment_State != "Closed"`). Enforces `"6 - Ready Condition"` when `Target_Customer` is selected. For `Target_Office`: updates `Current_Office`, clears `Current_Customer`, and sets `Device_State = "Available in Stock"`. For `Target_Customer`: updates `Current_Customer`, clears `Current_Office`, and sets `Device_State = "Idle with Customer"` (so devices are parked at the customer site until an active `Deployments` record marks them `"Rentally Active"` or `"Active in Demo"`). Alerts only on validation/runtime error, and redirects silently to `All_Devices` on success.

### C. Utility 3: Bulk Create Devices (`Bulk_Create_Devices`)
- **Form Type**: Stateless Form (No record storage; launched from Sidebar navigation; executes Deluge on submit)
- **Form Link Name**: `Bulk_Create_Devices`
- **Fields**:
  1. **Serial Numbers** `[Multi Line]`
     - Field Link Name: `Serial_Numbers`
     - Behavior: Mandatory user entry supporting Excel/Google Sheets pasted columns, spaces, commas, tabs, and semicolons.
  2. **Device Rows (Editable Table)** `[Multi Line]`
     - Field Link Name: `Device_Rows`
     - Behavior: Editable pipe-separated tabular text (`Serial_Number | Product_Model | Lot | Device_Stage | Current_Office`) auto-populated on `Serial_Numbers` input via `scripts/functions/generate_bulk_device_rows.deluge` (defaulting `Lot` to latest Lot, `Device_Stage` to `"0 - Mfg Not Yet Started"`, and `Current_Office` to `"Bangalore Office"`). Users can edit any row's model, lot, stage, office, or serial directly before submitting, and `On User Input` live-refreshes `Reference_Guide`.
  3. **Reference Guide** `[Multi Line]`
     - Field Link Name: `Reference_Guide`
     - Behavior: Disabled read-only context box populated cleanly with bulleted sections for Live Validation Status plus valid `Lots`, `Product_Models`, `Offices`, and grouped `Device_Stage` names (`Shared`, `Control Unit Only`, `Overlay Only`). Updated live on both `Serial_Numbers` and `Device_Rows` user input via `scripts/functions/validate_bulk_device_rows.deluge`.
- **Workflows**:
  - **On Load**: `scripts/05_devices/stateless_bulk_create/1_Bulk_Create_Devices_onload.deluge`
    - Disables `Reference_Guide` and loads the `Device_Rows` header template plus valid master reference values via `generate_bulk_device_rows("")`.
  - **On User Input (Serial Numbers)**: `scripts/05_devices/stateless_bulk_create/2_1_Bulk_Create_Devices_onuserinput_serial_numbers.deluge`
    - Calls `scripts/functions/generate_bulk_device_rows.deluge` to parse tokens, classify hardware & product model via `parse_device_serial`, populate editable `Device_Rows` (with latest `Lot` and `"Bangalore Office"`), and delegate batch validation & formatted `Reference_Guide` output to `validate_bulk_device_rows`.
  - **On User Input (Device Rows)**: `scripts/05_devices/stateless_bulk_create/2_2_Bulk_Create_Devices_onuserinput_device_rows.deluge`
    - Calls `scripts/functions/validate_bulk_device_rows.deluge` to re-validate edited `Device_Rows` in real time and immediately refresh `Reference_Guide` so corrected errors clear and new row mistakes are flagged before submission.
  - **On Click of Submit**: `scripts/05_devices/stateless_bulk_create/5_1_Bulk_Create_Devices_onclick_submit.deluge` (delegates to `scripts/functions/handle_bulk_create_devices.deluge`)
    - Thin wrapper invokes `thisapp.handle_bulk_create_devices(input.Device_Rows, input.Serial_Numbers)` inside `try-catch`. Reuses `validate_bulk_device_rows` for strict per-row validation (with automatic `Serial_Numbers` fallback if submitted before blur), alerts exact row/field fixes needed on any error, inserts all `Devices` records with `Device_State = "Available in Stock"` once 100% pass, and redirects to `All_Devices`.

### D. Report Custom Action Integration on `All_Devices`
- **Action 1**: `Update Stage` -> Validates none of the selected devices are `"Rentally Active"` or `"Active in Demo"`, then opens `Update_Device_Stage` in a pop-up window mapping selected record IDs into `Select_Devices`.
  - **Function File**: `scripts/05_devices/report_all_devices/5_1_All_Devices_customaction_open_update_stage.deluge`
  - **Function Signature**: `void open_update_stage(Devices selected_devices)`
- **Action 2**: `Update Custody` -> Validates none of the selected devices are `"Rentally Active"` or `"Active in Demo"`, then opens `Update_Device_Custody` in a pop-up window mapping selected record IDs into `Select_Devices`.
  - **Function File**: `scripts/05_devices/report_all_devices/5_2_All_Devices_customaction_open_update_custody.deluge`
  - **Function Signature**: `void open_update_custody(Devices selected_devices)`
- **Action 3**: `Deploy Devices` -> Validates 1 or 2 ready devices (at most 1 Control Unit and 1 Overlay, matching `"Idle with Customer"` custody if applicable), then opens `Deployments` in a pre-filled popup window (`Control_Unit`, `Overlay`, `Customer`, and `Return_Report=All_Devices` so submit refreshes `All_Devices`).
  - **Function File**: `scripts/05_devices/report_all_devices/5_3_All_Devices_customaction_open_deploy_devices.deluge`
  - **Function Signature**: `void open_deploy_devices(Devices selected_devices)`
