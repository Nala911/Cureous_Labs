# CUREOUS PRS - MASTER DATA & DEPENDENCY DICTIONARY

This file serves as the **central cross-reference and dependency matrix** for the entire Cureous Patient Repositioning System (PRS) application in Zoho Creator.

Whenever any field, status, or logic is created, modified, or removed:
1. Search this document to identify every script and dependent module referencing it.
2. Update the target scripts and documentation.
3. Update this dictionary to keep the dependency matrix 100% accurate.

---

## High-Level Relational Entity Diagram

```
[Product_Model] (Master)
   │ (1:N)           │ (1:N)
   ▼                 ▼
[Devices] (Module 5) [Rental_Items] (Module 4)
   ▲                       │
   │ (1:N)                 │
[Lots] (Module 1)          ▼
   │                 [Rental_Agreements] (Module 6)
   ▼                       │
[Devices] ◄── [Offices]    │
   │          (Module 3)   │
   ├───────────────────────┼──────┐
   │ (Multi-Select Lookup) │      │ (Dynamic Bedside Pairing)
   ▼                       ▼      ▼
[Stock_Transfers] (M7)   [Bed_Stations] (Module 8) ◄─── [Customers] (Module 2)
                           │                            ▲
                           └───────────────┐            │
                                           ▼            │
                                     [Rental_Logs] ─────┘
                                      (Module 10)
```

---

## Module: Product Model (`Product_Model`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Product Model Name** | `Product_Model_Name` | Single Line | **User Entry** (Mandatory, Unique, PK Display; Enabled on Add/Edit) | — | `scripts/product_models/1_Product_Model_onload.deluge`<br>`scripts/product_models/3_Product_Model_onvalidate.deluge` | Display label for Module 4 (`Rental_Items.Product_Model`) and Module 5 (`Devices.Product_Model`) |

---

## Module 1: Lots (`Lots`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Lot ID** | `Lot_ID` | Single Line | **User Entry** (Mandatory, Unique, PK; Locked in Edit) | — | `scripts/01_lots/1_Lots_onload.deluge`<br>`scripts/01_lots/3_Lots_onvalidate.deluge` | Module 5 (`Devices.Lot`)<br>Module 11 (`Quality_Control.Lot_Number`) |

---

## Module 2: Customers (`Customers`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Customer Name** | `Customer_Name` | Dropdown | **User Entry** (Mandatory, Synced Books; Locked in Edit) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge`<br>`scripts/02_customers/3_Customers_onvalidate.deluge` | Display label for Module 5, 6, 7, 8, 9, 10 |
| **Customer ID** | `Customer_ID` | Number | **Hidden Background Field** (PK / Books Contact ID, Read-Only) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge`<br>`scripts/02_customers/3_Customers_onvalidate.deluge` | Zoho Books API Invoicing (`Rental_Logs`) |
| **Customer Type** | `Customer_Type` | Dropdown | **Disabled Context** (Read-Only: "Business", "Individual")| — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge` | Commercial contract categorization |
| **Billing Address** | `Billing_Address` | Multi-Line | **Disabled Context** (Read-Only, Synced Books) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge` | Reference on challans and agreements |

---

## Module 3: Offices (`Offices`)

| Field Label | Field Link Name | Type | Key / Constraint | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Office Name** | `Office_Name` | Single Line | Mandatory, Unique | — | `scripts/03_offices/1_Offices_onload.deluge`<br>`scripts/03_offices/3_Offices_onvalidate.deluge` | Display label in `Devices.Current_Office` and `Stock_Transfers` |
| **Is Primary Hub** | `Is_Primary` | Decision (Checkbox) | Default: false | — | `scripts/03_offices/1_Offices_onload.deluge`<br>`scripts/03_offices/4_Offices_onsuccess.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge` | Sets default intake warehouse in Module 5 (`Bangalore HQ`) |

---

## Module 4: Rental Items (`Rental_Items`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Item Name** | `Item_Name` | Dropdown | **User Entry** (Mandatory, Synced Books / Standard Choices; Locked in Edit) | — | `scripts/04_rental_items/1_Rental_Items_onload.deluge`<br>`scripts/04_rental_items/2_1_Rental_Items_onuserinput_itemname.deluge`<br>`scripts/04_rental_items/3_Rental_Items_onvalidate.deluge` | Display label for `Rental_Agreements.Rental_Item` |
| **Product Model** | `Product_Model` | Lookup | **Auto-Derived & Editable** (`Altern4` or `Eturnal`) | `Product_Model` (`Product_Model_Name`) | `scripts/04_rental_items/1_Rental_Items_onload.deluge`<br>`scripts/04_rental_items/2_1_Rental_Items_onuserinput_itemname.deluge`<br>`scripts/04_rental_items/3_Rental_Items_onvalidate.deluge` | Links hardware model family |
| **Billing Unit** | `Billing_Unit` | Dropdown | **Auto-Derived & Editable** ("Day", "Month") | — | `scripts/04_rental_items/1_Rental_Items_onload.deluge`<br>`scripts/04_rental_items/2_1_Rental_Items_onuserinput_itemname.deluge`<br>`scripts/04_rental_items/3_Rental_Items_onvalidate.deluge` | Defines billing cadence; referenced directly by `Bed_Stations` and `Rental_Logs` |

---

## Module 5: Devices (`Devices`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Serial Number** | `Serial_Number` | Single Line | **User Entry** (Mandatory, Unique, PK; Locked in Edit) | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Module 7 (`Stock_Transfers`), Module 8 (`Bed_Station`), Module 9, Module 10 |
| **Lot** | `Lot` | Lookup | **User Entry** (Mandatory; Locked in Edit) | `Lots` (`Lot_ID`) | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Auto-filled with latest lot on serial input (editable by user); parent manufacturing run traceability |
| **Device Type** | `Device_Type` | Dropdown | **Disabled Context** (Read-Only: "Control Unit", "Overlay") | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge` | Auto-derived from serial string; filters lookups in Module 7 & 8 |
| **Product Model** | `Product_Model` | Lookup | **Disabled Context** (Read-Only: Altern4, Altern8, EturnalOne, Eturnal One with foam) | `Product_Model` (`Product_Model_Name`) | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Auto-derived from serial prefix |
| **Current Office** | `Current_Office` | Lookup | **Hidden on Add** (Report / Custody Only, Read-Only) | `Offices` (`Office_Name`) | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Auto-initialized to Primary Hub; updated by Module 7 (`Stock_Transfers`) |
| **Current Customer** | `Current_Customer` | Lookup | **Hidden on Add** (Report / Custody Only, Read-Only) | `Customers` (`Customer_Name`) | `scripts/05_devices/1_Devices_onload.deluge` | Updated by Module 7 on customer delivery / collection |
| **Device Stage** | `Device_Stage` | Dropdown | **Hidden on Add** (Auto-init Stage 0: "0 - CU Mfg Not Yet Started" / "0 - OV Mfg Not Yet Started"; Enabled in Edit mode with dynamic choices) | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/2_2_Devices_onupdate_devicestage.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Auto-harmonized on bulk edit; Stage gating for Module 7 dispatch and Module 11 QC |
| **Rental State** | `Rental_State` | Dropdown | **Hidden on Add** (Report / Status Only, Read-Only, Options: "Active", "Inactive") | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Updated by Module 8 (`Bed_Station`) on deployment |
| **Current Custody** | `Current_Custody` | Lookup | **Hidden on Add** (Report / Custody Only, Read-Only in Edit, Blank on Add) | `Stock_Transfers` (`Transfer_Status`) | `scripts/05_devices/1_Devices_onload.deluge` | Relational pointer to latest `Stock_Transfers` record; displays current `Transfer_Status` |

---

## Module 6: Rental Agreements (`Rental_Agreements`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Customer** | `Customer` | Lookup | **User Entry** (Mandatory; Locked in Edit) | `Customers` (`Customer_Name`) | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/2_2_Rental_Agreements_onuserinput_customer.deluge`<br>`scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge` | Anchors contract client |
| **Rental Item** | `Rental_Item` | Lookup | **User Entry** (Mandatory; Locked in Edit) | `Rental_Items` (`Item_Name`) | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/2_1_Rental_Agreements_onuserinput_rentalitem.deluge`<br>`scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge` | Source for item pricing and Books item mapping |
| **Agreed Rate** | `Agreed_Rate` | Currency | **User Entry** (Mandatory, Editable) | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge` | Default rate pulled into Module 10 (`Rental_Logs`) |
| **Agreement Title** | `Agreement_Title` | Single Line | **User Entry** (Unique Business PK; Auto-filled on Cust/Item select as `[First 3 words of Cust] - [Clean Item] #[Count+1]`, Editable, Uniqueness validated on submit) | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/2_1_Rental_Agreements_onuserinput_rentalitem.deluge`<br>`scripts/06_rental_agreements/2_2_Rental_Agreements_onuserinput_customer.deluge`<br>`scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge` | Display label for Module 8 and 10 lookups |
| **Agreement Start Date** | `Agreement_Start_Date` | Date | **User Entry** (Auto-filled today, Editable) | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge` | Anchors recurring monthly billing cycles |
| **Security Deposit** | `Security_Deposit` | Currency | **Hidden on Add** (Locked in Edit) | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge` | Commercial deposit tracking |
| **Agreement End Date** | `Agreement_End_Date` | Date | **Hidden on Add** (Filled on contract closure) | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge` | Contract termination guard |
| **Agreement Status** | `Agreement_Status` | Dropdown | **Hidden on Add** (Default: "Active" in background) | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge` | Active deployment validation |

---

## Module 7: Stock Transfers (`Stock_Transfers`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Transfer Type** | `Transfer_Type` | Dropdown | **User Entry** (Mandatory master trigger) | — | `scripts/07_stock_transfers/1_Stock_Transfers_onload.deluge`<br>`scripts/07_stock_transfers/2_1_Stock_Transfers_onuserinput_transfertype.deluge`<br>`scripts/07_stock_transfers/3_Stock_Transfers_onvalidate.deluge` | Master trigger controlling routing fields |
| **Transfer Date** | `Transfer_Date` | Date | **User Entry** (Mandatory, Default: today, Editable) | — | `scripts/07_stock_transfers/1_Stock_Transfers_onload.deluge` | Date of movement |
| **Source Office** | `Source_Office` | Lookup | **User Entry** (Visible when Inter Branch or Office to Cust)| `Offices` (`Office_Name`) | `scripts/07_stock_transfers/2_1_Stock_Transfers_onuserinput_transfertype.deluge`<br>`scripts/07_stock_transfers/2_2_Stock_Transfers_onuserinput_sourceoffice.deluge`<br>`scripts/07_stock_transfers/3_Stock_Transfers_onvalidate.deluge` | Filters available devices at originating warehouse |
| **Source Customer** | `Source_Customer` | Lookup | **User Entry** (Visible when Customer to Office) | `Customers` (`Customer_Name`) | `scripts/07_stock_transfers/2_1_Stock_Transfers_onuserinput_transfertype.deluge`<br>`scripts/07_stock_transfers/2_3_Stock_Transfers_onuserinput_sourcecustomer.deluge`<br>`scripts/07_stock_transfers/3_Stock_Transfers_onvalidate.deluge` | Filters devices at originating customer site |
| **Destination Office** | `Destination_Office` | Lookup | **User Entry** (Visible when Inter Branch or Cust to Office) | `Offices` (`Office_Name`) | `scripts/07_stock_transfers/2_1_Stock_Transfers_onuserinput_transfertype.deluge`<br>`scripts/07_stock_transfers/2_2_Stock_Transfers_onuserinput_sourceoffice.deluge`<br>`scripts/07_stock_transfers/3_Stock_Transfers_onvalidate.deluge`<br>`scripts/07_stock_transfers/4_Stock_Transfers_onsuccess.deluge` | Receiving branch warehouse |
| **Destination Customer**| `Destination_Customer`| Lookup | **User Entry** (Visible when Office to Customer) | `Customers` (`Customer_Name`) | `scripts/07_stock_transfers/2_1_Stock_Transfers_onuserinput_transfertype.deluge`<br>`scripts/07_stock_transfers/3_Stock_Transfers_onvalidate.deluge`<br>`scripts/07_stock_transfers/4_Stock_Transfers_onsuccess.deluge` | Receiving customer site |
| **Devices** | `Devices` | Multi-Select Lookup | **User Entry** (Visible once Source selected; Office-to-Customer restricted to `Ready Condition`) | `Devices` (`Serial_Number`) | `scripts/07_stock_transfers/2_2_Stock_Transfers_onuserinput_sourceoffice.deluge`<br>`scripts/07_stock_transfers/2_3_Stock_Transfers_onuserinput_sourcecustomer.deluge`<br>`scripts/07_stock_transfers/3_Stock_Transfers_onvalidate.deluge`<br>`scripts/07_stock_transfers/4_Stock_Transfers_onsuccess.deluge` | Direct multi-unit batch selection; synchronizes hardware ledger in `Devices` |
| **Stock Transfer ID** | `Stock_Transfer_ID` | Auto-Number | **Auto-Hidden on Add** (PK; Natively locked in Edit) | — | — | Audit identifier |
| **Transfer Status** | `Transfer_Status` | Dropdown | **User Entry** (Mandatory, Default: "In-Transit", Options: "In-Transit", "In-Office", "With-Customer") | — | `scripts/07_stock_transfers/1_Stock_Transfers_onload.deluge`<br>`scripts/07_stock_transfers/3_Stock_Transfers_onvalidate.deluge`<br>`scripts/07_stock_transfers/4_Stock_Transfers_onsuccess.deluge` | Triggers custody updates in Module 5 (`Devices`) |

---

## Module 8: Bed Stations (`Bed_Stations`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Deployment Type** | `Deployment_Type` | Dropdown | **User Entry** (Mandatory, Default: "Rental", Options: "Rental", "For Trial"; Locked in Edit) | - | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_1_Bed_Stations_onuserinput_deploymenttype.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge` | Determines commercial contract vs evaluation workflow |
| **Rental Agreement** | `Rental_Agreement` | Lookup | **User Entry** (Mandatory when Deployment Type is "Rental", Hidden when "For Trial"; Locked in Edit) | `Rental_Agreements` (`Agreement_Title`) | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_1_Bed_Stations_onuserinput_deploymenttype.deluge`<br>`scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_rentalagreement.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Auto-fills Customer; validates contract limits |
| **Customer** | `Customer` | Lookup | **User Entry / Context** (Disabled on "Rental"; Enabled on "For Trial"; Locked in Edit) | `Customers` (`Customer_Name`) | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_1_Bed_Stations_onuserinput_deploymenttype.deluge`<br>`scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_rentalagreement.deluge`<br>`scripts/08_bed_stations/2_3_Bed_Stations_onuserinput_customer.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Filters Control Unit and Overlay lookup choices; client snapshot |
| **Bed Station Title** | `Bed_Station_Title` | Single Line | **User Entry** (Unique Business PK; Auto-filled on Agreement/Cust select as `[First 3 words of Cust/Agr] #[Count+1]`, Editable, Uniqueness validated on submit) | - | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_rentalagreement.deluge`<br>`scripts/08_bed_stations/2_3_Bed_Stations_onuserinput_customer.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge` | Display label for Module 10 (`Rental_Logs`) |
| **Control Unit** | `Control_Unit` | Lookup | **User Entry** (Mandatory; Filtered to Customer's Site; Swappable in Edit) | `Devices` (`Serial_Number`) | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_rentalagreement.deluge`<br>`scripts/08_bed_stations/2_3_Bed_Stations_onuserinput_customer.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Populated dynamically from `Devices[Current_Customer == Customer]`; updates `Devices.Rental_State = "Active"` |
| **Overlay Asset** | `Overlay_Asset` | Lookup | **User Entry** (Mandatory; Filtered to Customer's Site; Swappable in Edit) | `Devices` (`Serial_Number`) | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_rentalagreement.deluge`<br>`scripts/08_bed_stations/2_3_Bed_Stations_onuserinput_customer.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Populated dynamically from `Devices[Current_Customer == Customer]`; updates `Devices.Rental_State = "Active"` |
| **Deployment Start Date**| `Deployment_Start_Date`| Date-Time | **User Entry** (Mandatory, Auto-filled, Editable) | - | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge` | Initial pairing timestamp |
| **Deployment Status** | `Deployment_Status` | Dropdown | **Hidden on Add** (Default: "Active"; Enabled in Edit: Active, Inactive, Closed) | - | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_4_Bed_Stations_onuserinput_deploymentstatus.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Triggers custody updates and contract counters |
| **Deployment End Date** | `Deployment_End_Date` | Date-Time | **Hidden on Add** (Auto-filled on closure in Edit mode) | - | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_4_Bed_Stations_onuserinput_deploymentstatus.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge` | De-installation timestamp |
| **Swap Hardware?** | `Swap` | Decision Box | **Hidden on Add** (Default: false / unchecked; Visible in Edit). Triggers Hardware Swap Mode on form submission. | - | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_5_Bed_Stations_onuserinput_swap.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge` | Distinguishes operational hardware swaps from administrative edits |
| **Swap Reason** | `Swap_Reason` | Single Line | **Hidden on Add** (Hidden by default in Edit; Visible & Mandatory when Swap == true). Documents defect reason. | - | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_5_Bed_Stations_onuserinput_swap.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge` | Captures failure reason (e.g. error code E-02) for biomedical maintenance |

---

## Module 10: Rental Logs (`Rental_Logs`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Rental Log ID** | `Rental_Log_ID` | Auto-Number | **Disabled Context** (PK, Prefix `RL-`) | — | `docs/modules/10_rental_logs.md` | Primary Key billing log identifier |
| **Bed Station** | `Bed_Station` | Lookup | **User Entry** (Mandatory) | `Bed_Stations` (`Bed_Station_Title`) | `scripts/10_rental_logs/1_Rental_Logs_onload.deluge`<br>`scripts/10_rental_logs/2_1_Rental_Logs_onuserinput_bedstation.deluge`<br>`scripts/10_rental_logs/3_Rental_Logs_onvalidate.deluge` | Primary relational anchor; links Customer, Agreement, Serials |
| **Control Unit** | `Control_Unit` | Lookup | **User Entry** (Mandatory, Auto-filled, Editable) | `Devices` (`Serial_Number`) | `scripts/10_rental_logs/1_Rental_Logs_onload.deluge`<br>`scripts/10_rental_logs/2_1_Rental_Logs_onuserinput_bedstation.deluge`<br>`scripts/10_rental_logs/3_Rental_Logs_onvalidate.deluge` | Active Control Unit paired to bed |
| **Overlay** | `Overlay` | Lookup | **User Entry** (Mandatory, Auto-filled, Editable) | `Devices` (`Serial_Number`) | `scripts/10_rental_logs/1_Rental_Logs_onload.deluge`<br>`scripts/10_rental_logs/2_1_Rental_Logs_onuserinput_bedstation.deluge`<br>`scripts/10_rental_logs/3_Rental_Logs_onvalidate.deluge` | Active Overlay paired to bed |
| **Usage Start Date** | `Usage_Start_Date` | Date | **User Entry** (Mandatory) | — | `scripts/10_rental_logs/1_Rental_Logs_onload.deluge`<br>`scripts/10_rental_logs/2_2_Rental_Logs_onuserinput_dates.deluge`<br>`scripts/10_rental_logs/3_Rental_Logs_onvalidate.deluge` | Billing start date |
| **Usage End Date** | `Usage_End_Date` | Date | **User Entry** (Mandatory) | — | `scripts/10_rental_logs/1_Rental_Logs_onload.deluge`<br>`scripts/10_rental_logs/2_2_Rental_Logs_onuserinput_dates.deluge`<br>`scripts/10_rental_logs/3_Rental_Logs_onvalidate.deluge` | Billing end date |
| **Agreed Rate** | `Agreed_Rate` | Currency | **User Entry** (Mandatory, Auto-filled, Editable) | — | `scripts/10_rental_logs/1_Rental_Logs_onload.deluge`<br>`scripts/10_rental_logs/2_1_Rental_Logs_onuserinput_bedstation.deluge`<br>`scripts/10_rental_logs/2_3_Rental_Logs_onuserinput_agreedrate.deluge`<br>`scripts/10_rental_logs/3_Rental_Logs_onvalidate.deluge` | Daily or Monthly rate from Agreement |
| **Number of Days** | `Number_of_Days` | Number | **User Entry / Auto-Computed** (Editable) | — | `scripts/10_rental_logs/1_Rental_Logs_onload.deluge`<br>`scripts/10_rental_logs/2_2_Rental_Logs_onuserinput_dates.deluge`<br>`scripts/10_rental_logs/3_Rental_Logs_onvalidate.deluge` | Inclusive days: `(End - Start) + 1` |
| **Taxable Value** | `Taxable_Value` | Currency | **Auto-Computed Context** (Read-Only) | — | `scripts/10_rental_logs/1_Rental_Logs_onload.deluge`<br>`scripts/10_rental_logs/2_2_Rental_Logs_onuserinput_dates.deluge`<br>`scripts/10_rental_logs/2_3_Rental_Logs_onuserinput_agreedrate.deluge` | Base amount before GST |
| **GST Amount** | `GST_Amount` | Formula | **Native Formula** (`Taxable_Value * 0.05`) | — | — | Native Formula Field; 5% GST value |
| **Invoice Amount** | `Invoice_Amount` | Formula | **Native Formula** (`Taxable_Value + GST_Amount`) | — | — | Native Formula Field; Grand total invoice value |
| **Billing Status** | `Billing_Status` | Dropdown | **Disabled Context** (Default: "Pending Invoice") | — | `scripts/10_rental_logs/1_Rental_Logs_onload.deluge`<br>`scripts/10_rental_logs/3_Rental_Logs_onvalidate.deluge` | "Pending Invoice", "Invoiced", "Cancelled / Void" |
