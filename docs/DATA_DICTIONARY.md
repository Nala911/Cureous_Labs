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
                           ├───────────────┐            │
                           ▼               ▼            │
                         [Bed_Station_Logs] [Rental_Logs] ┘
                          (Module 9)       (Module 10)
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
| **Lot ID** | `Lot_ID` | Single Line | **User Entry** (Mandatory, Unique, PK; Locked in Edit) | — | `scripts/01_lots/1_Lots_onload.deluge`<br>`scripts/01_lots/3_Lots_onvalidate.deluge` | Module 5 (`Devices.Lot_Number`)<br>Module 11 (`Quality_Control.Lot_Number`) |

---

## Module 2: Customers (`Customers`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Customer Name** | `Customer_Name` | Dropdown | **User Entry** (Mandatory, Synced Books; Locked in Edit) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge`<br>`scripts/02_customers/3_Customers_onvalidate.deluge` | Display label for Module 5, 6, 7, 8, 9, 10 |
| **Customer ID** | `Customer_ID` | Single Line | **Disabled Context** (PK / Books Contact ID, Read-Only) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge`<br>`scripts/02_customers/3_Customers_onvalidate.deluge` | Zoho Books API Invoicing (`Rental_Logs`) |
| **Customer Type** | `Customer_Type` | Dropdown | **Disabled Context** (Read-Only: "Business", "Individual")| — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge` | Commercial contract categorization |
| **Billing Address** | `Billing_Address` | Multi-Line | **Disabled Context** (Read-Only, Synced Books) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge` | Reference on challans and agreements |
| **Customer Status** | `Customer_Status` | Dropdown | **Hidden on Add** (Report / Status Only, Read-Only) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/3_Customers_onvalidate.deluge` | Active customer validation |

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
| **Billing Unit** | `Billing_Unit` | Dropdown | **Auto-Derived & Editable** ("Day", "Month") | — | `scripts/04_rental_items/1_Rental_Items_onload.deluge`<br>`scripts/04_rental_items/2_1_Rental_Items_onuserinput_itemname.deluge`<br>`scripts/04_rental_items/3_Rental_Items_onvalidate.deluge` | Auto-populates `Rental_Agreement.Billing_Unit` |

---

## Module 5: Devices (`Devices`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Serial Number** | `Serial_Number` | Single Line | **User Entry** (Mandatory, Unique, PK; Locked in Edit) | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Module 7 (`Stock_Transfers`), Module 8 (`Bed_Station`), Module 9, Module 10 |
| **Lot Number** | `Lot_Number` | Lookup | **User Entry** (Mandatory; Locked in Edit) | `Lots` (`Lot_ID`) | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Auto-filled with latest lot on serial input (editable by user); parent manufacturing run traceability |
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
| **Agreement Title** | `Agreement_Title` | Single Line | **User Entry** (Auto-filled on Cust/Item select, Editable) | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/2_1_Rental_Agreements_onuserinput_rentalitem.deluge`<br>`scripts/06_rental_agreements/2_2_Rental_Agreements_onuserinput_customer.deluge`<br>`scripts/06_rental_agreements/4_Rental_Agreements_onsuccess.deluge` | Display label for Module 8, 9, 10 lookups |
| **Agreement Start Date** | `Agreement_Start_Date` | Date | **User Entry** (Auto-filled today, Editable) | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge` | Anchors recurring monthly billing cycles |
| **Billing Unit** | `Billing_Unit` | Dropdown | **Hidden on Add/Edit** (Auto-Derived Background Field: "Day", "Month") | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/2_1_Rental_Agreements_onuserinput_rentalitem.deluge`<br>`scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge` | Governs whether agreement permits multiple beds (Day) or 1-to-1 (Month) |
| **Rental Agreement ID** | `Rental_Agreement_ID` | Auto-Number | **Auto-Generated** (PK, Prefix `RA-`, e.g. `RA-0001`; Natively locked in Edit) | — | `scripts/06_rental_agreements/4_Rental_Agreements_onsuccess.deluge` | Primary identifier in Module 8, 9, 10 |
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
| **Rental Agreement** | `Rental_Agreement` | Lookup | **User Entry** (Mandatory when Deployment Type is "Rental", Hidden when "For Trial"; Locked in Edit) | `Rental_Agreements` (`Agreement_Title`) | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_1_Bed_Stations_onuserinput_deploymenttype.deluge`<br>`scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_rentalagreement.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Auto-fills Customer and Rental Item; validates contract limits |
| **Customer** | `Customer` | Lookup | **User Entry / Context** (Disabled on "Rental"; Enabled on "For Trial"; Locked in Edit) | `Customers` (`Customer_Name`) | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_1_Bed_Stations_onuserinput_deploymenttype.deluge`<br>`scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_rentalagreement.deluge`<br>`scripts/08_bed_stations/2_3_Bed_Stations_onuserinput_customer.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Filters Control Unit and Overlay lookup choices; client snapshot |
| **Bed Station Title** | `Bed_Station_Title` | Single Line | **User Entry** (Auto-prefilled from Customer/Contract, Editable, Appended with BST- ID on save) | - | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_rentalagreement.deluge`<br>`scripts/08_bed_stations/2_3_Bed_Stations_onuserinput_customer.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Display label for Module 9 (`Bed_Station_Log`) & Module 10 (`Rental_Logs`) |
| **Control Unit** | `Control_Unit` | Lookup | **User Entry** (Mandatory; Filtered to Customer's Site excluding Active Beds; Locked in Edit) | `Devices` (`Serial_Number`) | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_rentalagreement.deluge`<br>`scripts/08_bed_stations/2_3_Bed_Stations_onuserinput_customer.deluge`<br>`scripts/08_bed_stations/2_4_Bed_Stations_onuserinput_controlunit.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Populated dynamically from `Devices[Current_Customer == Customer]`; updates `Devices.Rental_State = "Active"` |
| **Overlay Asset** | `Overlay_Asset` | Lookup | **User Entry** (Mandatory; Filtered to Customer's Site excluding Active Beds; Locked in Edit) | `Devices` (`Serial_Number`) | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_rentalagreement.deluge`<br>`scripts/08_bed_stations/2_3_Bed_Stations_onuserinput_customer.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Populated dynamically from `Devices[Current_Customer == Customer]`; updates `Devices.Rental_State = "Active"` |
| **Deployment Start Date**| `Deployment_Start_Date`| Date-Time | **User Entry** (Mandatory, Auto-filled, Editable) | - | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge` | Initial pairing timestamp |
| **Deployment Status** | `Deployment_Status` | Dropdown | **User Entry** (Mandatory: Active, Inactive, Closed) | - | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_6_Bed_Stations_onuserinput_deploymentstatus.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Triggers custody updates and contract counters |
| **Rental Item** | `Rental_Item` | Single Line | **Disabled Context** (Auto-filled from Agreement in Rental, or from Control Unit in Trial) | - | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_rentalagreement.deluge`<br>`scripts/08_bed_stations/2_4_Bed_Stations_onuserinput_controlunit.deluge` | Hardware family description / Rental item name |
| **Bed Station ID** | `Bed_Station_ID` | Auto-Number | **Disabled Context** (PK, e.g. `BST-0001`) | - | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Primary identifier in Module 9, 10 |
| **Deployment End Date** | `Deployment_End_Date` | Date-Time | **Disabled Context** (Auto-filled on closure) | - | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_6_Bed_Stations_onuserinput_deploymentstatus.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge` | De-installation timestamp |

---

## Module 9: Bed Station Logs (`Bed_Station_Logs`)

| Field Label | Field Link Name | Type | Key / Constraint | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Bed Station Log ID** | `Bed_Station_Log_ID` | Auto-Number | Primary Key | — | — | Audit ledger identifier |
| **Bed Station** | `Bed_Station` | Lookup | Mandatory Foreign Key | `Bed_Stations` (`Bed_Station_Title`) | `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Clickable link to parent Bed Station |
| **Customer** | `Customer` | Lookup | Read-Only Context | `Customers` (`Customer_Name`) | `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Clickable link to customer account |
| **Rental Agreement** | `Rental_Agreement` | Lookup | Read-Only Context | `Rental_Agreements` (`Agreement_Title`) | `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Clickable link to commercial contract |
| **Control Unit** | `Control_Unit` | Lookup | Read-Only Context | `Devices` (`Serial_Number`) | `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Clickable link to deployed Control Unit |
| **Overlay** | `Overlay` | Lookup | Read-Only Context | `Devices` (`Serial_Number`) | `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Clickable link to deployed Overlay mattress |
| **Period Start Date** | `Period_Start_Date` | Date-Time | Mandatory | — | `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Start timestamp of this exact hardware combination |
| **Period End Date** | `Period_End_Date` | Date-Time | Null while active | — | `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Closed on swap or station de-installation |
| **Duration Days** | `Duration_Days` | Number | Read-Only | — | — | Operating days during this specific pairing period |

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

---

## Module 11: Quality Control & Sanitization (`Quality_Control`)

| Field Label | Field Link Name | Type | Key / Constraint | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Testing ID / QC ID** | `Testing_ID` | Auto-Number | Primary Key | — | — | Test certificate number |
| **Asset ID** | `Asset_ID` | Lookup | Mandatory | `Devices` (`Serial_Number`) | `docs/modules/11_qc_sanitization.md` | Direct link to physical unit |
| **Lot Number** | `Lot_Number` | Lookup | Read-Only | `Lots` (`Lot_ID`) | `docs/modules/11_qc_sanitization.md` | Auto-populated from Asset; links to parent production batch |
| **Asset Type** | `Asset_Type` | Single Line | Read-Only | — | `docs/modules/11_qc_sanitization.md` | Control Unit or Overlay |
| **Inspector / Tester** | `Inspector` | User Lookup | Mandatory | — | `docs/modules/11_qc_sanitization.md` | Logged in QA engineer |
| **Test Date & Time** | `Test_Date_Time` | Date-Time | Mandatory | — | `docs/modules/11_qc_sanitization.md` | Test timestamp |
| **QC Stage** | `QC_Stage` | Dropdown | Mandatory | — | `docs/modules/11_qc_sanitization.md` | "Post-Assembly", "Post-Rental Return", "Preventive Maintenance" |
| **Pump Pressure Output** | `Pump_Pressure_Output` | Decimal | CU Only (30 - 80 mmHg)| — | `docs/modules/11_qc_sanitization.md` | Physical pneumatic test |
| **Cycle Timing Check** | `Cycle_Timing_Check` | Decimal | CU Only (10/12 min) | — | `docs/modules/11_qc_sanitization.md` | Micro-controller cycle verification |
| **Alarm Test** | `Alarm_Test` | Dropdown | CU Only ("Pass", "Fail")| — | `docs/modules/11_qc_sanitization.md` | Audible / visual alarm safety |
| **Electrical Safety Test** | `Electrical_Safety_Test`| Dropdown | CU Only ("Pass", "Fail")| — | `docs/modules/11_qc_sanitization.md` | Ground continuity & leakage |
| **Cell 2-Hour Drop Test** | `Cell_Airtightness_Drop` | Dropdown | Overlay Only ("Pass", "Fail") | — | `docs/modules/11_qc_sanitization.md` | Puncture detection |
| **CPR Quick Deflate Test**| `CPR_Deflate_Test` | Dropdown | Overlay Only ("Pass", "Fail") | — | `docs/modules/11_qc_sanitization.md` | Emergency deflation (<15s) |
| **Cover Waterproof Check** | `Cover_Waterproof_Check`| Dropdown | Overlay Only | — | `docs/modules/11_qc_sanitization.md` | Biological barrier integrity |
| **Sanitization Status** | `Sanitization_Status` | Dropdown | Mandatory | — | `docs/modules/11_qc_sanitization.md` | Medical grade disinfection clearance |
| **Overall QC Verdict** | `Overall_QC_Verdict` | Dropdown | Mandatory | — | `docs/modules/11_qc_sanitization.md` | "Passed & Certified for Clinical Use", "Failed - Needs Repair", "Scrap" |
| **Next QC Due Date** | `Next_QC_Due_Date` | Date | Default: +180 days | — | `docs/modules/11_qc_sanitization.md` | Preventive maintenance scheduling |
