# MODULE 7: STOCK TRANSFERS

**Form Name**: `Stock_Transfers`  
**Form Link Name**: `Stock_Transfers`  
**Default Report**: `All_Stock_Transfers`  
**Purpose**: Governs physical shipments, multi-unit branch transfers (e.g. Bangalore HQ to Nagpur Branch), customer dispatches (Rentals, Demos, Trials), and returns. Guarantees 100% serial-number custody tracking and provides backing data for Delivery Challans and E-Way Bills (Rule 55 CGST) without generating unwanted Transfer Orders in Zoho Books.

---

## 1. Design Principle: Ultra-Lean Single-Form Architecture
On load, only `Transfer Type` and `Transfer Date` are enabled. Selecting the transfer type instantly activates and displays the exact directional routing pair needed. A multi-select hardware lookup field (`Devices`) allows batch selection of devices (Control Units and Overlays alike) in seconds. By operating directly through the native multi-select lookup without intermediate child tables, inventory synchronization is instantaneous, statement-efficient, and eliminates orphaned records.

---

## 2. Field Specifications

### 1. User Interactive Fields (Progressive Disclosure):
1. **Transfer Type** `[Dropdown]` (Mandatory master trigger)
   - Field Link Name: `Transfer_Type`
   - Options: `"Inter Branch"`, `"Office to Customer"`, `"Customer to Office"`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.
2. **Transfer Date** `[Date]` (Mandatory, Default: today)
   - Field Link Name: `Transfer_Date`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.
3. **Source Office** `[Lookup -> Offices, displaying Office_Name]`
   - Field Link Name: `Source_Office`
   - Form Behavior: Visible & Enabled when Transfer Type is "Inter Branch" or "Office to Customer"; hidden otherwise.
4. **Source Customer** `[Lookup -> Customers, displaying Customer_Name]`
   - Field Link Name: `Source_Customer`
   - Form Behavior: Visible & Enabled when Transfer Type is "Customer to Office"; hidden otherwise.
5. **Destination Office** `[Lookup -> Offices, displaying Office_Name]`
   - Field Link Name: `Destination_Office`
   - Form Behavior: Visible & Enabled when Transfer Type is "Inter Branch" or "Customer to Office"; hidden otherwise.
6. **Destination Customer** `[Lookup -> Customers, displaying Customer_Name]`
   - Field Link Name: `Destination_Customer`
   - Form Behavior: Visible & Enabled when Transfer Type is "Office to Customer"; hidden otherwise.
7. **Devices** `[Multi-Select Lookup -> Devices, displaying Serial_Number]`
   - Field Link Name: `Devices`
   - Form Behavior: Visible & Enabled once Source is selected; hidden on initial load; locked in Edit mode.

### 2. System & Lifecycle Fields:
8. **Stock Transfer ID** `[Auto-Number - Native Auto-Hidden on Add]` (Primary Key, Prefix: ST-, e.g. `ST-0001`)
   - Field Link Name: `Stock_Transfer_ID`
   - Form Behavior: Natively hidden on Add by Zoho Creator; automatically visible & locked in Edit mode and reports.
9. **Transfer Status** `[Dropdown]` (Mandatory lifecycle status, Default: `"In-Transit"`)
   - Field Link Name: `Transfer_Status`
   - Options: `"In-Transit"`, `"In-Office"`, `"With-Customer"`
   - Form Behavior: Visible & Enabled on Add (defaults to "In-Transit"); Visible & Enabled in Edit mode for status updates upon receipt.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/07_stock_transfers/1_Stock_Transfers_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable `Transfer_Type` and `Transfer_Date` (defaulted to `zoho.currentdate`). Enable `Transfer_Status` (visible and defaulted to `"In-Transit"`). (Note: `Stock_Transfer_ID` is an Auto-Number field, so Zoho Creator hides it natively). Hide and disable directional routing fields and `Devices` lookup until triggered by user input of Transfer Type.
  - In EDIT Mode: Lock primary routing parameters and `Devices`. Allow updates to `Transfer_Status`.

### Workflow 2: On User Input (Step 1: Transfer Type)
- **File**: `scripts/07_stock_transfers/2_1_Stock_Transfers_onuserinput_transfertype.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Transfer_Type -> On User Input
- **Plain English Logic**:
  - Step 1: Reset previously selected devices (`Devices = null`). Hide and disable `Devices`.
  - Step 2: Route directional fields dynamically:
    - `"Inter Branch"`: Show & enable `Source_Office` & `Destination_Office`; hide & disable Customer fields.
    - `"Office to Customer"`: Show & enable `Source_Office` & `Destination_Customer`; hide & disable others.
    - `"Customer to Office"`: Show & enable `Source_Customer` & `Destination_Office`; hide & disable others.

### Workflow 3: On User Input (Step 2: Source Office / Source Customer)
- **Files**:
  - `scripts/07_stock_transfers/2_2_Stock_Transfers_onuserinput_sourceoffice.deluge`
  - `scripts/07_stock_transfers/2_3_Stock_Transfers_onuserinput_sourcecustomer.deluge`
- **Plain English Logic**:
  - For `Source_Office`: If `Transfer_Type == "Inter Branch"`, filter `Destination_Office` dropdown choices to exclude the chosen Source Office (and clear Destination if identical).
  - Reset previous device selections (`Devices = null`).
  - Dynamically query `Devices` in a single indexed query:
    - If `Transfer_Type == "Office to Customer"`, enforce **Clinical Safety Guard**: Only units in **`Ready Condition`** (`6 - CU Ready Condition` or `6 - OV Ready Condition`) are populated in `Devices` via `:ui.add()`, preventing defective, un-assembled, or contaminated hardware from shipping to customer hospitals.
    - If `Transfer_Type == "Inter Branch"`, all warehouse devices (`Rental_State == "Inactive"`) are available (facilitating inter-facility maintenance transfers).
  - Reveal and enable `Devices`.

### Workflow 4: On Form Validation & Safety Guard
- **File**: `scripts/07_stock_transfers/3_Stock_Transfers_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Mandatory validation on `Transfer_Type`, Source, Destination, and `Transfer_Status` (defaults to `"In-Transit"` if blank; enforces valid choices: `In-Transit`, `In-Office`, `With-Customer`).
  - Directional parity check: `Office to Customer` cannot be marked `In-Office`; `Inter Branch` and `Customer to Office` cannot be marked `With-Customer`.
  - Conflict check: For Inter Branch, `Source_Office != Destination_Office`.
  - Inventory check: Verify that at least one device is selected (`Devices != null && Devices.size() > 0`).
  - Clinical Safety Validation: For `Office to Customer` transfers, verifies every selected device has `Device_Stage` in `Ready Condition`. Cancels submission if any defective or uncertified device is detected.

### Workflow 5: On Form Submission Success
- **File**: `scripts/07_stock_transfers/4_Stock_Transfers_onsuccess.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Iterate native multi-select lookup collection `input.Devices` directly.
  - Step 2: Loop through selected devices, stamp `Current_Custody` with the transfer record ID (`dev.Current_Custody = input.ID;`), and synchronize physical office and customer based on `Transfer_Status`:
    - `In-Transit`: `Current_Office = null`, `Current_Customer = null`, `Rental_State = "Inactive"`.
    - `In-Office`: `Current_Office = Destination_Office`, `Current_Customer = null`, `Rental_State = "Inactive"`.
    - `With-Customer`: `Current_Customer = Destination_Customer`, `Current_Office = null`, `Rental_State = "Inactive"`.
  - Step 3: Redirect cleanly to `All_Stock_Transfers` report view (`openUrl("#Report:All_Stock_Transfers", "same window");`).
