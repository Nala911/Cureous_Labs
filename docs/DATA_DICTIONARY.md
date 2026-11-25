# CUREOUS PRS - MASTER DATA & DEPENDENCY DICTIONARY

This file serves as the **central cross-reference and dependency matrix** for the entire Cureous Patient Repositioning System (PRS) application in Zoho Creator.

Whenever any field, status, or logic is created, modified, or removed:
1. Search this document to identify every script and dependent module referencing it.
2. Update the target scripts and documentation.
3. Update this dictionary to keep the dependency matrix 100% accurate.

---

## High-Level Relational Entity Diagram

```
[Lots] (Module 6) ──────────┐
                            ▼
[Offices] (Module 3) ──> [Devices] (Module 5) <──── [Deployments] (Module 7)
                            ▲        ▲                   │
                            │        │                   ▼
[Product_Models] (Mod 8) ───┘        └────── [Customers] (Module 2) <┘
```

---

## Module 2: Customers (`Customers`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Customer Name** | `Customer_Name` | Dropdown | **User Entry** (Mandatory, Synced Books; Locked in Edit) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge`<br>`scripts/02_customers/3_Customers_onvalidate.deluge` | Display label for Module 5 and commercial modules |
| **Customer ID** | `Customer_ID` | Number | **Hidden Background Field** (PK / Books Contact ID, Read-Only) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge`<br>`scripts/02_customers/3_Customers_onvalidate.deluge` | Zoho Books API Invoicing / Contact mapping |
| **Customer Type** | `Customer_Type` | Dropdown | **Disabled Context** (Read-Only: "Business", "Individual")| — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge` | Commercial contract categorization |
| **Billing Address** | `Billing_Address` | Multi-Line | **Disabled Context** (Read-Only, Synced Books) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge` | Reference on challans and agreements |

---

## Module 3: Offices (`Offices`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Office Name** | `Office_Name` | Single Line | **User Entry** (Mandatory, Unique) | — | `scripts/03_offices/1_Offices_onload.deluge`<br>`scripts/03_offices/3_Offices_onvalidate.deluge` | Master registry of branch offices / hubs for device custody & transfers |

---

## Module 5: Devices (`Devices`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Serial Number** | `Serial_Number` | Single Line | **User Entry** (Mandatory, Unique, PK; Locked in Edit) | — | `scripts/05_devices/form_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/form_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/form_devices/3_Devices_onvalidate.deluge` | Module 11 (`Quality_Control`) |
| **Device Type** | `Device_Type` | Dropdown | **Disabled Context** (Read-Only: "Control Unit", "Overlay") | — | `scripts/05_devices/form_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/form_devices/2_1_Devices_onuserinput_serialnumber.deluge` | Auto-derived from serial string; hardware classification |
| **Product Model** | `Product_Model` | Lookup | **Auto-Derived & User Editable on Add; Locked in Edit** | `Product_Models` (`Product_Model_Name`) | `scripts/05_devices/form_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/form_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/form_devices/3_Devices_onvalidate.deluge` | Module 08 (`Product_Models`) hardware model classification |
| **Lot** | `Lot` | Lookup | **Auto-Default Latest & User Editable on Add and Edit** | `Lots` (`Lot_ID`) | `scripts/05_devices/form_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/form_devices/3_Devices_onvalidate.deluge` | Module 06 (`Lots`) batch traceability & recall management |
| **Current Customer** | `Current_Customer` | Lookup | **Hidden on Add** (Report / Allocation Only, Read-Only) | `Customers` (`Customer_Name`) | `scripts/05_devices/form_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/stateless_update_custody/5_1_Update_Device_Custody_onclick_submit.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge`<br>`scripts/07_deployments/stateless_close_deployments/5_1_Close_Deployments_onclick_submit.deluge`<br>`scripts/07_deployments/stateless_swap_devices/5_1_Swap_Devices_onclick_submit.deluge` | Direct customer allocation; cleared when `Collected == true` or retained when `Collected == false` in `Close_Deployments` / `Swap_Devices`; updated by `Deployments` and `Update_Device_Custody` |
| **Current Office** | `Current_Office` | Lookup | **Hidden on Add** (Report / Location Only, Read-Only) | `Offices` (`Office_Name`) | `scripts/05_devices/form_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/stateless_update_custody/5_1_Update_Device_Custody_onclick_submit.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge`<br>`scripts/07_deployments/stateless_close_deployments/5_1_Close_Deployments_onclick_submit.deluge`<br>`scripts/07_deployments/stateless_swap_devices/5_1_Swap_Devices_onclick_submit.deluge` | Direct branch / warehouse custody tracking; updated by `Update_Device_Custody` & `Close_Deployments`/`Swap_Devices` (when `Collected == true`), cleared (`null`) when at customer |
| **Device Stage** | `Device_Stage` | Dropdown | **Hidden on Add** (Auto-init Stage 0: `"0 - Mfg Not Yet Started"`; Enabled in Edit mode with unified shared stages + hardware-specific Stages 1–2) | — | `scripts/05_devices/form_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/form_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/form_devices/2_2_Devices_onupdate_devicestage.deluge`<br>`scripts/05_devices/form_devices/3_Devices_onvalidate.deluge`<br>`scripts/05_devices/stateless_update_stage/5_1_Update_Device_Stage_onclick_submit.deluge`<br>`scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/2_1_Deployments_onuserinput_control_unit.deluge`<br>`scripts/07_deployments/form_deployments/2_2_Deployments_onuserinput_overlay.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/stateless_close_deployments/5_1_Close_Deployments_onclick_submit.deluge`<br>`scripts/07_deployments/stateless_swap_devices/2_2_Swap_Devices_onuserinput_control_unit.deluge`<br>`scripts/07_deployments/stateless_swap_devices/2_3_Swap_Devices_onuserinput_overlay.deluge`<br>`scripts/07_deployments/stateless_swap_devices/5_1_Swap_Devices_onclick_submit.deluge` | Auto-harmonized on Stages 1–2 during bulk edit; readiness gate (`"6 - Ready Condition"`) for `Deployments` & `Swap_Devices`; updated via `Update_Device_Stage`, `Close_Deployments`, and `Swap_Devices` |
| **Device State** | `Device_State` | Dropdown | **Hidden on Add** (Report / Status Only, Read-Only, Options: "Rentally Active", "Active in Demo", "Idle with Customer", "Available in Stock") | — | `scripts/05_devices/form_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/form_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/form_devices/3_Devices_onvalidate.deluge`<br>`scripts/05_devices/stateless_update_custody/5_1_Update_Device_Custody_onclick_submit.deluge`<br>`scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/2_1_Deployments_onuserinput_control_unit.deluge`<br>`scripts/07_deployments/form_deployments/2_2_Deployments_onuserinput_overlay.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge`<br>`scripts/07_deployments/stateless_close_deployments/5_1_Close_Deployments_onclick_submit.deluge`<br>`scripts/07_deployments/stateless_swap_devices/5_1_Swap_Devices_onclick_submit.deluge` | Real-time commercial status & Add-mode `Deployments` readiness filter (`"Available in Stock"`); updated automatically by Module 07 and `Update_Device_Custody` |

---

## Module 6: Lots (`Lots`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Lot ID** | `Lot_ID` | Single Line | **User Entry** (Mandatory, Unique, Primary Identifier) | — | `scripts/05_devices/form_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/form_devices/3_Devices_onvalidate.deluge` | Source of truth for `Devices.Lot` manufacturing batch tracking |

---

## Module 7: Deployments (`Deployments`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Customer** | `Customer` | Lookup | **User Entry** (Mandatory, Relational Link to Customer Master) | `Customers` (`Customer_Name`) | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge` | Sets `Devices.Current_Customer` on paired assets |
| **Control Unit** | `Control_Unit` | Lookup | **User Entry on Add / Locked on Edit** (Optional if Overlay provided; Form Builder Lookup Filter: `Device_Type == "Control Unit"`; Add-Mode dynamic filter & validation: `Device_Stage == "6 - Ready Condition" && Device_State == "Available in Stock"`; swaps go through `Swap_Devices`) | `Devices` (`Serial_Number`) | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/2_1_Deployments_onuserinput_control_unit.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge` | Direct asset allocation and readiness/state tracking |
| **Overlay** | `Overlay` | Lookup | **User Entry on Add / Locked on Edit** (Optional if Control Unit provided; Form Builder Lookup Filter: `Device_Type == "Overlay"`; Add-Mode dynamic filter & validation: `Device_Stage == "6 - Ready Condition" && Device_State == "Available in Stock"`; swaps go through `Swap_Devices`) | `Devices` (`Serial_Number`) | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/2_2_Deployments_onuserinput_overlay.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge` | Direct asset allocation and readiness/state tracking |
| **Start Date** | `Start_Date` | Date | **User Entry** (Mandatory, Defaults to `zoho.currentdate`) | — | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge` | Commercial billing start anchor |
| **End Date** | `End_Date` | Date | **Hidden on Add** (Revealed, enabled, and mandatory when `Deployment_State == "Closed"`; must be `>= Start_Date`) | — | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/2_3_Deployments_onuserinput_deployment_state.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge` | Commercial billing conclusion anchor |
| **Deployment State** | `Deployment_State` | Dropdown | **User Entry** (Mandatory; Add Mode choices: `"Rental"`, `"Demo"` defaulting to `"Rental"`; Edit Mode choices: `"Rental"`, `"Demo"`, `"Idle"`; closures flow through `Close_Deployments`) | — | `scripts/07_deployments/form_deployments/1_Deployments_onload.deluge`<br>`scripts/07_deployments/form_deployments/2_3_Deployments_onuserinput_deployment_state.deluge`<br>`scripts/07_deployments/form_deployments/3_Deployments_onvalidate.deluge`<br>`scripts/07_deployments/form_deployments/4_Deployments_onsuccess.deluge` | Triggers automated updates to `Devices.Device_State` |

---

## Module 8: Product Models (`Product_Models`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Product Model Name** | `Product_Model_Name` | Single Line | **User Entry** (Mandatory, Unique, Primary Identifier) | — | `scripts/05_devices/form_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/form_devices/3_Devices_onvalidate.deluge` | Source of truth for `Devices.Product_Model` hardware model lookup |

---

### Module 5 Stateless Utility: Update Device Stage (`Update_Device_Stage`)
- **Parent Module**: Module 5 (`Devices`)
- **Pre-fill URL Parameters**: `?Select_Devices=<comma-separated Device IDs>` (passed from `All_Devices` via `open_update_stage`)
- **Cross-Form Updates**: Updates `Devices.Device_Stage` (with CU/OV auto-harmonization)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Select Devices** | `Select_Devices` | Lookup (Multi-Select) | **User Entry / Report Action Pre-fill** (Mandatory) | `Devices` (`Serial_Number`) | `scripts/05_devices/stateless_update_stage/1_Update_Device_Stage_onload.deluge`<br>`scripts/05_devices/stateless_update_stage/5_1_Update_Device_Stage_onclick_submit.deluge`<br>`scripts/05_devices/report_all_devices/5_1_All_Devices_customaction_open_update_stage.deluge` | References target devices in `Devices` to bulk update stage |
| **New Device Stage** | `New_Device_Stage` | Dropdown | **User Entry** (Mandatory; CU & OV manufacturing stages) | — | `scripts/05_devices/stateless_update_stage/5_1_Update_Device_Stage_onclick_submit.deluge` | Target stage applied across selected devices with auto-harmonization |

---

### Module 5 Stateless Utility: Update Device Custody (`Update_Device_Custody`)
- **Parent Module**: Module 5 (`Devices`)
- **Pre-fill URL Parameters**: `?Select_Devices=<comma-separated Device IDs>` (passed from `All_Devices` via `open_update_custody`)
- **Cross-Form Updates**: Updates `Devices.Current_Office`, `Devices.Current_Customer`, and `Devices.Device_State`

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Select Devices** | `Select_Devices` | Lookup (Multi-Select) | **User Entry / Report Action Pre-fill** (Mandatory) | `Devices` (`Serial_Number`) | `scripts/05_devices/stateless_update_custody/1_Update_Device_Custody_onload.deluge`<br>`scripts/05_devices/stateless_update_custody/5_1_Update_Device_Custody_onclick_submit.deluge`<br>`scripts/05_devices/report_all_devices/5_2_All_Devices_customaction_open_update_custody.deluge` | References target devices in `Devices` to bulk update location/custody |
| **Destination Type** | `Destination_Type` | Dropdown / Radio | **User Entry** (Mandatory; Choices: "Internal Office Transfer", "Customer Deployment") | — | `scripts/05_devices/stateless_update_custody/2_1_Update_Device_Custody_onuserinput_destinationtype.deluge`<br>`scripts/05_devices/stateless_update_custody/5_1_Update_Device_Custody_onclick_submit.deluge` | Toggles dynamic display of Office vs Customer lookup |
| **Target Office** | `Target_Office` | Lookup | **Dynamic User Entry** (Mandatory when Transfer is Office; hidden otherwise) | `Offices` (`Office_Name`) | `scripts/05_devices/stateless_update_custody/1_Update_Device_Custody_onload.deluge`<br>`scripts/05_devices/stateless_update_custody/2_1_Update_Device_Custody_onuserinput_destinationtype.deluge`<br>`scripts/05_devices/stateless_update_custody/5_1_Update_Device_Custody_onclick_submit.deluge` | Sets `Devices.Current_Office` |
| **Target Customer** | `Target_Customer` | Lookup | **Dynamic User Entry** (Mandatory when Transfer is Customer; hidden otherwise) | `Customers` (`Customer_Name`) | `scripts/05_devices/stateless_update_custody/1_Update_Device_Custody_onload.deluge`<br>`scripts/05_devices/stateless_update_custody/2_1_Update_Device_Custody_onuserinput_destinationtype.deluge`<br>`scripts/05_devices/stateless_update_custody/5_1_Update_Device_Custody_onclick_submit.deluge` | Sets `Devices.Current_Customer` |


---

### Module 7 Stateless Utility: Close Deployments (`Close_Deployments`)
- **Parent Module**: Module 7 (`Deployments`)
- **Pre-fill URL Parameters**: `?Deployments=<comma-separated Deployment IDs>` (passed from `All_Deployments` via `open_close_deployments`)
- **Cross-Form Updates**: Updates `Deployments.Deployment_State = "Closed"` and `Deployments.End_Date`. Updates `Devices.Device_Stage = input.Condition` on all linked assets. When `Collected == true`, sets `Devices.Device_State = "Available in Stock"`, `Devices.Current_Customer = null`, and `Devices.Current_Office = input.Return_Office`; when `Collected == false`, sets `Devices.Device_State = "Idle with Customer"`, `Devices.Current_Office = null`, and preserves `Devices.Current_Customer`.

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Deployments** | `Deployments` | Lookup (Multi-Select) | **User Entry / Report Action Pre-fill** (Mandatory; Form Builder Filter: `Deployment_State != "Closed"`) | `Deployments` | `scripts/07_deployments/stateless_close_deployments/1_Close_Deployments_onload.deluge`<br>`scripts/07_deployments/stateless_close_deployments/5_1_Close_Deployments_onclick_submit.deluge`<br>`scripts/07_deployments/report_all_deployments/5_1_All_Deployments_customaction_open_close_deployments.deluge` | Target deployments to close in bulk or single mode |
| **End Date** | `End_Date` | Date | **User Entry** (Mandatory, Defaults to `zoho.currentdate`; must be `>= Start_Date` of all selected deployments) | — | `scripts/07_deployments/stateless_close_deployments/1_Close_Deployments_onload.deluge`<br>`scripts/07_deployments/stateless_close_deployments/5_1_Close_Deployments_onclick_submit.deluge` | Sets `Deployments.End_Date` |
| **Collected?** | `Collected` | Decision Box | **User Entry** (Defaults to `false`) | — | `scripts/07_deployments/stateless_close_deployments/1_Close_Deployments_onload.deluge`<br>`scripts/07_deployments/stateless_close_deployments/2_1_Close_Deployments_onuserinput_collected.deluge`<br>`scripts/07_deployments/stateless_close_deployments/5_1_Close_Deployments_onclick_submit.deluge` | Controls visibility of `Return_Office` and whether devices move to `"Available in Stock"` at `Return_Office` or stay `"Idle with Customer"` |
| **Return Office** | `Return_Office` | Lookup | **Dynamic User Entry** (Hidden by default; Shown & Mandatory when `Collected == true`) | `Offices` (`Office_Name`) | `scripts/07_deployments/stateless_close_deployments/1_Close_Deployments_onload.deluge`<br>`scripts/07_deployments/stateless_close_deployments/2_1_Close_Deployments_onuserinput_collected.deluge`<br>`scripts/07_deployments/stateless_close_deployments/5_1_Close_Deployments_onclick_submit.deluge` | Sets `Devices.Current_Office` on closed assets when collected |
| **Condition** | `Condition` | Dropdown | **User Entry** (Always visible, Mandatory, Defaults to `"Disinfection Needed"`; Choices: `"Disinfection Needed"`, `"Retesting Needed"`, `"Repair Needed"`, `"6 - Ready Condition"`) | — | `scripts/07_deployments/stateless_close_deployments/1_Close_Deployments_onload.deluge`<br>`scripts/07_deployments/stateless_close_deployments/5_1_Close_Deployments_onclick_submit.deluge` | Sets `Devices.Device_Stage` on closed deployment assets |

---

### Module 7 Stateless Utility: Swap Devices (`Swap_Devices`)
- **Parent Module**: Module 7 (`Deployments`)
- **Pre-fill URL Parameters**: `?Deployment=<Deployment ID>` (passed from `All_Deployments` via `open_swap_devices`)
- **Cross-Form Updates**: Closes selected `Deployments` record (`Deployment_State = "Closed"`, `End_Date = Swap_Date`), inserts a new `Deployments` record inheriting `Customer` and `Deployment_State` with `Start_Date = Swap_Date`. Updates swapped-out `Devices` with `Device_Stage = input.Condition`: when `Collected == true`, sets `Device_State = "Available in Stock"`, `Current_Customer = null`, and `Current_Office = Return_Office`; when `Collected == false`, sets `Device_State = "Idle with Customer"`, `Current_Office = null`, and preserves `Current_Customer`. Updates new/retained `Devices` to the active `Device_State`, `Current_Customer`, and `Current_Office = null`.

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Deployment** | `Deployment` | Lookup | **User Entry / Report Action Pre-fill** (Mandatory; Form Builder Filter: `Deployment_State != "Closed"`) | `Deployments` | `scripts/07_deployments/stateless_swap_devices/1_Swap_Devices_onload.deluge`<br>`scripts/07_deployments/stateless_swap_devices/2_1_Swap_Devices_onuserinput_deployment.deluge`<br>`scripts/07_deployments/stateless_swap_devices/5_1_Swap_Devices_onclick_submit.deluge`<br>`scripts/07_deployments/report_all_deployments/5_2_All_Deployments_customaction_open_swap_devices.deluge` | Source active deployment to close and replace |
| **Control Unit** | `Control_Unit` | Lookup | **Auto-Filled / User Entry** (Optional if Overlay provided; Auto-populated from `Deployment`; Form Builder Filter: `Device_Type == "Control Unit"`; replacement units validated to `Device_Stage == "6 - Ready Condition" && Device_State == "Available in Stock"`) | `Devices` (`Serial_Number`) | `scripts/07_deployments/stateless_swap_devices/1_Swap_Devices_onload.deluge`<br>`scripts/07_deployments/stateless_swap_devices/2_1_Swap_Devices_onuserinput_deployment.deluge`<br>`scripts/07_deployments/stateless_swap_devices/2_2_Swap_Devices_onuserinput_control_unit.deluge`<br>`scripts/07_deployments/stateless_swap_devices/5_1_Swap_Devices_onclick_submit.deluge` | Target Control Unit for the new deployment |
| **Overlay** | `Overlay` | Lookup | **Auto-Filled / User Entry** (Optional if Control Unit provided; Auto-populated from `Deployment`; Form Builder Filter: `Device_Type == "Overlay"`; replacement units validated to `Device_Stage == "6 - Ready Condition" && Device_State == "Available in Stock"`) | `Devices` (`Serial_Number`) | `scripts/07_deployments/stateless_swap_devices/1_Swap_Devices_onload.deluge`<br>`scripts/07_deployments/stateless_swap_devices/2_1_Swap_Devices_onuserinput_deployment.deluge`<br>`scripts/07_deployments/stateless_swap_devices/2_3_Swap_Devices_onuserinput_overlay.deluge`<br>`scripts/07_deployments/stateless_swap_devices/5_1_Swap_Devices_onclick_submit.deluge` | Target Overlay for the new deployment |
| **Date** | `Swap_Date` | Date | **User Entry** (Mandatory, Defaults to `zoho.currentdate`; must be `>= Start_Date` of the selected deployment) | — | `scripts/07_deployments/stateless_swap_devices/1_Swap_Devices_onload.deluge`<br>`scripts/07_deployments/stateless_swap_devices/5_1_Swap_Devices_onclick_submit.deluge` | Sets old `Deployments.End_Date` and new `Deployments.Start_Date` |
| **Collected?** | `Collected` | Decision Box | **User Entry** (Defaults to `false`) | — | `scripts/07_deployments/stateless_swap_devices/1_Swap_Devices_onload.deluge`<br>`scripts/07_deployments/stateless_swap_devices/2_4_Swap_Devices_onuserinput_collected.deluge`<br>`scripts/07_deployments/stateless_swap_devices/5_1_Swap_Devices_onclick_submit.deluge` | Controls visibility of `Return_Office` and whether swapped-out devices move to `"Available in Stock"` at `Return_Office` or stay `"Idle with Customer"` |
| **Return Office** | `Return_Office` | Lookup | **Dynamic User Entry** (Hidden by default; Shown & Mandatory when `Collected == true`) | `Offices` (`Office_Name`) | `scripts/07_deployments/stateless_swap_devices/1_Swap_Devices_onload.deluge`<br>`scripts/07_deployments/stateless_swap_devices/2_4_Swap_Devices_onuserinput_collected.deluge`<br>`scripts/07_deployments/stateless_swap_devices/5_1_Swap_Devices_onclick_submit.deluge` | Sets `Devices.Current_Office` on swapped-out assets when collected |
| **Condition** | `Condition` | Dropdown | **User Entry** (Always visible, Mandatory, Defaults to `"Disinfection Needed"`; Choices: `"Disinfection Needed"`, `"Retesting Needed"`, `"Repair Needed"`, `"6 - Ready Condition"`) | — | `scripts/07_deployments/stateless_swap_devices/1_Swap_Devices_onload.deluge`<br>`scripts/07_deployments/stateless_swap_devices/5_1_Swap_Devices_onclick_submit.deluge` | Sets `Devices.Device_Stage` on swapped-out assets |
