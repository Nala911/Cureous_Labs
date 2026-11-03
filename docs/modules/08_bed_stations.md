# MODULE 8: BED STATIONS

**Form Name**: `Bed Stations`  
**Form Link Name**: `Bed_Stations`  
**Default Report**: `All_Bed_Stations`  
**Purpose**: Dynamic master representing ONE active Patient Bed Station (or clinical bed space) at a customer facility under a Rental Agreement. Dynamically pairs 1 Control Unit and 1 Overlay at the patient bedside. Provides live status, current hardware deployment, and anchors all operational billing and audit history.

---

## 1. Design Principle: Intelligent Bedside Pairing

A complete Cureous Patient Repositioning System consists of two independent hardware components paired dynamically at the patient bedside:
- **Control Unit (CU)**: Electronic pump unit with micro-compressor and alternating cycle management.
- **Overlay (OV)**: Clinical alternating pressure mattress.

### Core Architecture:
1. **Dynamic Pairing**: Control Units and Overlays are never manufactured in fixed pairs. They are assembled independently, stocked independently, and dynamically paired in `Bed Stations`.
2. **Lean & Context-Aware Entry**: User enters `Rental Agreement`, `Bed Label`, and selects available hardware (`Control Unit` and `Overlay Asset`). Context fields (`Customer`, `Product Model`, `Bed Station ID`) are auto-populated and disabled to prevent typing overhead.
3. **Editable Auto-Defaults**: `Bed Station Title` and `Deployment Start Date` are auto-calculated and defaulted, while remaining fully editable by the field technician.
4. **Clean Scope**: Swapping workflows (Stateless Swap Wizard) and automated stock transfer pairing are isolated to keep the primary commissioning form ultra-clean.

---

## 2. Field Specifications

### A. User Interactive Fields (The ONLY fields user enters / edits):
1. **Rental Agreement** `[Lookup -> Rental_Agreements, displaying Agreement_Title]` (Mandatory)
   - Field Link Name: `Rental_Agreement`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.
2. **Bed Label** `[Single Line]` (Mandatory, e.g. "ICU Bed 04", "Ward 3 Bed 12")
   - Field Link Name: `Bed_Label`
   - Form Behavior: Visible & Enabled on Add and Edit.
3. **Bed Station Title** `[Single Line]` (Auto-filled from Customer + Bed Label + ID, fully editable by user)
   - Field Link Name: `Bed_Station_Title`
   - Form Behavior: Visible & Enabled on Add and Edit.
4. **Control Unit** `[Lookup -> Devices, displaying Serial_Number]` (Mandatory; renamed from `Current_Control_Unit_Asset`)
   - Field Link Name: `Control_Unit`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.
5. **Overlay Asset** `[Lookup -> Devices, displaying Serial_Number]` (Mandatory; renamed from `Current_Overlay_Asset`)
   - Field Link Name: `Overlay_Asset`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.
6. **Deployment Start Date** `[Date-Time / Date]` (Mandatory; auto-filled with current timestamp, fully editable by user)
   - Field Link Name: `Deployment_Start_Date`
   - Form Behavior: Visible & Enabled on Add and Edit.
7. **Deployment Status** `[Dropdown]` (Mandatory)
   - Field Link Name: `Deployment_Status`
   - Options: `"Active"`, `"Inactive"`, `"Closed"` (Default: `"Active"`)
   - Form Behavior: Visible & Enabled on Add and Edit.

### B. Auto-Derived Context Fields (Disabled - Provide Visual Context without Editing):
8. **Customer** `[Lookup -> Customers, displaying Customer_Name]` (Auto-filled from Rental Agreement)
   - Field Link Name: `Customer`
   - Form Behavior: Visible & Disabled on Add and Edit.
9. **Product Model** `[Single Line]` (Auto-filled from Rental Agreement / Product Catalog)
   - Field Link Name: `Product_Model`
   - Form Behavior: Visible & Disabled on Add and Edit.
10. **Bed Station ID** `[Single Line / Auto-Number]` (Primary Key, e.g. `BST-2026-0001`)
    - Field Link Name: `Bed_Station_ID`
    - Form Behavior: Visible & Disabled on Add and Edit.
11. **Deployment End Date** `[Date-Time / Date]` (Auto-filled with current timestamp when status is set to "Closed")
    - Field Link Name: `Deployment_End_Date`
    - Form Behavior: Visible & Disabled on Add and Edit.

### C. Redundant & Deferred Fields (Removed from Master Form):
- **Control Unit Serial No & Overlay Serial No** (old fields 9 & 10): Removed. Redundant snapshots since the `Control_Unit` and `Overlay_Asset` lookups directly reference and display the device serial numbers.
- **Last Changed Date** (old field 12): Removed. Simplifies form state; historical transition timestamps are maintained in `Bed_Station_Logs`.
- **Delivery Stock Transfer & Return Stock Transfer** (old fields 15 & 16): Removed. Direct warehouse movements and dispatch links are decoupled to be handled in dedicated logistics workflows.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable interactive user entry fields (`Rental_Agreement`, `Bed_Label`, `Bed_Station_Title`, `Control_Unit`, `Overlay_Asset`, `Deployment_Start_Date`, `Deployment_Status`). Default `Deployment_Start_Date = zoho.currenttime`, `Deployment_Status = "Active"`. Disable context fields (`Customer`, `Product_Model`, `Bed_Station_ID`, `Deployment_End_Date`). Auto-generate sequential preview for `Bed_Station_ID`.
  - In EDIT Mode: Lock primary keys, contract anchors, and hardware pairings (`Bed_Station_ID`, `Rental_Agreement`, `Customer`, `Product_Model`, `Control_Unit`, `Overlay_Asset`, `Deployment_End_Date`). Allow updates to `Bed_Label`, `Bed_Station_Title`, `Deployment_Status`, and `Deployment_Start_Date`.

### Workflow 2: On User Input (Rental Agreement)
- **File**: `scripts/08_bed_stations/2_1_Bed_Stations_onuserinput_rentalagreement.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Rental_Agreement -> On User Input
- **Plain English Logic**:
  - Step 1: Query linked `Rental_Agreements` record.
  - Step 2: Auto-populate `Customer` lookup and `Product_Model` from agreement.
  - Step 3: Ensure `Bed_Station_ID` is generated.
  - Step 4: Auto-compose default `Bed_Station_Title` (`[Customer] - [Bed Label] ([ID])`), keeping it fully editable.
  - Step 5: If parent agreement is Fixed Monthly (`Billing_Unit == "Month"`), display non-blocking alert if an active Bed Station is already attached.

### Workflow 3: On User Input (Bed Label)
- **File**: `scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_bedlabel.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Bed_Label -> On User Input
- **Plain English Logic**:
  - Step 1: When user enters or modifies `Bed_Label` (e.g. "ICU Bed 04"), dynamically refresh `Bed_Station_Title` with format `[Customer] - [Bed Label] ([ID])`.
  - Step 2: Retain full user editability of `Bed_Station_Title`.

### Workflow 4: On User Input (Deployment Status)
- **File**: `scripts/08_bed_stations/2_3_Bed_Stations_onuserinput_deploymentstatus.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Deployment_Status -> On User Input
- **Plain English Logic**:
  - Step 1: When user changes status to `"Closed"`, auto-populate `Deployment_End_Date` with `zoho.currenttime`.
  - Step 2: If status is reverted to `"Active"` or `"Inactive"`, clear `Deployment_End_Date = null`.

### Workflow 5: On Form Validation & Safety Guard
- **File**: `scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory validation on `Rental_Agreement`, `Bed_Label`, `Control_Unit`, `Overlay_Asset`, `Deployment_Start_Date`, and `Deployment_Status`.
  - Step 2: Hardware type integrity: Verify that `Control_Unit` has `Device_Type == "Control Unit"` and `Overlay_Asset` has `Device_Type == "Overlay"`.
  - Step 3: Duplicate active hardware guard: Ensure neither `Control_Unit` nor `Overlay_Asset` is already deployed on another active bed station.
  - Step 4: Fixed Monthly 1-to-1 enforcement: Cancel submit if agreement is Fixed Monthly and another active station already exists.
  - Step 5: Fallbacks: Ensure `Bed_Station_ID` and `Bed_Station_Title` are populated; set `Deployment_End_Date` if status is "Closed".

### Workflow 6: On Form Submission Success
- **File**: `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Update physical asset lifecycle in `Devices`:
    - If `Deployment_Status == "Active"`: Set `Rental_State = "Active"`, `Current_Custody = "Customer's Site"`, and `Current_Customer = input.Customer` for both Control Unit and Overlay.
    - If `Deployment_Status == "Closed"`: Set `Rental_State = "Inactive"`, `Current_Custody = "Customer's Site"` for both Control Unit and Overlay.
  - Step 2: Atomically recalculate and update parent `Rental_Agreements.Active_Deployed_Stations` count.
  - Step 3: Maintain historical interval in `Bed_Station_Logs`:
    - If `Deployment_Status == "Active"`: Insert initial active log record into `Bed_Station_Logs` if one does not already exist.
    - If `Deployment_Status == "Closed"`: Close any open log record with `Period_End_Date` and calculate `Duration_Days`.
  - Step 4: Submission Complete: Relies on Zoho Creator native form submission handling (eliminates modal redirect hangs).
