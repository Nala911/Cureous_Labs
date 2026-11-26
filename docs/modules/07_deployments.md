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
| **Customer** | `Customer` | Lookup | **User Entry on Add / Locked on Edit** (Mandatory, Relational Link to Customer Master) | `Customers` (`Customer_Name`) | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge` | Sets `Devices.Current_Customer` on paired assets |
| **Control Unit** | `Control_Unit` | Lookup | **User Entry on Add / Locked on Edit** (Optional if Overlay provided; Add-Mode Deluge Lookup Filter & Validation: `Device_Type == "Control Unit" && Device_Stage == "6 - Ready Condition" && Device_State == "Available in Stock"`; swaps go through `Swap_Devices`) | `Devices` (`Serial_Number`) | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/2_1_Deployments_onuserinput_control_unit.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge` | Direct asset allocation and readiness/state tracking |
| **Overlay** | `Overlay` | Lookup | **User Entry on Add / Locked on Edit** (Optional if Control Unit provided; Add-Mode Deluge Lookup Filter & Validation: `Device_Type == "Overlay" && Device_Stage == "6 - Ready Condition" && Device_State == "Available in Stock"`; swaps go through `Swap_Devices`) | `Devices` (`Serial_Number`) | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/2_2_Deployments_onuserinput_overlay.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge` | Direct asset allocation and readiness/state tracking |
| **Start Date** | `Start_Date` | Date | **User Entry** (Mandatory, Defaults to `zoho.currentdate`) | — | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge` | Commercial billing start anchor |
| **End Date** | `End_Date` | Date | **Hidden on Add** (Revealed, enabled, and mandatory when `Deployment_State == "Closed"`; must be `>= Start_Date`) | — | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/2_3_Deployments_onuserinput_deployment_state.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge` | Commercial billing conclusion anchor |
| **Deployment State** | `Deployment_State` | Dropdown | **User Entry** (Mandatory; Add Mode choices: `"Rental"`, `"Demo"` defaulting to `"Rental"`; Edit Mode choices: `"Rental"`, `"Demo"`, `"Idle"`; closures flow through `Close_Deployments`) | — | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/2_3_Deployments_onuserinput_deployment_state.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge` | Triggers automated updates to `Devices.Device_State` |
| **Deployment Summary** | `Deployment_Summary` | Single Line | **Hidden on Add & Edit** (System-populated compact label: `<First 2 Words of Customer> \| <CU Serial> \| <OV Serial>`) | — | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/stateless_swap_devices/5_1_Swap_Devices_onclick_submit.deluge` | Human-readable Lookup Display Field for `Close_Deployments` and `Swap_Devices` |

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable user input fields (`Customer`, `Control_Unit`, `Overlay`, `Start_Date`, `Deployment_State`) and hide `End_Date`. Default `Start_Date` to `zoho.currentdate`. Clear `Deployment_State`, dynamically populate only `"Rental"` and `"Demo"`, and default to `"Rental"`. Dynamically filter `Control_Unit` and `Overlay` lookups to only include devices matching `Device_Type` + `Device_Stage == "6 - Ready Condition"` + `Device_State == "Available in Stock"` (ignoring `Current_Customer` and `Current_Office`).
  - In EDIT Mode: If the record is already marked `"Closed"`, reveal `End_Date` and lock all deployment fields (`Customer`, `Control_Unit`, `Overlay`, `Start_Date`, `End_Date`, `Deployment_State`) to prevent accidental modification of closed audit logs. Otherwise, lock `Customer`, `Control_Unit`, and `Overlay` (so customer reassignments and device swaps flow through `Close_Deployments` / `Swap_Devices`), populate active lifecycle choices (`"Rental"`, `"Demo"`, `"Idle"`) without `"Closed"`, preserve the current selection, and hide `End_Date` unless already populated.

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
  - Step 7: Summary Label generation: Fetch `Customers` to extract the first 2 words of `Customer_Name` and format `Deployment_Summary` as `<First 2 Words> | <CU Serial> | <OV Serial>` (omitting `null` devices).

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
  1. **Deployments** `[Lookup -> Deployments, displaying Deployment_Summary - Multi-Select]`
     - Field Link Name: `Deployments`
     - Behavior: Mandatory; pre-filled from `All_Deployments` report custom actions (`?Deployments=...`) or manually selected standalone; Deluge Lookup Filter: `Deployment_State != "Closed"`.
  2. **End Date** `[Date]`
     - Field Link Name: `End_Date`
     - Behavior: Mandatory; defaults to `zoho.currentdate` on load; must be `>= Start_Date` of all selected deployments.
  3. **Collected?** `[Decision Box]`
     - Field Link Name: `Collected`
     - Behavior: Defaults to `false`. Controls whether `Return_Office` is shown and whether devices move to `"Available in Stock"` at `Return_Office` or remain `"Idle with Customer"`.
  4. **Return Office** `[Lookup -> Offices, displaying Office_Name]`
     - Field Link Name: `Return_Office`
     - Behavior: Hidden by default; dynamically displayed and mandatory when `Collected == true`.
  5. **Device Condition** `[Dropdown]`
     - Field Link Name: `Device_Condition`
     - Behavior: Always visible, mandatory, defaults to `"Disinfection Needed"` on load; Choices: `"Disinfection Needed"`, `"Retesting Needed"`, `"Repair Needed"`, `"6 - Ready Condition"`.
- **Workflows**:
  - **On Load**: `scripts/07_deployments/stateless_close_deployments/1_Close_Deployments_onload.deluge`
    - Dynamically filters `Deployments` via Deluge to active records (`Deployment_State != "Closed"`), preserves eligible URL pre-fills (alerting if any closed IDs were excluded), defaults `End_Date` to `zoho.currentdate`, defaults `Device_Condition` to `"Disinfection Needed"` (always shown/enabled), and hides `Return_Office` unless `Collected == true`.
  - **On User Input (Collected?)**: `scripts/07_deployments/stateless_close_deployments/2_1_Close_Deployments_onuserinput_collected.deluge`
    - Shows and enables `Return_Office` when `Collected == true`; hides and clears `Return_Office` when `Collected == false`.
  - **On Click of Submit**: `scripts/07_deployments/stateless_close_deployments/5_1_Close_Deployments_onclick_submit.deluge`
    - Validates inputs (`Deployments`, `End_Date`, `Device_Condition`, and `Return_Office` when `Collected == true`), fetches selected deployments in 1 batch query, verifies `End_Date >= Start_Date`, marks each active deployment `"Closed"` with `End_Date`, collects all linked `Control_Unit` and `Overlay` IDs, and fetches affected `Devices` in 1 batch query. Sets `Device_Stage = input.Device_Condition` on all affected devices; if `Collected == true`, sets `Device_State = "Available in Stock"`, `Current_Customer = null`, and `Current_Office = input.Return_Office`; if `Collected == false`, sets `Device_State = "Idle with Customer"`, `Current_Office = null`, and preserves `Current_Customer`. Alerts count and refreshes `All_Deployments`.

### Report Custom Action Integration on `All_Deployments`
- **Action 1 (Multiple Records)**: `Close Deployments` -> `scripts/07_deployments/report_all_deployments/5_1_All_Deployments_customaction_open_close_deployments.deluge` (`void open_close_deployments(Deployments selected_deployments)` — filters selected rows to `Deployment_State != "Closed"`, blocks if zero active rows remain)
- **Action 2 (Multiple Records)**: `Swap Devices` -> `scripts/07_deployments/report_all_deployments/5_2_All_Deployments_customaction_open_swap_devices.deluge` (`void open_swap_devices(Deployments selected_deployments)` — validates single active row)

---

### Utility: Swap Devices (`Swap_Devices`)
- **Form Type**: Stateless Form (No record storage; executes Deluge on submit)
- **Form Link Name**: `Swap_Devices`
- **Fields**:
  1. **Deployment** `[Lookup -> Deployments, displaying Deployment_Summary - Single-Select]`
     - Field Link Name: `Deployment`
     - Behavior: Mandatory; pre-filled from `All_Deployments` report custom action (`?Deployment=...`) or manually selected standalone; Deluge Lookup Filter: `Deployment_State != "Closed"`.
  2. **Control Unit** `[Lookup -> Devices, displaying Serial_Number]`
     - Field Link Name: `Control_Unit`
     - Behavior: Disabled until `Deployment` is selected; auto-populated from `Deployment`; user-editable (can be swapped, added, or cleared if `Overlay` remains selected); Deluge Filter: `[current unit] + [Device_Type == "Control Unit" && Device_Stage == "6 - Ready Condition" && Device_State == "Available in Stock"]`.
  3. **Overlay** `[Lookup -> Devices, displaying Serial_Number]`
     - Field Link Name: `Overlay`
     - Behavior: Disabled until `Deployment` is selected; auto-populated from `Deployment`; user-editable (can be swapped, added, or cleared if `Control_Unit` remains selected); Deluge Filter: `[current unit] + [Device_Type == "Overlay" && Device_Stage == "6 - Ready Condition" && Device_State == "Available in Stock"]`.
  4. **Date** `[Date]`
     - Field Link Name: `Swap_Date`
     - Behavior: Mandatory; defaults to `zoho.currentdate` on load; must be `>= Start_Date` of the selected deployment.
  5. **Collected?** `[Decision Box]`
     - Field Link Name: `Collected`
     - Behavior: Hidden until an existing device is swapped out or removed; defaults to `false`. Controls visibility of `Return_Office` and whether swapped-out devices move to `"Available in Stock"` at `Return_Office` or remain `"Idle with Customer"`.
  6. **Return Office** `[Lookup -> Offices, displaying Office_Name]`
     - Field Link Name: `Return_Office`
     - Behavior: Hidden by default; dynamically displayed and mandatory when a device is swapped out and `Collected == true`.
  7. **Device Condition** `[Dropdown]`
     - Field Link Name: `Device_Condition`
     - Behavior: Hidden until an existing device is swapped out or removed; mandatory when swapping out a device; defaults to `"Disinfection Needed"`; Choices: `"Disinfection Needed"`, `"Retesting Needed"`, `"Repair Needed"`, `"6 - Ready Condition"`.
- **Workflows**:
  - **On Load**: `scripts/07_deployments/stateless_swap_devices/1_Swap_Devices_onload.deluge`
    - Dynamically filters `Deployment` to non-closed records, defaults `Swap_Date` to `zoho.currentdate`, keeps `Collected`, `Return_Office`, and `Device_Condition` hidden until a swap-out occurs, disables `Control_Unit` and `Overlay` until a `Deployment` is selected, and when pre-filled via URL, populates and filters `Control_Unit` and `Overlay` to `[current unit] + [Ready & Available in Stock]` (guarding against `"Closed"` records).
  - **On User Input (Deployment)**: `scripts/07_deployments/stateless_swap_devices/2_1_Swap_Devices_onuserinput_deployment.deluge`
    - Resets return-tracking visibility (`Collected`, `Return_Office`, `Device_Condition` hidden), enables & dynamically filters `Control_Unit` and `Overlay` (`[current unit] + [Ready & Available in Stock]`), and populates current devices, or clears/disables both when cleared.
  - **On User Input (Control_Unit)**: `scripts/07_deployments/stateless_swap_devices/2_2_Swap_Devices_onuserinput_control_unit.deluge`
    - Validates `Device_Type == "Control Unit"`. If differing from the source deployment's unit, enforces `"6 - Ready Condition"`, `"Available in Stock"`, and no active open deployment conflict (`ID != input.Deployment`), reverting invalid selections cleanly to the retained unit. Reveals `Collected?` and `Device_Condition` only when an existing unit is replaced or removed.
  - **On User Input (Overlay)**: `scripts/07_deployments/stateless_swap_devices/2_3_Swap_Devices_onuserinput_overlay.deluge`
    - Validates `Device_Type == "Overlay"`. If differing from the source deployment's unit, enforces `"6 - Ready Condition"`, `"Available in Stock"`, and no active open deployment conflict (`ID != input.Deployment`), reverting invalid selections cleanly to the retained unit. Reveals `Collected?` and `Device_Condition` only when an existing unit is replaced or removed.
  - **On User Input (Collected?)**: `scripts/07_deployments/stateless_swap_devices/2_4_Swap_Devices_onuserinput_collected.deluge`
    - Shows and enables `Return_Office` when `Collected == true`; hides and clears `Return_Office` when `Collected == false`.
  - **On Click of Submit**: `scripts/07_deployments/stateless_swap_devices/5_1_Swap_Devices_onclick_submit.deluge`
    - Validates mandatory inputs (`Deployment`, at least one of `Control_Unit`/`Overlay`, `Swap_Date`), ensures a device change/addition/removal occurred, enforces `Device_Condition` (and `Return_Office` when `Collected == true`) only if an existing device is being swapped out, validates `Swap_Date >= Start_Date`, and validates replacement readiness (`"6 - Ready Condition"` & `"Available in Stock"`) plus no open deployment conflict. Closes the old deployment (`Deployment_State = "Closed"`, `End_Date = Swap_Date`), builds `Deployment_Summary`, inserts a new `Deployments` record inheriting `Customer` and `Deployment_State` with `Start_Date = Swap_Date`, and batch-updates `Devices`: swapped-out devices receive `Device_Stage = input.Device_Condition` and either `"Available in Stock"` / `Current_Customer = null` / `Current_Office = Return_Office` (when `Collected == true`) or `"Idle with Customer"` / `Current_Office = null` with `Current_Customer` preserved (when `Collected == false`), while new and retained devices are set to the active `Device_State`, `Current_Customer`, and `Current_Office = null`. Alerts confirmation and refreshes `All_Deployments`.
