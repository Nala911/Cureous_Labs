# MODULE 8: BED STATIONS

**Form Name**: `Bed Stations`  
**Form Link Name**: `Bed_Stations`  
**Default Report**: `All_Bed_Stations`  
**Purpose**: Dynamic master representing ONE active Patient Bed Station (or clinical bed space) at a customer facility under either a commercial Rental Agreement or a Clinical Trial / Evaluation. Dynamically pairs 1 Control Unit and 1 Overlay at the patient bedside from the hardware assets physically present at that customer's site. Provides live status, current hardware deployment, and anchors all operational billing and audit history.

---

## 1. Design Principle: Intelligent Bedside Pairing

A complete Cureous Patient Repositioning System consists of two independent hardware components paired dynamically at the patient bedside:
- **Control Unit (CU)**: Electronic pump unit with micro-compressor and alternating cycle management.
- **Overlay (OV)**: Clinical alternating pressure mattress.

### Core Architecture:
1. **Dynamic Pairing**: Control Units and Overlays are never manufactured in fixed pairs. They are assembled independently, stocked independently, and dynamically paired in `Bed Stations`.
2. **Dual Deployment Modes**:
   - **Rental**: Anchored to a commercial contract (`Rental_Agreement`), which determines billing terms and automatically populates customer and product family.
   - **For Trial**: Used for clinical evaluations, demonstrations, and pilot trials where no commercial rental agreement exists. The user selects the `Customer` directly.
3. **Customer Site Hardware Filtering**: Rather than showing all manufactured devices globally, the `Control_Unit` and `Overlay_Asset` dropdowns dynamically query and show only devices currently located at that specific customer's facility (`Devices.Current_Customer == Customer`).
4. **Editable Auto-Defaults**: `Bed Station Title` and `Deployment Start Date` are auto-calculated and defaulted, while remaining fully editable by the field technician.
5. **Clean Scope**: Swapping workflows (Stateless Swap Wizard) and automated stock transfer pairing are isolated to keep the primary commissioning form ultra-clean.

---

## 2. Field Specifications

### A. User Interactive Fields:
1. **Deployment Type** `[Dropdown]` (Mandatory, Options: `"Rental"`, `"For Trial"`, Default: `"Rental"`)
   - Field Link Name: `Deployment_Type`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.
2. **Rental Agreement** `[Lookup -> Rental_Agreements, displaying Agreement_Title]` (Mandatory for Rental; Hidden for Trial)
   - Field Link Name: `Rental_Agreement`
   - Form Behavior: Visible & Enabled on Add when `Deployment_Type == "Rental"`; Hidden and cleared when `Deployment_Type == "For Trial"`. Locked in Edit mode.
3. **Customer** `[Lookup -> Customers, displaying Customer_Name]` (Auto-filled on Rental; User selectable on Trial)
   - Field Link Name: `Customer`
   - Form Behavior: Disabled on Add when `Deployment_Type == "Rental"` (auto-filled from agreement); Enabled on Add when `Deployment_Type == "For Trial"`. Locked in Edit mode.
4. **Bed Label** `[Single Line]` (Mandatory, e.g. "ICU Bed 04", "Ward 3 Bed 12")
   - Field Link Name: `Bed_Label`
   - Form Behavior: Visible & Enabled on Add and Edit.
5. **Bed Station Title** `[Single Line]` (Auto-filled from Customer + Bed Label, fully editable by user)
   - Field Link Name: `Bed_Station_Title`
   - Form Behavior: Visible & Enabled on Add and Edit.
6. **Control Unit** `[Lookup -> Devices, displaying Serial_Number]` (Mandatory; Filtered to Customer's site)
   - Field Link Name: `Control_Unit`
   - Form Behavior: Visible & Enabled on Add once Customer is determined; Locked in Edit mode. Dynamically populated with Control Units where `Current_Customer == Customer`.
7. **Overlay Asset** `[Lookup -> Devices, displaying Serial_Number]` (Mandatory; Filtered to Customer's site)
   - Field Link Name: `Overlay_Asset`
   - Form Behavior: Visible & Enabled on Add once Customer is determined; Locked in Edit mode. Dynamically populated with Overlays where `Current_Customer == Customer`.
8. **Deployment Start Date** `[Date-Time / Date]` (Mandatory; auto-filled with current timestamp, fully editable by user)
   - Field Link Name: `Deployment_Start_Date`
   - Form Behavior: Visible & Enabled on Add and Edit.
9. **Deployment Status** `[Dropdown]` (Mandatory)
   - Field Link Name: `Deployment_Status`
   - Options: `"Active"`, `"Inactive"`, `"Closed"` (Default: `"Active"`)
   - Form Behavior: Visible & Enabled on Add and Edit.

### B. Auto-Derived Context Fields (Disabled / System-Managed):
10. **Rental Item** `[Single Line]` (Auto-filled from Rental Agreement in Rental mode, or from Control Unit Device Model in Trial mode)
    - Field Link Name: `Rental_Item`
    - Form Behavior: Visible & Disabled on Add and Edit.
11. **Bed Station ID** `[Auto-Number - Native Auto-Hidden on Add]` (Primary Key, Prefix: BST-, e.g. `BST-0001`)
    - Field Link Name: `Bed_Station_ID`
    - Form Behavior: Auto-Hidden on Add (PK; Natively locked in Edit).
12. **Deployment End Date** `[Date-Time / Date]` (Auto-filled with current timestamp when status is set to "Closed")
    - Field Link Name: `Deployment_End_Date`
    - Form Behavior: Visible & Disabled on Add and Edit.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable interactive user entry fields (`Deployment_Type`, `Rental_Agreement`, `Bed_Label`, `Bed_Station_Title`, `Deployment_Start_Date`, `Deployment_Status`). Default `Deployment_Type = "Rental"`, `Deployment_Start_Date = zoho.currenttime`, `Deployment_Status = "Active"`. Disable context fields (`Customer`, `Rental_Item`, `Deployment_End_Date`). Disable hardware lookups (`Control_Unit`, `Overlay_Asset`) until customer/agreement is chosen.
  - In EDIT Mode: Lock primary keys, contract anchors, deployment mode, customer, and hardware pairings (`Deployment_Type`, `Rental_Agreement`, `Customer`, `Rental_Item`, `Control_Unit`, `Overlay_Asset`, `Deployment_End_Date`). If `Deployment_Type == "For Trial"`, hide `Rental_Agreement`. Allow updates to `Bed_Label`, `Bed_Station_Title`, `Deployment_Status`, and `Deployment_Start_Date`.

### Workflow 2: On User Input (Deployment Type)
- **File**: `scripts/08_bed_stations/2_1_Bed_Stations_onuserinput_deploymenttype.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Deployment_Type -> On User Input
- **Plain English Logic**:
  - When switched to `"Rental"`: Show and enable `Rental_Agreement`; show and disable `Customer`; clear `Rental_Agreement`, `Customer`, `Rental_Item`, `Control_Unit`, and `Overlay_Asset`; disable hardware fields until agreement is picked.
  - When switched to `"For Trial"`: Hide and disable `Rental_Agreement` and clear it to `null`; show and enable `Customer` for direct selection; clear `Customer`, `Rental_Item`, `Control_Unit`, and `Overlay_Asset`; disable hardware fields until customer is picked.
  - Re-evaluate `Bed_Station_Title`.

### Workflow 3: On User Input (Rental Agreement)
- **File**: `scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_rentalagreement.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Rental_Agreement -> On User Input
- **Plain English Logic**:
  - Step 1: Query linked `Rental_Agreements` record.
  - Step 2: Auto-populate `Customer` lookup and `Rental_Item` from agreement.
  - Step 3: Auto-compose default `Bed_Station_Title` (`[Customer] - [Bed Label]`), keeping it fully editable.
  - Step 4: Dynamically query `Devices` where `Current_Customer == input.Customer` and populate `Control_Unit` (filtered to `Device_Type == "Control Unit"`) and `Overlay_Asset` (filtered to `Device_Type == "Overlay"`). Enable both hardware lookups.
  - Step 5: If parent agreement is Fixed Monthly (`Billing_Unit == "Month"`), display non-blocking alert if an active Bed Station is already attached.

### Workflow 4: On User Input (Customer)
- **File**: `scripts/08_bed_stations/2_3_Bed_Stations_onuserinput_customer.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Customer -> On User Input
- **Plain English Logic**:
  - Step 1: When user selects `Customer` (in "For Trial" mode), dynamically refresh `Bed_Station_Title` (`[Customer] - [Bed Label]`).
  - Step 2: Dynamically query `Devices` where `Current_Customer == input.Customer`.
  - Step 3: Populate `Control_Unit` choices with Control Units located at this customer's site, and `Overlay_Asset` choices with Overlays located at this customer's site.
  - Step 4: Clear prior hardware selections and enable both lookups. If customer has no devices at their site, display a non-blocking notification.

### Workflow 5: On User Input (Control Unit)
- **File**: `scripts/08_bed_stations/2_4_Bed_Stations_onuserinput_controlunit.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Control_Unit -> On User Input
- **Plain English Logic**:
  - In "For Trial" mode (or if `Rental_Item` is blank), query selected `Devices` record and auto-populate `Rental_Item = cu_rec.Device_Model`.

### Workflow 6: On User Input (Bed Label)
- **File**: `scripts/08_bed_stations/2_5_Bed_Stations_onuserinput_bedlabel.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Bed_Label -> On User Input
- **Plain English Logic**:
  - Step 1: When user enters or modifies `Bed_Label` (e.g. "ICU Bed 04"), dynamically refresh `Bed_Station_Title` with format `[Customer] - [Bed Label]` (or append ID if in Edit mode).
  - Step 2: Retain full user editability of `Bed_Station_Title`.

### Workflow 7: On User Input (Deployment Status)
- **File**: `scripts/08_bed_stations/2_6_Bed_Stations_onuserinput_deploymentstatus.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Deployment_Status -> On User Input
- **Plain English Logic**:
  - Step 1: When user changes status to `"Closed"`, auto-populate `Deployment_End_Date` with `zoho.currenttime`.
  - Step 2: If status is reverted to `"Active"` or `"Inactive"`, clear `Deployment_End_Date = null`.

### Workflow 8: On Form Validation & Safety Guard
- **File**: `scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory validation on `Deployment_Type`, `Bed_Label`, `Control_Unit`, `Overlay_Asset`, `Deployment_Start_Date`, and `Deployment_Status`.
  - Step 2: Conditional contract check: If `Deployment_Type == "Rental"`, require `Rental_Agreement`. If `Deployment_Type == "For Trial"`, require `Customer`.
  - Step 3: Hardware classification check: Verify that `Control_Unit` has `Device_Type == "Control Unit"` and `Overlay_Asset` has `Device_Type == "Overlay"`.
  - Step 4: Customer site custody guard: Verify that both selected `Control_Unit` and `Overlay_Asset` currently reside at `input.Customer` site (`Current_Customer == input.Customer`).
  - Step 5: Duplicate active hardware guard: Ensure neither `Control_Unit` nor `Overlay_Asset` is already deployed on another active bed station.
  - Step 6: Fixed Monthly 1-to-1 enforcement: If `Deployment_Type == "Rental"` and agreement is Fixed Monthly, cancel submit if another active station already exists under that contract.
  - Step 7: Fallbacks: Ensure `Bed_Station_Title` is populated; set `Deployment_End_Date` if status is "Closed".

### Workflow 9: On Form Submission Success
- **File**: `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Update physical asset lifecycle in `Devices`:
    - If `Deployment_Status == "Active"`: Set `Rental_State = "Active"`, `Current_Custody = "Customer's Site"`, and `Current_Customer = input.Customer` for both Control Unit and Overlay.
    - If `Deployment_Status == "Closed"`: Set `Rental_State = "Inactive"`, `Current_Custody = "Customer's Site"` for both Control Unit and Overlay.
  - Step 2: If `Rental_Agreement` is present, atomically recalculate and update parent `Rental_Agreements.Active_Deployed_Stations` count.
  - Step 3: Maintain historical interval in `Bed_Station_Logs`:
    - If `Deployment_Status == "Active"`: Insert initial active log record into `Bed_Station_Logs` if one does not already exist (recording `Customer`, `Rental_Agreement` (or null), `Control_Unit`, `Overlay`, and `Period_Start_Date`).
    - If `Deployment_Status == "Closed"`: Close any open log record with `Period_End_Date` and calculate `Duration_Days`.
  - Step 4: Append generated `Bed_Station_ID` to `Bed_Station_Title` in database record if not already present.
