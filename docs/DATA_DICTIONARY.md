# CUREOUS PRS - MASTER DATA & DEPENDENCY DICTIONARY

This file serves as the **central cross-reference and dependency matrix** for the entire Cureous Patient Repositioning System (PRS) application in Zoho Creator.

Whenever any field, status, or logic is created, modified, or removed:
1. Search this document to identify every script and dependent module referencing it.
2. Update the target scripts and documentation.
3. Update this dictionary to keep the dependency matrix 100% accurate.

---

## High-Level Relational Entity Diagram

```
[Devices] (Module 5)     [Rental_Items] (Module 4)
                               ▲
                               │
                         [Customers] (Module 2)
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
| **Product Model** | `Product_Model` | Dropdown | **Auto-Derived & Editable on Add; Locked in Edit** ("Altern4", "Eturnal One", "Eturnal One with foam") | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Hardware model classification |
| **Lot** | `Lot` | Dropdown | **Auto-Default Latest & User Editable on Add and Edit** (e.g. "26AB02", "26FB03") | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Manufacturing lot tracking |
| **Current Customer** | `Current_Customer` | Lookup | **Hidden on Add** (Report / Allocation Only, Read-Only) | `Customers` (`Customer_Name`) | `scripts/05_devices/1_Devices_onload.deluge` | Direct customer allocation |
| **Current Office** | `Current_Office` | Lookup | **Hidden on Add** (Report / Location Only, Read-Only) | `Offices` (`Office_Name`) | `scripts/05_devices/1_Devices_onload.deluge` | Direct branch / warehouse custody tracking |
| **Device Stage** | `Device_Stage` | Dropdown | **Hidden on Add** (Auto-init Stage 0: "0 - CU Mfg Not Yet Started" / "0 - OV Mfg Not Yet Started"; Enabled in Edit mode with dynamic choices) | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/2_2_Devices_onupdate_devicestage.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Auto-harmonized on bulk edit; Stage gating for manufacturing and Module 11 QC |
| **Rental State** | `Rental_State` | Dropdown | **Hidden on Add** (Report / Status Only, Read-Only, Options: "Active", "Inactive") | — | `scripts/05_devices/1_Devices_onload.deluge`<br>`scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Updated on field deployment (`Active` on patient deployment, `Inactive` when in warehouse/buffer) |
