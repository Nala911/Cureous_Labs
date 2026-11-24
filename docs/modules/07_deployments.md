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
  - **Rental**: Device State becomes `"Rentally Active"`, Current Customer is assigned.
  - **Demo**: Device State becomes `"Active in Demo"`, Current Customer is assigned.
  - **Idle**: Device State becomes `"Idle with Customer"`, Current Customer is assigned.
  - **Closed**: Device State reverts to `"Available in Stock"`, Current Customer is unassigned (`null`).
- **Conflict Prevention**: Validates that neither the selected Control Unit nor Overlay is currently actively deployed with another customer.

---

## 2. Field Specifications

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Customer** | `Customer` | Lookup | **User Entry** (Mandatory, Relational Link to Customer Master) | `Customers` (`Customer_Name`) | `scripts/07_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/4_Deployments_onsuccess.deluge` | Sets `Devices.Current_Customer` on paired assets |
| **Control Unit** | `Control_Unit` | Lookup | **User Entry** (Optional if Overlay provided; Form Builder Lookup Filter: `Device_Type == "Control Unit"`) | `Devices` (`Serial_Number`) | `scripts/07_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/2_1_Deployments_onuserinput_control_unit.deluge`<br>`scripts/07_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/4_Deployments_onsuccess.deluge` | Direct asset allocation and state tracking |
| **Overlay** | `Overlay` | Lookup | **User Entry** (Optional if Control Unit provided; Form Builder Lookup Filter: `Device_Type == "Overlay"`) | `Devices` (`Serial_Number`) | `scripts/07_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/2_2_Deployments_onuserinput_overlay.deluge`<br>`scripts/07_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/4_Deployments_onsuccess.deluge` | Direct asset allocation and state tracking |
| **Start Date** | `Start_Date` | Date | **User Entry** (Mandatory, Defaults to `zoho.currentdate`) | — | `scripts/07_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/3_Deployments_onvalidate.deluge` | Commercial billing start anchor |
| **End Date** | `End_Date` | Date | **Hidden on Add** (Revealed, enabled, and mandatory when `Deployment_State == "Closed"`; must be `>= Start_Date`) | — | `scripts/07_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/2_3_Deployments_onuserinput_deployment_state.deluge`<br>`scripts/07_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/4_Deployments_onsuccess.deluge` | Commercial billing conclusion anchor |
| **Deployment State** | `Deployment_State` | Dropdown | **User Entry** (Mandatory; Add Mode choices: `"Rental"`, `"Demo"` defaulting to `"Rental"`; Edit Mode choices: `"Rental"`, `"Demo"`, `"Idle"`, `"Closed"`) | — | `scripts/07_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/2_3_Deployments_onuserinput_deployment_state.deluge`<br>`scripts/07_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/4_Deployments_onsuccess.deluge` | Triggers automated updates to `Devices.Device_State` |

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/07_deployments/1_Deployments_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable user input fields (`Customer`, `Control_Unit`, `Overlay`, `Start_Date`, `Deployment_State`) and hide `End_Date`. Default `Start_Date` to `zoho.currentdate`. Clear `Deployment_State`, dynamically populate only `"Rental"` and `"Demo"`, and default to `"Rental"`.
  - In EDIT Mode: If the record is already marked `"Closed"`, reveal `End_Date` and lock all deployment fields (`Customer`, `Control_Unit`, `Overlay`, `Start_Date`, `End_Date`, `Deployment_State`) to prevent accidental modification of closed audit logs. Otherwise, populate the full lifecycle choices (`"Rental"`, `"Demo"`, `"Idle"`, `"Closed"`) while preserving the current selection, and hide `End_Date` unless already populated.

### Workflow 2.1: On User Input (Control Unit)
- **File**: `scripts/07_deployments/2_1_Deployments_onuserinput_control_unit.deluge`
- **Trigger**: Created or Edited -> Field Actions -> `Control_Unit` -> On User Input
- **Plain English Logic**:
  - Step 1: Check if a Control Unit is selected. If cleared, do nothing.
  - Step 2: Query the `Devices` record using pre-assigned ID variable to verify hardware type is `"Control Unit"`. If mismatched, alert user and clear field.
  - Step 3: In Add mode, check if the device's current `Device_State` is already `"Rentally Active"`, `"Active in Demo"`, or `"Idle with Customer"`. If active elsewhere, warn the user to ensure previous deployment was closed.

### Workflow 2.2: On User Input (Overlay)
- **File**: `scripts/07_deployments/2_2_Deployments_onuserinput_overlay.deluge`
- **Trigger**: Created or Edited -> Field Actions -> `Overlay` -> On User Input
- **Plain English Logic**:
  - Step 1: Check if an Overlay is selected. If cleared, do nothing.
  - Step 2: Query the `Devices` record using pre-assigned ID variable to verify hardware type is `"Overlay"`. If mismatched, alert user and clear field.
  - Step 3: In Add mode, check if the device's current `Device_State` is already active elsewhere. If active, warn the user to ensure previous deployment was closed.

### Workflow 2.3: On User Input (Deployment State)
- **File**: `scripts/07_deployments/2_3_Deployments_onuserinput_deployment_state.deluge`
- **Trigger**: Created or Edited -> Field Actions -> `Deployment_State` -> On User Input
- **Plain English Logic**:
  - Step 1: If user selects `"Closed"`, show and enable `End_Date`, and auto-populate it with `zoho.currentdate` if empty.
  - Step 2: If user switches back to an active state (`"Rental"`, `"Demo"`, `"Idle"`), hide `End_Date` and clear it if it was auto-set to today.

### Workflow 3: On Form Validate
- **File**: `scripts/07_deployments/3_Deployments_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory checks: Assert `Customer`, `Start_Date`, and `Deployment_State` are provided.
  - Step 2: System Pairing guard: Assert at least one physical component (`Control_Unit` or `Overlay`) is selected.
  - Step 3: Date validation: If `End_Date` is provided, assert `End_Date >= Start_Date`.
  - Step 4: Closure validation: If `Deployment_State == "Closed"`, assert `End_Date` is not empty.
  - Step 5: Double-Booking guard: If `Deployment_State != "Closed"`, query active deployments to confirm neither the selected Control Unit nor Overlay is already deployed in another ongoing deployment.

### Workflow 4: On Form Submission Success
- **File**: `scripts/07_deployments/4_Deployments_onsuccess.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Map `Deployment_State` to target `Device_State` and target `Current_Customer`:
    - `"Rental"` -> `new_state = "Rentally Active"`, `target_cust = input.Customer`
    - `"Demo"` -> `new_state = "Active in Demo"`, `target_cust = input.Customer`
    - `"Idle"` -> `new_state = "Idle with Customer"`, `target_cust = input.Customer`
    - `"Closed"` -> `new_state = "Available in Stock"`, `target_cust = null`
  - Step 2: If `Control_Unit` is selected, fetch the target device record by ID, update its `Device_State` and `Current_Customer`.
  - Step 3: If `Overlay` is selected, fetch the target device record by ID, update its `Device_State` and `Current_Customer`.
  - Step 4: Redirect cleanly to `#Report:All_Deployments`.
