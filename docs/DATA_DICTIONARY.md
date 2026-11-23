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
[Offices] (Module 3) ──> [Devices] (Module 5) <──── [Deployment_Logs] (Module 7)
                            ▲        ▲                   │
                            │        │                   ▼
[Product_Models] (Mod 8) ───┘        └────── [Customers] (Module 2) <┘
                                         ▲
                                         │
                             [Rental_Items] (Module 4)
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

## Module 4: Rental Items (`Rental_Items`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Item Name** | `Item_Name` | Dropdown | **User Entry** (Mandatory, Synced Books / Standard Choices; Locked in Edit) | — | `scripts/04_rental_items/1_Rental_Items_onload.deluge`<br>`scripts/04_rental_items/2_1_Rental_Items_onuserinput_itemname.deluge`<br>`scripts/04_rental_items/3_Rental_Items_onvalidate.deluge` | Display label for catalog items |
| **Rental item id** | `Rental_Item_ID` | Single Line | **System Managed / Disabled Context** (Hidden, Auto-populated from Zoho Books) | — | `scripts/04_rental_items/1_Rental_Items_onload.deluge`<br>`scripts/04_rental_items/2_1_Rental_Items_onuserinput_itemname.deluge`<br>`scripts/04_rental_items/3_Rental_Items_onvalidate.deluge` | Direct Books Item ID passed to line items when generating invoices in deployment logs |

---

## Module 5: Devices (`Devices`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Serial Number** | `Serial_Number` | Single Line | **User Entry** (Mandatory, Unique, PK; Locked in Edit) | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Module 11 (`Quality_Control`) |
| **Device Type** | `Device_Type` | Dropdown | **Disabled Context** (Read-Only: "Control Unit", "Overlay") | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge` | Auto-derived from serial string; hardware classification |
| **Product Model** | `Product_Model` | Lookup | **Auto-Derived & User Editable on Add; Locked in Edit** | `Product_Models` (`Product_Model_Name`) | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Module 08 (`Product_Models`) hardware model classification |
| **Lot** | `Lot` | Lookup | **Auto-Default Latest & User Editable on Add and Edit** | `Lots` (`Lot_ID`) | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Module 06 (`Lots`) batch traceability & recall management |
| **Current Customer** | `Current_Customer` | Lookup | **Hidden on Add** (Report / Allocation Only, Read-Only) | `Customers` (`Customer_Name`) | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/5_1_Update_Device_Custody_onclick_submit.deluge`<br>`scripts/07_deployment_logs/4_Deployment_Logs_onsuccess.deluge` | Direct customer allocation; dynamically assigned and unassigned by Module 07 (`Deployment_Logs`) and `Update_Device_Custody` |
| **Current Office** | `Current_Office` | Lookup | **Hidden on Add** (Report / Location Only, Read-Only) | `Offices` (`Office_Name`) | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/5_1_Update_Device_Custody_onclick_submit.deluge` | Direct branch / warehouse custody tracking; updated by `Update_Device_Custody` |
| **Device Stage** | `Device_Stage` | Dropdown | **Hidden on Add** (Auto-init Stage 0: "0 - CU Mfg Not Yet Started" / "0 - OV Mfg Not Yet Started"; Enabled in Edit mode with dynamic choices) | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/2_2_Devices_onupdate_devicestage.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge`<br>`scripts/05_devices/5_1_Update_Device_Stage_onclick_submit.deluge` | Auto-harmonized on bulk edit; Stage gating for manufacturing and Module 11 QC; updated via `Update_Device_Stage` |
| **Device State** | `Device_State` | Dropdown | **Hidden on Add** (Report / Status Only, Read-Only, Options: "Rentally Active", "Active in Demo", "Idle with Customer", "Available in Stock") | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge`<br>`scripts/05_devices/5_1_Update_Device_Custody_onclick_submit.deluge`<br>`scripts/07_deployment_logs/2_1_Deployment_Logs_onuserinput_control_unit.deluge`<br>`scripts/07_deployment_logs/2_2_Deployment_Logs_onuserinput_overlay.deluge`<br>`scripts/07_deployment_logs/4_Deployment_Logs_onsuccess.deluge` | Real-time commercial and operational status tracking; updated automatically by Module 07 (`Deployment_Logs`) and `Update_Device_Custody` |

---

## Module 6: Lots (`Lots`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Lot ID** | `Lot_ID` | Single Line | **User Entry** (Mandatory, Unique, Primary Identifier) | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Source of truth for `Devices.Lot` manufacturing batch tracking |

---

## Module 7: Deployment Logs (`Deployment_Logs`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Customer** | `Customer` | Lookup | **User Entry** (Mandatory, Relational Link to Customer Master) | `Customers` (`Customer_Name`) | `scripts/07_deployment_logs/1_Deployment_Logs_onload.deluge`<br>`scripts/07_deployment_logs/3_Deployment_Logs_onvalidate.deluge`<br>`scripts/07_deployment_logs/4_Deployment_Logs_onsuccess.deluge` | Sets `Devices.Current_Customer` on paired assets |
| **Control Unit** | `Control_Unit` | Lookup | **User Entry** (Optional if Overlay provided; Filtered to `Device_Type == "Control Unit"`) | `Devices` (`Serial_Number`) | `scripts/07_deployment_logs/1_Deployment_Logs_onload.deluge`<br>`scripts/07_deployment_logs/2_1_Deployment_Logs_onuserinput_control_unit.deluge`<br>`scripts/07_deployment_logs/3_Deployment_Logs_onvalidate.deluge`<br>`scripts/07_deployment_logs/4_Deployment_Logs_onsuccess.deluge` | Direct asset allocation and state tracking |
| **Overlay** | `Overlay` | Lookup | **User Entry** (Optional if Control Unit provided; Filtered to `Device_Type == "Overlay"`) | `Devices` (`Serial_Number`) | `scripts/07_deployment_logs/1_Deployment_Logs_onload.deluge`<br>`scripts/07_deployment_logs/2_2_Deployment_Logs_onuserinput_overlay.deluge`<br>`scripts/07_deployment_logs/3_Deployment_Logs_onvalidate.deluge`<br>`scripts/07_deployment_logs/4_Deployment_Logs_onsuccess.deluge` | Direct asset allocation and state tracking |
| **Start Date** | `Start_Date` | Date | **User Entry** (Mandatory, Defaults to current date) | — | `scripts/07_deployment_logs/1_Deployment_Logs_onload.deluge`<br>`scripts/07_deployment_logs/3_Deployment_Logs_onvalidate.deluge` | Commercial billing start anchor |
| **End Date** | `End_Date` | Date | **User Entry** (Optional on Add, Mandatory when State is Returned; must be `>= Start_Date`) | — | `scripts/07_deployment_logs/2_3_Deployment_Logs_onuserinput_state.deluge`<br>`scripts/07_deployment_logs/3_Deployment_Logs_onvalidate.deluge`<br>`scripts/07_deployment_logs/4_Deployment_Logs_onsuccess.deluge` | Commercial billing conclusion anchor |
| **State** | `State` | Dropdown | **User Entry** (Mandatory; Choices: `"Rental"`, `"Demo"`, `"Idle"`, `"Returned"`; Defaults to `"Rental"`) | — | `scripts/07_deployment_logs/1_Deployment_Logs_onload.deluge`<br>`scripts/07_deployment_logs/2_3_Deployment_Logs_onuserinput_state.deluge`<br>`scripts/07_deployment_logs/3_Deployment_Logs_onvalidate.deluge`<br>`scripts/07_deployment_logs/4_Deployment_Logs_onsuccess.deluge` | Triggers automated updates to `Devices.Device_State` |

---

## Module 8: Product Models (`Product_Models`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Product Model Name** | `Product_Model_Name` | Single Line | **User Entry** (Mandatory, Unique, Primary Identifier) | — | `scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Source of truth for `Devices.Product_Model` hardware model lookup |

---

## Stateless Utility: Update Device Stage (`Update_Device_Stage`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Select Devices** | `Select_Devices` | Lookup (Multi-Select) | **User Entry / Report Action Pre-fill** (Mandatory) | `Devices` (`Serial_Number`) | `scripts/05_devices/1_Update_Device_Stage_onload.deluge`<br>`scripts/05_devices/5_1_Update_Device_Stage_onclick_submit.deluge` | References target devices in `Devices` to bulk update stage |
| **New Device Stage** | `New_Device_Stage` | Dropdown | **User Entry** (Mandatory; CU & OV manufacturing stages) | — | `scripts/05_devices/5_1_Update_Device_Stage_onclick_submit.deluge` | Target stage applied across selected devices with auto-harmonization |

---

## Stateless Utility: Update Device Custody (`Update_Device_Custody`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Select Devices** | `Select_Devices` | Lookup (Multi-Select) | **User Entry / Report Action Pre-fill** (Mandatory) | `Devices` (`Serial_Number`) | `scripts/05_devices/1_Update_Device_Custody_onload.deluge`<br>`scripts/05_devices/5_1_Update_Device_Custody_onclick_submit.deluge` | References target devices in `Devices` to bulk update location/custody |
| **Destination Type** | `Destination_Type` | Dropdown / Radio | **User Entry** (Mandatory; Choices: "Internal Office Transfer", "Customer Deployment") | — | `scripts/05_devices/2_1_Update_Device_Custody_onuserinput_destinationtype.deluge`<br>`scripts/05_devices/5_1_Update_Device_Custody_onclick_submit.deluge` | Toggles dynamic display of Office vs Customer lookup |
| **Target Office** | `Target_Office` | Lookup | **Dynamic User Entry** (Mandatory when Transfer is Office; hidden otherwise) | `Offices` (`Office_Name`) | `scripts/05_devices/1_Update_Device_Custody_onload.deluge`<br>`scripts/05_devices/2_1_Update_Device_Custody_onuserinput_destinationtype.deluge`<br>`scripts/05_devices/5_1_Update_Device_Custody_onclick_submit.deluge` | Sets `Devices.Current_Office` |
| **Target Customer** | `Target_Customer` | Lookup | **Dynamic User Entry** (Mandatory when Transfer is Customer; hidden otherwise) | `Customers` (`Customer_Name`) | `scripts/05_devices/1_Update_Device_Custody_onload.deluge`<br>`scripts/05_devices/2_1_Update_Device_Custody_onuserinput_destinationtype.deluge`<br>`scripts/05_devices/5_1_Update_Device_Custody_onclick_submit.deluge` | Sets `Devices.Current_Customer` |
