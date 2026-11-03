# CUREOUS PRS - MASTER DATA & DEPENDENCY DICTIONARY

This file serves as the **central cross-reference and dependency matrix** for the entire Cureous Patient Repositioning System (PRS) application in Zoho Creator.

Whenever any field, status, or logic is created, modified, or removed:
1. Search this document to identify every script and dependent module referencing it.
2. Update the target scripts and documentation.
3. Update this dictionary to keep the dependency matrix 100% accurate.

---

## High-Level Relational Entity Diagram

```
[Lots] (Module 1)
   │ (1:N)
   ▼
[Devices] (Module 5) ◄───── [Locations] (Module 3 - Warehouses/Branches)
   │
   ├──────────────────────────────┐
   │ (N:M Line Items)             │ (Dynamic Bedside Pairing)
   ▼                              ▼
[Stock_Transfers] (Module 7)   [Bed_Stations] (Module 8) ◄─── [Customers] (Module 2)
   │                              │                            ▲
   │ (Auto-generates)             ├───────────────┐            │
   ▼                              ▼               ▼            │
[Stock_Transfer_Items]    [Bed_Station_Log]   [Rental_Log] ────┤
                             (Module 9)        (Module 10)     │
                                                   │           │
                                                   ▼           │
                                           [Rental_Agreements]─┘
                                                (Module 6)
                                                   ▲
                                                   │
                                          [Product_Catalog]
                                              (Module 4)
```

---

## Module 1: Lots (`Lots`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Lot ID** | `Lot_ID` | Single Line | **User Entry** (Mandatory, Unique, PK; Locked in Edit) | — | `scripts/01_lots/1_Lots_onload.deluge`<br>`scripts/01_lots/3_Lots_onvalidate.deluge` | Module 5 (`Devices.Lot_Number`)<br>Module 11 (`Quality_Control.Lot_Number`) |
| **Total CUs Count** | `Total_CUs_Count` | Number | **Hidden on Add** (Report / Metric Only, Read-Only) | — | `scripts/01_lots/1_Lots_onload.deluge`<br>`scripts/01_lots/3_Lots_onvalidate.deluge` | Updated by `fn_recalculate_lot_metrics` |
| **Total Overlays Count** | `Total_Overlays_Count` | Number | **Hidden on Add** (Report / Metric Only, Read-Only) | — | `scripts/01_lots/1_Lots_onload.deluge`<br>`scripts/01_lots/3_Lots_onvalidate.deluge` | Updated by `fn_recalculate_lot_metrics` |
| **Total Units Count** | `Total_Units_Count` | Number | **Hidden on Add** (Report / Metric Only, Read-Only) | — | `scripts/01_lots/1_Lots_onload.deluge`<br>`scripts/01_lots/3_Lots_onvalidate.deluge` | Sum: Total CUs + Total Overlays |
| **CUs In Progress** | `CUs_In_Progress` | Number | **Hidden on Add** (Report / Metric Only, Read-Only) | — | `scripts/01_lots/1_Lots_onload.deluge`<br>`scripts/01_lots/3_Lots_onvalidate.deluge` | Updated by `fn_recalculate_lot_metrics` |
| **CUs Rent Ready** | `CUs_Rent_Ready` | Number | **Hidden on Add** (Report / Metric Only, Read-Only) | — | `scripts/01_lots/1_Lots_onload.deluge`<br>`scripts/01_lots/3_Lots_onvalidate.deluge` | Updated by `fn_recalculate_lot_metrics` |
| **Overlays In Progress** | `Overlays_In_Progress` | Number | **Hidden on Add** (Report / Metric Only, Read-Only) | — | `scripts/01_lots/1_Lots_onload.deluge`<br>`scripts/01_lots/3_Lots_onvalidate.deluge` | Updated by `fn_recalculate_lot_metrics` |
| **Overlays Rent Ready** | `Overlays_Rent_Ready` | Number | **Hidden on Add** (Report / Metric Only, Read-Only) | — | `scripts/01_lots/1_Lots_onload.deluge`<br>`scripts/01_lots/3_Lots_onvalidate.deluge` | Updated by `fn_recalculate_lot_metrics` |

---

## Module 2: Customers (`Customers`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Customer Name** | `Customer_Name` | Dropdown | **User Entry** (Mandatory, Synced Books; Locked in Edit) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge`<br>`scripts/02_customers/3_Customers_onvalidate.deluge` | Display label for Module 5, 6, 7, 8, 9, 10 |
| **Customer ID** | `Customer_ID` | Single Line | **Disabled Context** (PK / Books Contact ID, Read-Only) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge`<br>`scripts/02_customers/3_Customers_onvalidate.deluge` | Zoho Books API Invoicing (`Rental_Log`) |
| **Customer Type** | `Customer_Type` | Dropdown | **Disabled Context** (Read-Only: "Business", "Individual")| — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge` | Commercial contract categorization |
| **Billing Address** | `Billing_Address` | Multi-Line | **Disabled Context** (Read-Only, Synced Books) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/2_1_Customers_onuserinput_customername.deluge` | Reference on challans and agreements |
| **Customer Status** | `Customer_Status` | Dropdown | **Hidden on Add** (Report / Status Only, Read-Only) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/02_customers/3_Customers_onvalidate.deluge` | Active customer validation |
| **Active Control Unit #** | `Active_Control_Unit_Count` | Number | **Hidden on Add** (Report / Rollup Only, Read-Only) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/functions/fn_recalculate_customer_active_assets.deluge` | Rollup from Module 5 & 8 |
| **Active Overlay #** | `Active_Overlay_Count` | Number | **Hidden on Add** (Report / Rollup Only, Read-Only) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/functions/fn_recalculate_customer_active_assets.deluge` | Rollup from Module 5 & 8 |
| **Active Bed Stations #** | `Active_Bed_Stations_Count` | Number | **Hidden on Add** (Report / Rollup Only, Read-Only) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/functions/fn_recalculate_customer_active_assets.deluge` | Rollup from Module 8 (`Bed_Station`) |
| **Spare Units #** | `Spare_Units_Count` | Number | **Hidden on Add** (Report / Rollup Only, Read-Only) | — | `scripts/02_customers/1_Customers_onload.deluge`<br>`scripts/functions/fn_recalculate_customer_active_assets.deluge` | Emergency buffer units at site |

---

## Module 3: Locations (`Locations`)

| Field Label | Field Link Name | Type | Key / Constraint | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Location's Name** | `Location_s_Name` | Single Line | Mandatory, Unique | — | `scripts/03_locations/1_Locations_onload.deluge`<br>`scripts/03_locations/3_Locations_onvalidate.deluge` | Display label in `Devices.Current_Location` and `Stock_Transfers` |
| **Location ID** | `Location_ID` | Single Line | Mandatory, Unique (Books Warehouse ID) | — | `scripts/03_locations/1_Locations_onload.deluge`<br>`scripts/03_locations/3_Locations_onvalidate.deluge` | Passed to Zoho Books API for Delivery Challans & E-Way Bills |
| **Is Primary Hub** | `Is_Primary` | Decision (Checkbox) | Default: false | — | `scripts/03_locations/1_Locations_onload.deluge`<br>`scripts/03_locations/4_Locations_onsuccess.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge` | Sets default intake warehouse in Module 5 (`Bangalore HQ`) |

---

## Module 4: Product Catalog (`Product_Catalog`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Item Name** | `Item_Name` | Dropdown | **User Entry** (Mandatory, Synced Books; Locked in Edit) | — | `scripts/04_product_catalog/1_Product_Catalog_onload.deluge`<br>`scripts/04_product_catalog/2_1_Product_Catalog_onuserinput_itemname.deluge`<br>`scripts/04_product_catalog/3_Product_Catalog_onvalidate.deluge` | Display label for `Rental_Agreements.Rental_Item` |
| **Item ID** | `Item_ID` | Single Line | **Disabled Context** (PK / Books Item ID, Read-Only) | — | `scripts/04_product_catalog/1_Product_Catalog_onload.deluge`<br>`scripts/04_product_catalog/2_1_Product_Catalog_onuserinput_itemname.deluge` | Mapped to `Rental_Agreements.Books_Item_ID` and used in `Rental_Log` invoicing |
| **Product Model** | `Product_Model` | Single Line | **Disabled Context** (Read-Only) | — | `scripts/04_product_catalog/1_Product_Catalog_onload.deluge`<br>`scripts/04_product_catalog/2_1_Product_Catalog_onuserinput_itemname.deluge` | Auto-populates `Rental_Agreement.Product_Model` |
| **Billing Unit** | `Billing_Unit` | Single Line | **Disabled Context** (Read-Only: "Day", "Month") | — | `scripts/04_product_catalog/1_Product_Catalog_onload.deluge`<br>`scripts/04_product_catalog/2_1_Product_Catalog_onuserinput_itemname.deluge` | Auto-populates `Rental_Agreement.Billing_Unit` |

---

## Module 5: Devices (`Devices`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Serial Number** | `Serial_Number` | Single Line | **User Entry** (Mandatory, Unique, PK; Locked in Edit) | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Module 7 (`Stock_Transfers`), Module 8 (`Bed_Station`), Module 9, Module 10 |
| **Lot Number** | `Lot_Number` | Lookup | **User Entry** (Mandatory; Locked in Edit) | `Lots` (`Lot_ID`) | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Updates Module 1 rollup counters via `fn_recalculate_lot_metrics` |
| **Device Type** | `Device_Type` | Dropdown | **Disabled Context** (Read-Only: "Control Unit", "Overlay") | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge` | Auto-derived from serial string; filters lookups in Module 7 & 8 |
| **Device Model** | `Device_Model` | Single Line | **Disabled Context** (Read-Only: Altern4™, Eturnal One™) | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge` | Auto-derived from serial prefix |
| **Current Location** | `Current_Location` | Lookup | **Hidden on Add** (Report / Custody Only, Read-Only) | `Locations` (`Location_s_Name`) | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Auto-initialized to Primary Hub; updated by Module 7 (`Stock_Transfers`) |
| **Current Customer** | `Current_Customer` | Lookup | **Hidden on Add** (Report / Custody Only, Read-Only) | `Customers` (`Customer_Name`) | `scripts/05_devices/1_Devices_onload.deluge` | Updated by Module 7 on customer delivery / collection |
| **Total Cumulative Days Used** | `Total_Cumulative_Days_Used` | Number | **Hidden on Add** (Report / Metric Only, Read-Only, Default: 0) | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Updated by Module 10 (`Rental_Log`) |
| **Device Stage** | `Device_Stage` | Dropdown | **Hidden on Add** (Auto-init Stage 0; Enabled in Edit mode with dynamic choices) | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Stage gating for Module 7 dispatch and Module 11 QC |
| **Rental State** | `Rental_State` | Dropdown | **Hidden on Add** (Report / Status Only, Read-Only, Options: "Active", "Inactive") | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Updated by Module 8 (`Bed_Station`) on deployment |
| **Current Custody** | `Current_Custody` | Dropdown | **Hidden on Add** (Report / Custody Only, Read-Only, Options: "Internal Office", "In-Transit", "Customer's Site") | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Updated by Module 7 (`Stock_Transfers`) and Module 8 |

---

## Module 6: Rental Agreements (`Rental_Agreements`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Customer** | `Customer` | Lookup | **User Entry** (Mandatory; Locked in Edit) | `Customers` (`Customer_Name`) | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/2_2_Rental_Agreements_onuserinput_customer.deluge`<br>`scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge` | Anchors contract client |
| **Rental Item** | `Rental_Item` | Lookup | **User Entry** (Mandatory; Locked in Edit) | `Product_Catalog` (`Item_Name`) | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/2_1_Rental_Agreements_onuserinput_rentalitem.deluge`<br>`scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge` | Source for item pricing and Books item mapping |
| **Agreed Rate** | `Agreed_Rate` | Currency | **User Entry** (Mandatory, Editable) | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge` | Default rate pulled into Module 10 (`Rental_Log`) |
| **Agreement Title** | `Agreement_Title` | Single Line | **User Entry** (Auto-filled on Cust/Item select, Editable) | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/2_1_Rental_Agreements_onuserinput_rentalitem.deluge`<br>`scripts/06_rental_agreements/2_2_Rental_Agreements_onuserinput_customer.deluge` | Display label for Module 8, 9, 10 lookups |
| **Agreement Start Date** | `Agreement_Start_Date` | Date | **User Entry** (Auto-filled today, Editable) | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge` | Anchors recurring monthly billing cycles |
| **Billing Unit** | `Billing_Unit` | Dropdown | **Disabled Context** (Read-Only: "Day", "Month") | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/2_1_Rental_Agreements_onuserinput_rentalitem.deluge` | Governs whether agreement permits multiple beds (Day) or 1-to-1 (Month) |
| **Rental Agreement ID** | `Rental_Agreement_ID` | Single Line | **Hidden on Add** (Unique PK, generated in background) | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/2_1_Rental_Agreements_onuserinput_rentalitem.deluge`<br>`scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge` | Primary identifier in Module 8, 9, 10 |
| **Security Deposit** | `Security_Deposit` | Currency | **Hidden on Add** (Locked in Edit) | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge` | Commercial deposit tracking |
| **Agreement End Date** | `Agreement_End_Date` | Date | **Hidden on Add** (Filled on contract closure) | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge` | Contract termination guard |
| **Agreement Status** | `Agreement_Status` | Dropdown | **Hidden on Add** (Default: "Active" in background) | — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge` | Active deployment validation |
| **Active Deployed Stations** | `Active_Deployed_Stations` | Number | **Hidden on Add** (Report / Rollup Only, Read-Only, Default: 0)| — | `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`<br>`scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge` | Live counter updated on Module 8 (`Bed_Station`) |

---

## Module 7: Stock Transfers & Line Items (`Stock_Transfers`)

### Parent Form: `Stock_Transfers`
| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Transfer Type** | `Transfer_Type` | Dropdown | **User Entry** (Mandatory master trigger) | — | `scripts/07_stock_transfers/1_Stock_Transfers_onload.deluge`<br>`scripts/07_stock_transfers/2_1_Stock_Transfers_onuserinput_transfertype.deluge`<br>`scripts/07_stock_transfers/3_Stock_Transfers_onvalidate.deluge` | Master trigger controlling routing fields |
| **Transfer Date** | `Transfer_Date` | Date | **User Entry** (Mandatory, Default: today, Editable) | — | `scripts/07_stock_transfers/1_Stock_Transfers_onload.deluge` | Date of movement |
| **Source Office** | `Source_Office` | Lookup | **User Entry** (Visible when Inter Branch or Office to Cust)| `Locations` (`Location_s_Name`) | `scripts/07_stock_transfers/2_1_Stock_Transfers_onuserinput_transfertype.deluge`<br>`scripts/07_stock_transfers/2_2_Stock_Transfers_onuserinput_sourceoffice.deluge`<br>`scripts/07_stock_transfers/3_Stock_Transfers_onvalidate.deluge` | Filters available devices at originating warehouse |
| **Source Customer** | `Source_Customer` | Lookup | **User Entry** (Visible when Customer to Office) | `Customers` (`Customer_Name`) | `scripts/07_stock_transfers/2_1_Stock_Transfers_onuserinput_transfertype.deluge`<br>`scripts/07_stock_transfers/2_3_Stock_Transfers_onuserinput_sourcecustomer.deluge`<br>`scripts/07_stock_transfers/3_Stock_Transfers_onvalidate.deluge` | Filters devices at originating customer site |
| **Destination Office** | `Destination_Office` | Lookup | **User Entry** (Visible when Inter Branch or Cust to Office) | `Locations` (`Location_s_Name`) | `scripts/07_stock_transfers/2_1_Stock_Transfers_onuserinput_transfertype.deluge`<br>`scripts/07_stock_transfers/2_2_Stock_Transfers_onuserinput_sourceoffice.deluge`<br>`scripts/07_stock_transfers/3_Stock_Transfers_onvalidate.deluge`<br>`scripts/07_stock_transfers/4_Stock_Transfers_onsuccess.deluge` | Receiving branch warehouse |
| **Destination Customer**| `Destination_Customer`| Lookup | **User Entry** (Visible when Office to Customer) | `Customers` (`Customer_Name`) | `scripts/07_stock_transfers/2_1_Stock_Transfers_onuserinput_transfertype.deluge`<br>`scripts/07_stock_transfers/3_Stock_Transfers_onvalidate.deluge`<br>`scripts/07_stock_transfers/4_Stock_Transfers_onsuccess.deluge` | Receiving customer site |
| **Select Control Unit** | `Select_Control_Unit` | Multi-Select Lookup | **User Entry** (Visible once Source selected) | `Devices` (`Serial_Number`) | `scripts/07_stock_transfers/2_2_Stock_Transfers_onuserinput_sourceoffice.deluge`<br>`scripts/07_stock_transfers/2_3_Stock_Transfers_onuserinput_sourcecustomer.deluge`<br>`scripts/07_stock_transfers/2_4_Stock_Transfers_onuserinput_selectcontrolunit.deluge` | Populates child line items in `Stock_Transfer_Items` |
| **Select Overlay** | `Select_Overlay` | Multi-Select Lookup | **User Entry** (Visible once Source selected) | `Devices` (`Serial_Number`) | `scripts/07_stock_transfers/2_2_Stock_Transfers_onuserinput_sourceoffice.deluge`<br>`scripts/07_stock_transfers/2_3_Stock_Transfers_onuserinput_sourcecustomer.deluge`<br>`scripts/07_stock_transfers/2_5_Stock_Transfers_onuserinput_selectoverlay.deluge` | Populates child line items in `Stock_Transfer_Items` |
| **Control Unit #** | `Control_Unit_Number` | Number | **Disabled Context** (Auto-calculated count, Read-Only) | — | `scripts/07_stock_transfers/1_Stock_Transfers_onload.deluge`<br>`scripts/07_stock_transfers/2_4_Stock_Transfers_onuserinput_selectcontrolunit.deluge` | Recalculated on user selection |
| **Overlay #** | `Overlay_Number` | Number | **Disabled Context** (Auto-calculated count, Read-Only) | — | `scripts/07_stock_transfers/1_Stock_Transfers_onload.deluge`<br>`scripts/07_stock_transfers/2_5_Stock_Transfers_onuserinput_selectoverlay.deluge` | Recalculated on user selection |
| **Stock Transfer ID** | `Stock_Transfer_ID` | Auto-Number | **Auto-Hidden on Add** (PK; Natively locked in Edit) | — | — | Audit identifier |
| **Transfer Status** | `Transfer_Status` | Dropdown | **Hidden on Add** (Defaults to "Draft"; Editable in Edit) | — | `scripts/07_stock_transfers/1_Stock_Transfers_onload.deluge`<br>`scripts/07_stock_transfers/3_Stock_Transfers_onvalidate.deluge`<br>`scripts/07_stock_transfers/4_Stock_Transfers_onsuccess.deluge` | Triggers custody updates in Module 5 (`Devices`) |

### Child Form: `Stock_Transfer_Items`
| Field Label | Field Link Name | Type | Key / Constraint | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Transfer Item ID** | `Transfer_Item_ID` | Auto-Number | Primary Key | — | — | Audit identifier |
| **Stock Transfer Ref** | `Stock_Transfer_Ref` | Lookup | Parent Link | `Stock_Transfers` | `scripts/07_stock_transfers/4_Stock_Transfers_onsuccess.deluge` | Relational parent link |
| **Device Ref** | `Device_Ref` | Lookup | Read-Only | `Devices` (`Serial_Number`) | `scripts/07_stock_transfers/4_Stock_Transfers_onsuccess.deluge` | Foreign key to physical asset; displays Serial Number |
| **Device Type** | `Device_Type` | Single Line | Read-Only | — | `scripts/07_stock_transfers/4_Stock_Transfers_onsuccess.deluge` | Control Unit or Overlay |
| **Device Model** | `Device_Model` | Single Line | Read-Only | — | `scripts/07_stock_transfers/4_Stock_Transfers_onsuccess.deluge` | Hardware model family |

---

## Module 8: Bed Stations (`Bed_Stations`)

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Rental Agreement** | `Rental_Agreement` | Lookup | **User Entry** (Mandatory; Locked in Edit) | `Rental_Agreements` (`Agreement_Title`) | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_1_Bed_Stations_onuserinput_rentalagreement.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Auto-fills Customer and Product Model; updates agreement active beds |
| **Bed Label** | `Bed_Label` | Single Line | **User Entry** (Mandatory, e.g. "ICU Bed 04") | — | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_bedlabel.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge` | Used to compose `Bed_Station_Title` |
| **Bed Station Title** | `Bed_Station_Title` | Single Line | **User Entry** (Auto-filled, Editable) | — | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_1_Bed_Stations_onuserinput_rentalagreement.deluge`<br>`scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_bedlabel.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge` | Display label for Module 9 (`Bed_Station_Log`) & Module 10 (`Rental_Log`) |
| **Control Unit** | `Control_Unit` | Lookup | **User Entry** (Mandatory; Locked in Edit) | `Devices` (`Serial_Number`) | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Updates `Devices.Rental_State = "Active"` and `Devices.Current_Custody = "Customer's Site"` |
| **Overlay Asset** | `Overlay_Asset` | Lookup | **User Entry** (Mandatory; Locked in Edit) | `Devices` (`Serial_Number`) | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Updates `Devices.Rental_State = "Active"` and `Devices.Current_Custody = "Customer's Site"` |
| **Deployment Start Date**| `Deployment_Start_Date`| Date-Time | **User Entry** (Mandatory, Auto-filled, Editable) | — | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge` | Initial pairing timestamp |
| **Deployment Status** | `Deployment_Status` | Dropdown | **User Entry** (Mandatory: Active, Inactive, Closed) | — | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_3_Bed_Stations_onuserinput_deploymentstatus.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Triggers custody updates and contract counters |
| **Customer** | `Customer` | Lookup | **Disabled Context** (Auto-filled from Agreement) | `Customers` (`Customer_Name`) | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_1_Bed_Stations_onuserinput_rentalagreement.deluge`<br>`scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Client snapshot |
| **Product Model** | `Product_Model` | Single Line | **Disabled Context** (Auto-filled from Agreement) | — | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_1_Bed_Stations_onuserinput_rentalagreement.deluge` | Hardware family description |
| **Bed Station ID** | `Bed_Station_ID` | Single Line | **Disabled Context** (PK, e.g. `BST-2026-0001`) | — | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_1_Bed_Stations_onuserinput_rentalagreement.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge` | Primary identifier in Module 9, 10 |
| **Deployment End Date** | `Deployment_End_Date` | Date-Time | **Disabled Context** (Auto-filled on closure) | — | `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`<br>`scripts/08_bed_stations/2_3_Bed_Stations_onuserinput_deploymentstatus.deluge`<br>`scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge` | De-installation timestamp |

---

## Module 9: Bed Station Logs (`Bed_Station_Logs`)

| Field Label | Field Link Name | Type | Key / Constraint | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Bed Station Log ID** | `Bed_Station_Log_ID` | Auto-Number | Primary Key | — | — | Audit ledger identifier |
| **Bed Station** | `Bed_Station` | Lookup | Mandatory Foreign Key | `Bed_Stations` (`Bed_Station_Title`) | `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge`<br>`scripts/wizards/wizard_01_component_swap.deluge` | Clickable link to parent Bed Station |
| **Customer** | `Customer` | Lookup | Read-Only Context | `Customers` (`Customer_Name`) | `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Clickable link to customer account |
| **Rental Agreement** | `Rental_Agreement` | Lookup | Read-Only Context | `Rental_Agreements` (`Agreement_Title`) | `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge` | Clickable link to commercial contract |
| **Control Unit** | `Control_Unit` | Lookup | Read-Only Context | `Devices` (`Serial_Number`) | `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge`<br>`scripts/wizards/wizard_01_component_swap.deluge` | Clickable link to deployed Control Unit |
| **Overlay** | `Overlay` | Lookup | Read-Only Context | `Devices` (`Serial_Number`) | `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge`<br>`scripts/wizards/wizard_01_component_swap.deluge` | Clickable link to deployed Overlay mattress |
| **Period Start Date** | `Period_Start_Date` | Date-Time | Mandatory | — | `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge`<br>`scripts/wizards/wizard_01_component_swap.deluge` | Start timestamp of this exact hardware combination |
| **Period End Date** | `Period_End_Date` | Date-Time | Null while active | — | `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge`<br>`scripts/wizards/wizard_01_component_swap.deluge` | Closed on swap or station de-installation |
| **Duration Days** | `Duration_Days` | Decimal / Formula | Read-Only | — | — | Operating days during this specific pairing period |

---

## Module 10: Rental Log (`Rental_Log`)

| Field Label | Field Link Name | Type | Key / Constraint | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Rental Log ID** | `Rental_Log_ID` | Auto-Number | Primary Key (e.g. `RL-2026-0045`)| — | — | Billing log identifier |
| **Bed Station** | `Bed_Station` | Lookup | Mandatory | `Bed_Station` (`Bed_Station_Title`) | `docs/modules/10_rental_log.md` | Primary anchor; auto-fills Customer, Agreement, Serials |
| **Customer** | `Customer` | Lookup | Read-Only | `Customers` | `docs/modules/10_rental_log.md` | Client snapshot |
| **Rental Agreement** | `Rental_Agreement` | Lookup | Read-Only | `Rental_Agreements` | `docs/modules/10_rental_log.md` | Contract terms |
| **Rental Item** | `Rental_Item` | Lookup | Read-Only | `Product_Catalog` | `docs/modules/10_rental_log.md` | Zoho Books line item mapping |
| **Product Model** | `Product_Model` | Single Line | Read-Only | — | `docs/modules/10_rental_log.md` | Model description |
| **Rental Mode** | `Rental_Mode` | Single Line | Read-Only | — | `docs/modules/10_rental_log.md` | "Pay Per Use (Daily)" or "Fixed Monthly Rental" |
| **Control Unit Serial No** | `Control_Unit_Serial_No` | Single Line | Read-Only | — | `docs/modules/10_rental_log.md` | Printed on invoice line item |
| **Overlay Serial No** | `Overlay_Serial_No` | Single Line | Read-Only | — | `docs/modules/10_rental_log.md` | Printed on invoice line item |
| **Usage Start Date** | `Usage_Start_Date` | Date | Mandatory | — | `docs/modules/10_rental_log.md` | Billing start date |
| **Usage End Date** | `Usage_End_Date` | Date | Mandatory | — | `docs/modules/10_rental_log.md` | Billing end date |
| **Number of Days** | `Number_of_Days` | Number | Read-Only | — | `docs/modules/10_rental_log.md` | Inclusive days: `(End - Start) + 1` |
| **Agreed Rate** | `Agreed_Rate` | Currency | Mandatory | — | `docs/modules/10_rental_log.md` | Daily or Monthly rate from Agreement |
| **Discount Amount** | `Discount_Amount` | Currency | Default: 0.00 | — | `docs/modules/10_rental_log.md` | Subtracted from taxable base |
| **Taxable Value** | `Taxable_Value` | Currency | Read-Only | — | `docs/modules/10_rental_log.md` | Base amount before GST |
| **GST Rate (%)** | `GST_Rate` | Decimal | Fixed: 5.00% | — | `docs/modules/10_rental_log.md` | Statutory GST rate |
| **GST Amount** | `GST_Amount` | Currency | Read-Only | — | `docs/modules/10_rental_log.md` | `Taxable_Value * 0.05` |
| **Invoice Value** | `Invoice_Value` | Currency | Read-Only | — | `docs/modules/10_rental_log.md` | Total: `Taxable_Value + GST_Amount` |
| **Billing Status** | `Billing_Status` | Dropdown | Mandatory | — | `docs/modules/10_rental_log.md` | "Pending Invoice", "Invoiced", "Cancelled / Void" |
| **Zoho Books Invoice ID** | `Zoho_Books_Invoice_ID` | Single Line | Read-Only | — | `docs/modules/10_rental_log.md` | Books invoice identifier |
| **Zoho Books Invoice No** | `Zoho_Books_Invoice_No` | Single Line | Read-Only | — | `docs/modules/10_rental_log.md` | Books invoice number (e.g. `INV-2026-00341`) |
| **Invoice Date** | `Invoice_Date` | Date | Read-Only | — | `docs/modules/10_rental_log.md` | Invoicing date |
| **Invoice URL** | `Invoice_URL` | URL | Read-Only | — | `docs/modules/10_rental_log.md` | Direct link to view invoice in Zoho Books |

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
