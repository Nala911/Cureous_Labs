# MODULE 07: DEPLOYMENTS (BEDSIDE PAIRING & FIELD DEPLOYMENT LEDGER)

**Form Name**: `Deployments`  
**Form Link Name**: `Deployments`  
**Default Report**: `All_Deployments`  
**Purpose**: Central operational ledger for pairing Control Units and Overlays at the patient bedside and tracking customer deployments across their lifecycle (Rental, Demo, Idle, Closed). Automatically synchronizes real-time device states and customer custody in the `Devices` module.

---

## 1. Core Operational Principles

- **Bedside Dynamic Pairing**: Control Units and Overlays are manufactured and inventoried independently. They are dynamically paired at the customer bedside during field deployment via this module.
- **Component Swap Flexibility**: A deployment supports deploying both a Control Unit and an Overlay, or a single component independently (e.g. replacing a faulty pump or soiled mattress without interrupting the patient's existing setup).
- **Automated Lifecycle Synchronization**: Whenever a deployment is created or updated, the system automatically synchronizes the physical asset status in the `Devices` master:
  - **Rental**: Device State becomes `"Rentally Active"`, `Current_Customer` is assigned, and `Current_Office` is cleared (`null`).
  - **Demo**: Device State becomes `"Active in Demo"`, `Current_Customer` is assigned, and `Current_Office` is cleared (`null`).
  - **Idle**: Device State becomes `"Idle with Customer"`, `Current_Customer` is assigned, and `Current_Office` is cleared (`null`).
  - **Closed**: Device State reverts to `"Available in Stock"`, `Current_Customer` is unassigned (`null`).
- **Conflict Prevention**: Validates that neither the selected Control Unit nor Overlay is currently actively deployed with another customer.

---

## 2. Field Specifications

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Customer** | `Customer` | Lookup | **User Entry** (Mandatory, Relational Link to Customer Master) | `Customers` (`Customer_Name`) | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge` | Sets `Devices.Current_Customer` on paired assets |
| **Control Unit** | `Control_Unit` | Lookup | **User Entry on Add / Locked on Edit** (Optional if Overlay provided; Form Builder Lookup Filter: `Device_Type == "Control Unit"`; Add Mode dynamically filtered & validated to `Device_Stage == "6 - Ready Condition" && Device_State == "Available in Stock"`; swaps go through `Swap_Devices`) | `Devices` (`Serial_Number`) | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/2_1_Deployments_onuserinput_control_unit.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge` | Direct asset allocation and readiness/state tracking |
| **Overlay** | `Overlay` | Lookup | **User Entry on Add / Locked on Edit** (Optional if Control Unit provided; Form Builder Lookup Filter: `Device_Type == "Overlay"`; Add Mode dynamically filtered & validated to `Device_Stage == "6 - Ready Condition" && Device_State == "Available in Stock"`; swaps go through `Swap_Devices`) | `Devices` (`Serial_Number`) | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/2_2_Deployments_onuserinput_overlay.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge` | Direct asset allocation and readiness/state tracking |
| **Start Date** | `Start_Date` | Date | **User Entry** (Mandatory, Defaults to `zoho.currentdate`) | — | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge` | Commercial billing start anchor |
| **End Date** | `End_Date` | Date | **Hidden on Add** (Revealed, enabled, and mandatory when `Deployment_State == "Closed"`; must be `>= Start_Date`) | — | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/2_3_Deployments_onuserinput_deployment_state.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge` | Commercial billing conclusion anchor |
| **Deployment State** | `Deployment_State` | Dropdown | **User Entry** (Mandatory; Add Mode choices: `"Rental"`, `"Demo"` defaulting to `"Rental"`; Edit Mode choices: `"Rental"`, `"Demo"`, `"Idle"`; closures flow through `Close_Deployments`) | — | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/2_3_Deployments_onuserinput_deployment_state.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge` | Triggers automated updates to `Devices.Device_State` |

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable user input fields (`Customer`, `Control_Unit`, `Overlay`, `Start_Date`, `Deployment_State`) and hide `End_Date`. Default `Start_Date` to `zoho.currentdate`. Clear `Deployment_State`, dynamically populate only `"Rental"` and `"Demo"`, and default to `"Rental"`. Dynamically filter `Control_Unit` and `Overlay` lookups to only include devices matching `Device_Type` + `Device_Stage == "6 - Ready Condition"` + `Device_State == "Available in Stock"` (ignoring `Current_Customer` and `Current_Office`).
  - In EDIT Mode: If the record is already marked `"Closed"`, reveal `End_Date` and lock all deployment fields (`Customer`, `Control_Unit`, `Overlay`, `Start_Date`, `End_Date`, `Deployment_State`) to prevent accidental modification of closed audit logs. Otherwise, lock `Control_Unit` and `Overlay` (so device swaps flow through `Swap_Devices`), populate active lifecycle choices (`"Rental"`, `"Demo"`, `"Idle"`) without `"Closed"` (so closures flow through `Close_Deployments`), preserve the current selection, and hide `End_Date` unless already populated.

### Workflow 2.1: On User Input (Control Unit)
- **File**: `scripts/07_deployments/form_deployments/2_1_Deployments_onuserinput_control_unit.deluge`
- **Trigger**: Created or Edited -> Field Actions -> `Control_Unit` -> On User Input
- **Plain English Logic**:
  - Step 1: Check if a Control Unit is selected. If cleared, do nothing.
  - Step 2: Query the `Devices` record using pre-assigned ID variable to verify hardware type is `"Control Unit"`. If mismatched, alert user and clear field.
  - Step 3: In Add mode, verify `Device_Stage == "6 - Ready Condition"` and `Device_State == "Available in Stock"`; if not, alert user with the exact blocking stage/state and clear the selection.

### Workflow 2.2: On User Input (Overlay)
- **File**: `scripts/07_deployments/form_deployments/2_2_Deployments_onuserinput_overlay.deluge`
- **Trigger**: Created or Edited -> Field Actions -> `Overlay` -> On User Input
- **Plain English Logic**:
  - Step 1: Check if an Overlay is selected. If cleared, do nothing.
  - Step 2: Query the `Devices` record using pre-assigned ID variable to verify hardware type is `"Overlay"`. If mismatched, alert user and clear field.
  - Step 3: In Add mode, verify `Device_Stage == "6 - Ready Condition"` and `Device_State == "Available in Stock"`; if not, alert user with the exact blocking stage/state and clear the selection.

### Workflow 2.3: On User Input (Deployment State)
- **File**: `scripts/07_deployments/form_deployments/2_3_Deployments_onuserinput_deployment_state.deluge`
- **Trigger**: Created or Edited -> Field Actions -> `Deployment_State` -> On User Input
- **Plain English Logic**:
  - Step 1: If user selects `"Closed"`, show and enable `End_Date`, and auto-populate it with `zoho.currentdate` if empty.
  - Step 2: If user switches back to an active state (`"Rental"`, `"Demo"`, `"Idle"`), hide `End_Date` and clear it if it was auto-set to today.

### Workflow 3: On Form Validate
- **File**: `scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory checks: Assert `Customer`, `Start_Date`, and `Deployment_State` are provided.
  - Step 2: System Pairing guard: Assert at least one physical component (`Control_Unit` or `Overlay`) is selected.
  - Step 3: Date validation: If `End_Date` is provided, assert `End_Date >= Start_Date`.
  - Step 4: Closure validation: If `Deployment_State == "Closed"`, assert `End_Date` is not empty.
  - Step 5: Hardware readiness check (Add Mode): Assert selected `Control_Unit` and `Overlay` have `Device_Stage == "6 - Ready Condition"` and `Device_State == "Available in Stock"`.
  - Step 6: Double-Booking guard: If `Deployment_State != "Closed"`, query active deployments to confirm neither the selected Control Unit nor Overlay is already deployed in another ongoing deployment.

### Workflow 4: On Form Submission Success
- **File**: `scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Map `Deployment_State` to target `Device_State`, `Current_Customer`, and whether to clear `Current_Office`:
    - `"Rental"` -> `new_state = "Rentally Active"`, `target_cust = input.Customer`, `clear_office = true`
    - `"Demo"` -> `new_state = "Active in Demo"`, `target_cust = input.Customer`, `clear_office = true`
    - `"Idle"` -> `new_state = "Idle with Customer"`, `target_cust = input.Customer`, `clear_office = true`
    - `"Closed"` -> `new_state = "Available in Stock"`, `target_cust = null`, `clear_office = false`
  - Step 2: If `Control_Unit` is selected, fetch the target device record by ID, update its `Device_State`, `Current_Customer`, and clear `Current_Office` (`null`) if active with customer.
  - Step 3: If `Overlay` is selected, fetch the target device record by ID, update its `Device_State`, `Current_Customer`, and clear `Current_Office` (`null`) if active with customer.
  - Step 4: Redirect cleanly to `#Report:All_Deployments`.


---

## 4. Stateless Action Utilities & Report Custom Actions

To enable fast, interactive operational workflows on active deployments without opening or manually cloning individual records, dedicated stateless wizards (`Close_Deployments` and `Swap_Devices`) are implemented and launched as pre-filled pop-up modals from `All_Deployments` toolbar actions (or opened directly standalone).

### Stateless Wizards & Action Buttons Map

| Action Button Label | Report & Placement | Condition | Launcher Function & Namespace | Target Stateless Form | Script Folder |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Close Deployments** | `All_Deployments` -> Multiple Records | Always | `open_close_deployments(Deployments selected_deployments)` *(Namespace: `deployments_actions`)* | `Close_Deployments` | `scripts/07_deployments/stateless_close_deployments/` |
| **Swap Devices** | `All_Deployments` -> Multiple Records | Always *(Single active row validated in script)* | `open_swap_devices(Deployments selected_deployments)` *(Namespace: `deployments_actions`)* | `Swap_Devices` | `scripts/07_deployments/stateless_swap_devices/` |

### Utility: Close Deployments (`Close_Deployments`)
- **Form Type**: Stateless Form (No record storage; executes Deluge on submit)
- **Form Link Name**: `Close_Deployments`
- **Fields**:
  1. **Deployments** `[Lookup -> Deployments - Multi-Select]`
     - Field Link Name: `Deployments`
     - Behavior: Mandatory; pre-filled from `All_Deployments` report custom actions (`?Deployments=...`) or manually selected standalone; Form Builder Filter: `Deployment_State != "Closed"`.
  2. **End Date** `[Date]`
     - Field Link Name: `End_Date`
     - Behavior: Mandatory; defaults to `zoho.currentdate` on load; must be `>= Start_Date` of all selected deployments.
  3. **Collected?** `[Decision Box]`
     - Field Link Name: `Collected`
     - Behavior: Defaults to `false`. Controls whether `Return_Office` is shown and whether devices move to `"Available in Stock"` at `Return_Office` or remain `"Idle with Customer"`.
  4. **Return Office** `[Lookup -> Offices, displaying Office_Name]`
     - Field Link Name: `Return_Office`
     - Behavior: Hidden by default; dynamically displayed and mandatory when `Collected == true`.
  5. **Condition** `[Dropdown]`
     - Field Link Name: `Condition`
     - Behavior: Always visible, mandatory, defaults to `"Disinfection Needed"` on load; Choices: `"Disinfection Needed"`, `"Retesting Needed"`, `"Repair Needed"`, `"6 - Ready Condition"`.
- **Workflows**:
  - **On Load**: `scripts/07_deployments/stateless_close_deployments/1_Close_Deployments_onload.deluge`
    - Defaults `End_Date` to `zoho.currentdate`, defaults `Condition` to `"Disinfection Needed"` (always shown/enabled), and hides `Return_Office` unless `Collected == true`.
  - **On User Input (Collected?)**: `scripts/07_deployments/stateless_close_deployments/2_1_Close_Deployments_onuserinput_collected.deluge`
    - Shows and enables `Return_Office` when `Collected == true`; hides and clears `Return_Office` when `Collected == false`.
  - **On Click of Submit**: `scripts/07_deployments/stateless_close_deployments/5_1_Close_Deployments_onclick_submit.deluge`
    - Validates inputs (`Deployments`, `End_Date`, `Condition`, and `Return_Office` when `Collected == true`), fetches selected deployments in 1 batch query, verifies `End_Date >= Start_Date`, marks each active deployment `"Closed"` with `End_Date`, collects all linked `Control_Unit` and `Overlay` IDs, and fetches affected `Devices` in 1 batch query. Sets `Device_Stage = input.Condition` on all affected devices; if `Collected == true`, sets `Device_State = "Available in Stock"`, `Current_Customer = null`, and `Current_Office = input.Return_Office`; if `Collected == false`, sets `Device_State = "Idle with Customer"`, `Current_Office = null`, and preserves `Current_Customer`. Alerts count and refreshes `All_Deployments`.

### Report Custom Action Integration on `All_Deployments`
- **Action 1 (Multiple Records)**: `Close Deployments` -> `scripts/07_deployments/report_all_deployments/5_1_All_Deployments_customaction_open_close_deployments.deluge` (`void open_close_deployments(Deployments selected_deployments)`)
- **Action 2 (Multiple Records)**: `Swap Devices` -> `scripts/07_deployments/report_all_deployments/5_2_All_Deployments_customaction_open_swap_devices.deluge` (`void open_swap_devices(Deployments selected_deployments)`)

---

### Utility: Swap Devices (`Swap_Devices`)
- **Form Type**: Stateless Form (No record storage; executes Deluge on submit)
- **Form Link Name**: `Swap_Devices`
- **Fields**:
  1. **Deployment** `[Lookup -> Deployments - Single-Select]`
     - Field Link Name: `Deployment`
     - Behavior: Mandatory; pre-filled from `All_Deployments` report custom action (`?Deployment=...`) or manually selected standalone; Form Builder Filter: `Deployment_State != "Closed"`.
  2. **Control Unit** `[Lookup -> Devices, displaying Serial_Number]`
     - Field Link Name: `Control_Unit`
     - Behavior: Auto-populated from the selected `Deployment` on load/selection; user-editable (can be swapped or cleared if `Overlay` remains selected); Form Builder Filter: `Device_Type == "Control Unit"`.
  3. **Overlay** `[Lookup -> Devices, displaying Serial_Number]`
     - Field Link Name: `Overlay`
     - Behavior: Auto-populated from the selected `Deployment` on load/selection; user-editable (can be swapped or cleared if `Control_Unit` remains selected); Form Builder Filter: `Device_Type == "Overlay"`.
  4. **Date** `[Date]`
     - Field Link Name: `Swap_Date`
     - Behavior: Mandatory; defaults to `zoho.currentdate` on load; must be `>= Start_Date` of the selected deployment.
  5. **Collected?** `[Decision Box]`
     - Field Link Name: `Collected`
     - Behavior: Defaults to `false`. Controls whether `Return_Office` is shown and whether swapped-out devices move to `"Available in Stock"` at `Return_Office` or remain `"Idle with Customer"`.
  6. **Return Office** `[Lookup -> Offices, displaying Office_Name]`
     - Field Link Name: `Return_Office`
     - Behavior: Hidden by default; dynamically displayed and mandatory when `Collected == true`.
  7. **Condition** `[Dropdown]`
     - Field Link Name: `Condition`
     - Behavior: Always visible, mandatory, defaults to `"Disinfection Needed"` on load; Choices: `"Disinfection Needed"`, `"Retesting Needed"`, `"Repair Needed"`, `"6 - Ready Condition"`.
- **Workflows**:
  - **On Load**: `scripts/07_deployments/stateless_swap_devices/1_Swap_Devices_onload.deluge`
    - Defaults `Swap_Date` to `zoho.currentdate`, defaults `Condition` to `"Disinfection Needed"` (always shown/enabled), hides `Return_Office` unless `Collected == true`, and if `Deployment` is pre-filled via URL parameter, fetches the record and populates `Control_Unit` and `Overlay` (guarding against `"Closed"` records).
  - **On User Input (Deployment)**: `scripts/07_deployments/stateless_swap_devices/2_1_Swap_Devices_onuserinput_deployment.deluge`
    - Populates `Control_Unit` and `Overlay` from the selected deployment, or clears both when cleared (guarding against `"Closed"` records).
  - **On User Input (Control_Unit)**: `scripts/07_deployments/stateless_swap_devices/2_2_Swap_Devices_onuserinput_control_unit.deluge`
    - Validates `Device_Type == "Control Unit"`. If the unit differs from the selected deployment's current Control Unit, enforces `"6 - Ready Condition"`, `"Available in Stock"`, and no active open deployment conflict (`ID != input.Deployment`).
  - **On User Input (Overlay)**: `scripts/07_deployments/stateless_swap_devices/2_3_Swap_Devices_onuserinput_overlay.deluge`
    - Validates `Device_Type == "Overlay"`. If the unit differs from the selected deployment's current Overlay, enforces `"6 - Ready Condition"`, `"Available in Stock"`, and no active open deployment conflict (`ID != input.Deployment`).
  - **On User Input (Collected?)**: `scripts/07_deployments/stateless_swap_devices/2_4_Swap_Devices_onuserinput_collected.deluge`
    - Shows and enables `Return_Office` when `Collected == true`; hides and clears `Return_Office` when `Collected == false`.
  - **On Click of Submit**: `scripts/07_deployments/stateless_swap_devices/5_1_Swap_Devices_onclick_submit.deluge`
    - Validates mandatory inputs (`Deployment`, at least one of `Control_Unit`/`Overlay`, `Swap_Date`, `Condition`, and `Return_Office` when `Collected == true`), ensures at least one device changed, validates `Swap_Date >= Start_Date`, and validates replacement readiness (`"6 - Ready Condition"` & `"Available in Stock"`) plus no open deployment conflict. Closes the old deployment (`Deployment_State = "Closed"`, `End_Date = Swap_Date`), inserts a new `Deployments` record inheriting `Customer` and `Deployment_State` with `Start_Date = Swap_Date`, and batch-updates `Devices`: swapped-out devices receive `Device_Stage = input.Condition` and either `"Available in Stock"` / `Current_Customer = null` / `Current_Office = Return_Office` (when `Collected == true`) or `"Idle with Customer"` / `Current_Office = null` with `Current_Customer` preserved (when `Collected == false`), while new and retained devices are set to the active `Device_State`, `Current_Customer`, and `Current_Office = null`. Alerts confirmation and refreshes `All_Deployments`.
