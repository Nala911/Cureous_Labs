# MODULE 7: STOCK TRANSFERS & LINE ITEM LEDGER

**Parent Form**: `Stock_Transfers`  
**Parent Link Name**: `Stock_Transfers`  
**Child Audit Form**: `Stock_Transfer_Items`  
**Child Link Name**: `Stock_Transfer_Items`  
**Default Report**: `All_Stock_Transfers`  
**Purpose**: Governs physical shipments, multi-unit branch transfers (e.g. Bangalore HQ to Nagpur Branch), customer dispatches (Rentals, Demos, Trials), and returns. Guarantees 100% serial-number custody tracking and provides backing data for Delivery Challans and E-Way Bills (Rule 55 CGST) without generating unwanted Transfer Orders in Zoho Books.

---

## 1. Design Principle: Ultra-Lean Progressive Disclosure
On load, only `Transfer Type` is enabled. Selecting the transfer type instantly activates and displays the exact directional routing pair needed. A unified multi-select hardware lookup allows batch selection of devices (Control Units and Overlays alike) in seconds, while Deluge automatically writes permanent itemized line items to `Stock_Transfer_Items` upon save.

---

## 2. Field Specifications

### A. Parent Form: `Stock_Transfers`

#### 1. User Interactive Fields (Progressive Disclosure):
1. **Transfer Type** `[Dropdown]` (Mandatory master trigger)
   - Field Link Name: `Transfer_Type`
   - Options: `"Inter Branch"`, `"Office to Customer"`, `"Customer to Office"`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.
2. **Transfer Date** `[Date]` (Mandatory, Default: today)
   - Field Link Name: `Transfer_Date`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.
3. **Source Office** `[Lookup -> Locations, displaying Location_s_Name]`
   - Field Link Name: `Source_Office`
   - Form Behavior: Visible & Enabled when Transfer Type is "Inter Branch" or "Office to Customer"; hidden otherwise.
4. **Source Customer** `[Lookup -> Customers, displaying Customer_Name]`
   - Field Link Name: `Source_Customer`
   - Form Behavior: Visible & Enabled when Transfer Type is "Customer to Office"; hidden otherwise.
5. **Destination Office** `[Lookup -> Locations, displaying Location_s_Name]`
   - Field Link Name: `Destination_Office`
   - Form Behavior: Visible & Enabled when Transfer Type is "Inter Branch" or "Customer to Office"; hidden otherwise.
6. **Destination Customer** `[Lookup -> Customers, displaying Customer_Name]`
   - Field Link Name: `Destination_Customer`
   - Form Behavior: Visible & Enabled when Transfer Type is "Office to Customer"; hidden otherwise.
7. **Select Devices** `[Multi-Select Lookup -> Devices, displaying Serial_Number]`
   - Field Link Name: `Select_Devices`
   - Form Behavior: Visible & Enabled once Source is selected; hidden on initial load.

#### 2. System & Lifecycle Fields:
8. **Stock Transfer ID** `[Auto-Number - Native Auto-Hidden on Add]` (Primary Key, Prefix: ST-, e.g. `ST-0001`)
   - Field Link Name: `Stock_Transfer_ID`
   - Form Behavior: Natively hidden on Add by Zoho Creator; automatically visible & locked in Edit mode and reports.
9. **Transfer Status** `[Dropdown]` (Mandatory lifecycle status, Default: `"Dispatched"`)
   - Field Link Name: `Transfer_Status`
   - Options: `"Dispatched"`, `"In-Transit"`, `"Delivered"`
   - Form Behavior: Visible & Enabled on Add (defaults to "Dispatched"); user can select "In-Transit" or "Delivered" during creation; Visible & Enabled in Edit mode for status updates.

### B. Child Audit Table: `Stock_Transfer_Items`
*(Populated automatically by Deluge On Success. Staff never enter data here manually).*
1. **Transfer Item ID** `[Auto-Number]` (Primary Key) -> `Transfer_Item_ID`
2. **Stock Transfer Ref** `[Lookup -> Stock_Transfers]` -> `Stock_Transfer_Ref`
3. **Device Ref** `[Lookup -> Devices, displaying Serial_Number]` -> `Device_Ref`
4. **Device Type** `[Single Line - Read Only]` -> `Device_Type`
5. **Device Model** `[Single Line - Read Only]` -> `Device_Model`

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/07_stock_transfers/1_Stock_Transfers_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable `Transfer_Type` and `Transfer_Date` (defaulted to `zoho.currentdate`). Enable `Transfer_Status` (visible and defaulted to `"Dispatched"`). (Note: `Stock_Transfer_ID` is an Auto-Number field, so Zoho Creator hides it natively). Hide and disable directional routing fields and `Select_Devices` lookup until triggered by user input of Transfer Type.
  - In EDIT Mode: Lock primary routing parameters and `Select_Devices`. Allow updates to `Transfer_Status`.

### Workflow 2: On User Input (Step 1: Transfer Type)
- **File**: `scripts/07_stock_transfers/2_1_Stock_Transfers_onuserinput_transfertype.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Transfer_Type -> On User Input
- **Plain English Logic**:
  - Step 1: Reset previously selected devices (`Select_Devices = null`). Hide and disable `Select_Devices`.
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
  - Reset previous device selections (`Select_Devices = null`).
  - Dynamically query `Devices` in a single indexed query and populate `Select_Devices` via `:ui.add()` with all available units (Control Units and Overlays) physically located at the selected Source Office (or held at `Source_Customer`). Reveal and enable `Select_Devices`.

### Workflow 4: On Form Validation & Safety Guard
- **File**: `scripts/07_stock_transfers/3_Stock_Transfers_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Mandatory validation on `Transfer_Type`, Source, Destination, and `Transfer_Status` (defaults to `"Dispatched"` if blank; enforces valid choices: Dispatched, In-Transit, Delivered).
  - Conflict check: For Inter Branch, `Source_Office != Destination_Office`.
  - Inventory check: Verify that at least one device is selected (`Select_Devices != null && Select_Devices.size() > 0`).

### Workflow 5: On Form Submission Success
- **File**: `scripts/07_stock_transfers/4_Stock_Transfers_onsuccess.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Collect selected device serial numbers from `Select_Devices` in Add mode (or fallback to existing `Stock_Transfer_Items` in Edit mode).
  - Step 2: Query `Devices` using Serial Number.
  - Step 3: Insert itemized audit rows into `Stock_Transfer_Items` only if not previously created for this transfer (preventing duplicates on status edit updates). Automatically populate `Device_Type` and `Device_Model` from the device record.
  - Step 4: Update device physical custody and location in `Devices` based on `Transfer_Status` and `Transfer_Type`:
    - Dispatched: `Current_Custody = "Dispatched"`, `Current_Location = null`, `Current_Customer = null`, `Rental_State = "Inactive"`.
    - In-Transit: `Current_Custody = "In-Transit"`, `Current_Location = null`, `Current_Customer = null`, `Rental_State = "Inactive"`.
    - Inter Branch (Delivered): `Current_Location = Destination_Office`, `Current_Customer = null`, `Current_Custody = "Internal Office"`, `Rental_State = "Inactive"`.
    - Office to Customer (Delivered): `Current_Customer = Destination_Customer`, `Current_Location = null`, `Current_Custody = "Customer's Site"`, `Rental_State = "Inactive"`.
    - Customer to Office (Delivered): `Current_Location = Destination_Office`, `Current_Customer = null`, `Current_Custody = "Internal Office"`, `Rental_State = "Inactive"`.
  - Step 5: Post-Submission Navigation: Relies on Zoho Creator's native Form Properties (**Form Properties -> General -> On Successful Submission -> Redirect to Report / Display Message**) to dismiss modal dialogs and refresh views without client-side Deluge script hangs.
