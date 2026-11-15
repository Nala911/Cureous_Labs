# MODULE 8: BED STATIONS

**Form Name**: `Bed Stations`  
**Form Link Name**: `Bed_Stations`  
**Default Report**: `All_Bed_Stations`  
**Purpose**: Dynamic master representing ONE active Patient Bed Station (or clinical bed space) at a customer facility under either a commercial Rental Agreement or a Clinical Trial / Evaluation. Dynamically pairs 1 Control Unit and 1 Overlay at the patient bedside from the hardware assets physically present at that customer's site. Provides live status, current hardware deployment, and anchors all operational billing and audit history.

---

## 1. Core Operating Principles
1. **Dynamic Bedside Pairing**: Hardware is never locked into permanent pairs. Any compatible Control Unit and Overlay sitting at a hospital can be paired onto a bed station in seconds.
2. **Native Bedside Swapping**: If a Control Unit or Overlay fails or requires maintenance, staff can swap the component directly in Edit mode by selecting a replacement buffer unit already present at that customer site. The system seamlessly marks the broken/replaced unit `Rental_State = "Inactive"` (sitting in customer storage awaiting return) and marks the newly attached unit `Rental_State = "Active"`.
3. **Two Deployment Pathways**:
   - **Rental Mode**: Anchored to a commercial `Rental_Agreements` contract. Customer and billing model flow from the agreement.
   - **For Trial Mode**: Deployed for clinical evaluation before any contract is signed. User selects the Customer directly.
4. **Auto-Composed & Editable Title**: `Bed_Station_Title` automatically pre-fills with smart auto-cleaned customer context (e.g. `[Clean Customer] - Bed`), remaining fully editable by staff for custom ward, room, or bed designations (e.g. "Dhanya Ortho - ICU Bed 4"). On submission, ` #BST-0001` is appended automatically for distinct audit identification.
5. **Physical Custody Decoupling**: Physical custody remains linked to the dispatch `Stock_Transfers` record (`With-Customer`). Module 8 exclusively manages the operational pairing state via `Rental_State = "Active"` or `"Inactive"`.

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
4. **Bed Station Title** `[Single Line]` (Auto-filled with Customer context, fully editable by user)
   - Field Link Name: `Bed_Station_Title`
   - Form Behavior: Visible & Enabled on Add and Edit.
5. **Control Unit** `[Lookup -> Devices, displaying Serial_Number]` (Mandatory; Filtered to Customer's site; Swappable in Edit)
   - Field Link Name: `Control_Unit`
   - Form Behavior: Visible & Enabled on Add and Edit. In Add mode, enabled once Customer is determined. In Edit mode, enabled for bedside swaps when `Deployment_Status == "Active"`. Populated with Control Units located at this customer's site (excluding units deployed on other active bed stations).
6. **Overlay Asset** `[Lookup -> Devices, displaying Serial_Number]` (Mandatory; Filtered to Customer's site; Swappable in Edit)
   - Field Link Name: `Overlay_Asset`
   - Form Behavior: Visible & Enabled on Add and Edit. In Add mode, enabled once Customer is determined. In Edit mode, enabled for bedside swaps when `Deployment_Status == "Active"`. Populated with Overlays located at this customer's site (excluding units deployed on other active bed stations).
7. **Deployment Start Date** `[Date-Time / Date]` (Mandatory; auto-filled with current timestamp, fully editable by user)
   - Field Link Name: `Deployment_Start_Date`
   - Form Behavior: Visible & Enabled on Add and Edit.
8. **Deployment Status** `[Dropdown - Hidden on Add]` (Default: `"Active"`. Options: `"Active"`, `"Inactive"`, `"Closed"`)
   - Field Link Name: `Deployment_Status`
   - Form Behavior: Hidden on Add (defaults to `"Active"`); Visible & Enabled in Edit mode.

### B. Auto-Derived Context Fields (Disabled / System-Managed):
9. **Bed Station ID** `[Auto-Number - Native Auto-Hidden on Add]` (Primary Key, Prefix: BST-, e.g. `BST-0001`)
   - Field Link Name: `Bed_Station_ID`
   - Form Behavior: Auto-Hidden on Add (PK; Natively locked in Edit).
10. **Deployment End Date** `[Date-Time - Hidden on Add]` (Auto-filled with current timestamp when status is set to "Closed")
    - Field Link Name: `Deployment_End_Date`
    - Form Behavior: Hidden on Add; Visible & Disabled in Edit mode.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/08_bed_stations/1_Bed_Stations_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable interactive user entry fields (`Deployment_Type`, `Rental_Agreement`, `Bed_Station_Title`, `Deployment_Start_Date`). Default `Deployment_Type = "Rental"`, `Deployment_Start_Date = zoho.currenttime`, and background default `Deployment_Status = "Active"`. Hide intake lifecycle fields (`Deployment_Status`, `Deployment_End_Date`). Disable context field (`Customer`). Disable hardware lookups (`Control_Unit`, `Overlay_Asset`) until customer/agreement is chosen.
  - In EDIT Mode: Lock primary keys, contract anchors, deployment mode, customer, and closure date (`Deployment_Type`, `Rental_Agreement`, `Customer`, `Deployment_End_Date`). If `Deployment_Type == "For Trial"`, hide `Rental_Agreement`. Allow updates to `Bed_Station_Title`, `Deployment_Status`, and `Deployment_Start_Date`. Show `Deployment_Status` and `Deployment_End_Date`.
    - If `Deployment_Status == "Active"`: Enable `Control_Unit` and `Overlay_Asset` to allow bedside hardware swaps. Dynamically query `Devices` at `input.Customer` site that are not deployed on other active bed stations (`ID != input.ID`) plus the currently paired devices, and populate both choice lists via UI tasks.
    - If `Deployment_Status == "Closed"`: Keep `Control_Unit` and `Overlay_Asset` disabled.

### Workflow 2: On User Input (Deployment Type)
- **File**: `scripts/08_bed_stations/2_1_Bed_Stations_onuserinput_deploymenttype.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Deployment_Type -> On User Input
- **Plain English Logic**:
  - When switched to `"Rental"`: Show and enable `Rental_Agreement`; show and disable `Customer`; clear `Rental_Agreement`, `Customer`, `Control_Unit`, and `Overlay_Asset`; disable hardware fields until agreement is picked.
  - When switched to `"For Trial"`: Hide and disable `Rental_Agreement` and clear it to `null`; show and enable `Customer` for direct selection; clear `Customer`, `Control_Unit`, and `Overlay_Asset`; disable hardware fields until customer is picked.

### Workflow 3: On User Input (Rental Agreement)
- **File**: `scripts/08_bed_stations/2_2_Bed_Stations_onuserinput_rentalagreement.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Rental_Agreement -> On User Input
- **Plain English Logic**:
  - Step 1: Query linked `Rental_Agreements` record.
  - Step 2: Auto-populate `Customer` lookup from agreement.
  - Step 3: Auto-prefill `Bed_Station_Title` with smart auto-cleaned customer context (`[Clean Customer] - Bed`) if blank, keeping it fully editable.
  - Step 4: Smart Bedside Hardware Retrieval: Query `Devices` located at `input.Customer` site. Exclude any devices already actively deployed on other `Active` Bed Stations at this facility. Populate `Control_Unit` choices with available Control Units and `Overlay_Asset` choices with available Overlays. Enable both hardware lookups.
  - Step 5: If parent agreement is Fixed Monthly (`Billing_Unit == "Month"`), display non-blocking alert if an active Bed Station is already attached.

### Workflow 4: On User Input (Customer)
- **File**: `scripts/08_bed_stations/2_3_Bed_Stations_onuserinput_customer.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Customer -> On User Input
- **Plain English Logic**:
  - Step 1: When user selects `Customer` (in "For Trial" mode), dynamically pre-fill `Bed_Station_Title` with smart auto-cleaned customer context (`[Clean Customer] - Bed`) if blank, keeping it editable.
  - Step 2: Smart Bedside Hardware Retrieval: Dynamically query `Devices` where `Current_Customer == input.Customer`. Exclude any devices already actively deployed on other `Active` Bed Stations.
  - Step 3: Populate `Control_Unit` choices with unassigned Control Units located at this customer's site, and `Overlay_Asset` choices with unassigned Overlays located at this customer's site.
  - Step 4: Clear prior hardware selections and enable both lookups. If customer has no available buffer units at their site, display a non-blocking notification.

### Workflow 5: On User Input (Deployment Status)
- **File**: `scripts/08_bed_stations/2_4_Bed_Stations_onuserinput_deploymentstatus.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Deployment_Status -> On User Input
- **Plain English Logic**:
  - Step 1: When user changes status to `"Closed"`, auto-populate `Deployment_End_Date` with `zoho.currenttime`.
  - Step 2: If status is reverted to `"Active"` or `"Inactive"`, clear `Deployment_End_Date = null`.

### Workflow 6: On Form Validation & Safety Guard
- **File**: `scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory validation on `Deployment_Type`, `Bed_Station_Title`, `Control_Unit`, `Overlay_Asset`, `Deployment_Start_Date`, and `Deployment_Status`.
  - Step 2: Conditional contract check: If `Deployment_Type == "Rental"`, require `Rental_Agreement`. If `Deployment_Type == "For Trial"`, require `Customer`.
  - Step 3: Hardware classification check: Verify that `Control_Unit` has `Device_Type == "Control Unit"` and `Overlay_Asset` has `Device_Type == "Overlay"`.
  - Step 4: Customer site custody guard: Verify that both selected `Control_Unit` and `Overlay_Asset` currently reside at `input.Customer` site (`Current_Customer == input.Customer`).
  - Step 5: Duplicate active hardware guard: Ensure neither `Control_Unit` nor `Overlay_Asset` is already deployed on another active bed station (excluding `ID != input.ID` in Edit mode).
  - Step 6: Fixed Monthly 1-to-1 enforcement: If `Deployment_Type == "Rental"` and agreement is Fixed Monthly, cancel submit if another active station already exists under that contract (excluding `ID != input.ID`).
  - Step 7: Fallbacks: Ensure `Bed_Station_Title` is populated (applying smart auto-clean fallback if blank); set `Deployment_End_Date` if status is "Closed".

### Workflow 7: On Form Submission Success
- **File**: `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Update physical asset lifecycle in `Devices`:
    - If `Deployment_Status == "Active"`: Set `Rental_State = "Active"` and `Current_Customer = input.Customer` for both `input.Control_Unit` and `input.Overlay_Asset`.
    - Native Hardware Swap Reconciliation: Query devices at this customer site that were previously active on this bed station but are no longer selected (`ID != input.Control_Unit && ID != input.Overlay_Asset`); if they are not active on any other active bed station, revert their `Rental_State = "Inactive"`.
    - If `Deployment_Status == "Closed"`: Set `Rental_State = "Inactive"` for both `input.Control_Unit` and `input.Overlay_Asset`.
  - Step 2: Append generated ` #[Bed_Station_ID]` identifier to `Bed_Station_Title` in database record if not already present.
  - Step 3: Redirect cleanly to `All_Bed_Stations` report view (`openUrl("#Report:All_Bed_Stations", "same window");`).
