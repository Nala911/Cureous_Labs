# MODULE 07: DEPLOYMENT LOGS (BEDSIDE PAIRING & FIELD DEPLOYMENT LEDGER)

**Form Name**: `Deployment Logs`  
**Form Link Name**: `Deployment_Logs`  
**Default Report**: `All_Deployment_Logs`  
**Purpose**: Central operational ledger for pairing Control Units and Overlays at the patient bedside and tracking customer deployments across their lifecycle (Rental, Demo, Idle, Returned). Automatically synchronizes real-time device states and customer custody in the `Devices` module.

---

## 1. Core Operational Principles

- **Bedside Dynamic Pairing**: Control Units and Overlays are manufactured and inventoried independently. They are dynamically paired at the customer bedside during field deployment via this module.
- **Component Swap Flexibility**: A deployment log supports deploying both a Control Unit and an Overlay, or a single component independently (e.g. replacing a faulty pump or soiled mattress without interrupting the patient's existing setup).
- **Automated Lifecycle Synchronization**: Whenever a deployment log is created or updated, the system automatically synchronizes the physical asset status in the `Devices` master:
  - **Rental**: Device State becomes `"Rentally Active"`, Current Customer is assigned.
  - **Demo**: Device State becomes `"Active in Demo"`, Current Customer is assigned.
  - **Idle**: Device State becomes `"Idle with Customer"`, Current Customer is assigned.
  - **Returned**: Device State reverts to `"Available in Stock"`, Current Customer is unassigned (`null`).
- **Conflict Prevention**: Validates that neither the selected Control Unit nor Overlay is currently actively deployed with another customer.

---

## 2. Field Specifications

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Customer** | `Customer` | Lookup | **User Entry** (Mandatory, Relational Link to Customer Master) | `Customers` (`Customer_Name`) | `scripts/07_deployment_logs/1_Deployment_Logs_onload.deluge`<br>`scripts/07_deployment_logs/3_Deployment_Logs_onvalidate.deluge`<br>`scripts/07_deployment_logs/4_Deployment_Logs_onsuccess.deluge` | Sets `Devices.Current_Customer` on paired assets |
| **Control Unit** | `Control_Unit` | Lookup | **User Entry** (Optional if Overlay provided; Filtered to `Device_Type == "Control Unit"`) | `Devices` (`Serial_Number`) | `scripts/07_deployment_logs/1_Deployment_Logs_onload.deluge`<br>`scripts/07_deployment_logs/2_1_Deployment_Logs_onuserinput_control_unit.deluge`<br>`scripts/07_deployment_logs/3_Deployment_Logs_onvalidate.deluge`<br>`scripts/07_deployment_logs/4_Deployment_Logs_onsuccess.deluge` | Direct asset allocation and state tracking |
| **Overlay** | `Overlay` | Lookup | **User Entry** (Optional if Control Unit provided; Filtered to `Device_Type == "Overlay"`) | `Devices` (`Serial_Number`) | `scripts/07_deployment_logs/1_Deployment_Logs_onload.deluge`<br>`scripts/07_deployment_logs/2_2_Deployment_Logs_onuserinput_overlay.deluge`<br>`scripts/07_deployment_logs/3_Deployment_Logs_onvalidate.deluge`<br>`scripts/07_deployment_logs/4_Deployment_Logs_onsuccess.deluge` | Direct asset allocation and state tracking |
| **Start Date** | `Start_Date` | Date | **User Entry** (Mandatory, Defaults to current date) | — | `scripts/07_deployment_logs/1_Deployment_Logs_onload.deluge`<br>`scripts/07_deployment_logs/3_Deployment_Logs_onvalidate.deluge` | Commercial billing start anchor |
| **End Date** | `End_Date` | Date | **User Entry** (Optional on Add, Mandatory when State is Returned; must be `>= Start_Date`) | — | `scripts/07_deployment_logs/2_3_Deployment_Logs_onuserinput_state.deluge`<br>`scripts/07_deployment_logs/3_Deployment_Logs_onvalidate.deluge`<br>`scripts/07_deployment_logs/4_Deployment_Logs_onsuccess.deluge` | Commercial billing conclusion anchor |
| **State** | `State` | Dropdown | **User Entry** (Mandatory; Choices: `"Rental"`, `"Demo"`, `"Idle"`, `"Returned"`; Defaults to `"Rental"`) | — | `scripts/07_deployment_logs/1_Deployment_Logs_onload.deluge`<br>`scripts/07_deployment_logs/2_3_Deployment_Logs_onuserinput_state.deluge`<br>`scripts/07_deployment_logs/3_Deployment_Logs_onvalidate.deluge`<br>`scripts/07_deployment_logs/4_Deployment_Logs_onsuccess.deluge` | Triggers automated updates to `Devices.Device_State` |

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/07_deployment_logs/1_Deployment_Logs_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable all user input fields. Default `Start_Date` to `zoho.currentdate` and `State` to `"Rental"`. Keep `End_Date` empty.
  - In EDIT Mode: If the log is already marked `"Returned"`, lock historical deployment fields (`Start_Date`, `Customer`, `Control_Unit`, `Overlay`) to prevent accidental modification of closed audit logs while leaving notes or administrative fields accessible if needed.

### Workflow 2.1: On User Input (Control Unit)
- **File**: `scripts/07_deployment_logs/2_1_Deployment_Logs_onuserinput_control_unit.deluge`
- **Trigger**: Created or Edited -> Field Actions -> `Control_Unit` -> On User Input
- **Plain English Logic**:
  - Step 1: Check if a Control Unit is selected. If cleared, do nothing.
  - Step 2: Query the `Devices` record using pre-assigned ID variable to verify hardware type is `"Control Unit"`. If mismatched, alert user and clear field.
  - Step 3: In Add mode, check if the device's current `Device_State` is already `"Rentally Active"`, `"Active in Demo"`, or `"Idle with Customer"`. If active elsewhere, warn the user.

### Workflow 2.2: On User Input (Overlay)
- **File**: `scripts/07_deployment_logs/2_2_Deployment_Logs_onuserinput_overlay.deluge`
- **Trigger**: Created or Edited -> Field Actions -> `Overlay` -> On User Input
- **Plain English Logic**:
  - Step 1: Check if an Overlay is selected. If cleared, do nothing.
  - Step 2: Query the `Devices` record using pre-assigned ID variable to verify hardware type is `"Overlay"`. If mismatched, alert user and clear field.
  - Step 3: In Add mode, check if the device's current `Device_State` is already active elsewhere. If active, warn the user.

### Workflow 2.3: On User Input (State)
- **File**: `scripts/07_deployment_logs/2_3_Deployment_Logs_onuserinput_state.deluge`
- **Trigger**: Created or Edited -> Field Actions -> `State` -> On User Input
- **Plain English Logic**:
  - Step 1: If user selects `"Returned"`, auto-populate `End_Date` with `zoho.currentdate` if empty, providing immediate feedback.
  - Step 2: If user switches back to an active state (`"Rental"`, `"Demo"`, `"Idle"`), reset `End_Date` if it was auto-set to today.

### Workflow 3: On Form Validate
- **File**: `scripts/07_deployment_logs/3_Deployment_Logs_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory checks: Assert `Customer`, `Start_Date`, and `State` are provided.
  - Step 2: System Pairing guard: Assert at least one physical component (`Control_Unit` or `Overlay`) is selected.
  - Step 3: Date validation: If `End_Date` is provided, assert `End_Date >= Start_Date`.
  - Step 4: Return validation: If `State == "Returned"`, assert `End_Date` is not empty.
  - Step 5: Double-Booking guard: If `State != "Returned"`, query active deployment logs to confirm neither the selected Control Unit nor Overlay is already deployed in another ongoing deployment.

### Workflow 4: On Form Submission Success
- **File**: `scripts/07_deployment_logs/4_Deployment_Logs_onsuccess.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Map the deployment `State` to target `Device_State` and target `Current_Customer`:
    - `"Rental"` -> `new_state = "Rentally Active"`, `target_cust = input.Customer`
    - `"Demo"` -> `new_state = "Active in Demo"`, `target_cust = input.Customer`
    - `"Idle"` -> `new_state = "Idle with Customer"`, `target_cust = input.Customer`
    - `"Returned"` -> `new_state = "Available in Stock"`, `target_cust = null`
  - Step 2: If `Control_Unit` is selected, fetch the target device record by ID, update its `Device_State` and `Current_Customer`.
  - Step 3: If `Overlay` is selected, fetch the target device record by ID, update its `Device_State` and `Current_Customer`.
  - Step 4: Redirect cleanly to `#Report:All_Deployment_Logs`.
