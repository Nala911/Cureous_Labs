# MODULE 2: CUSTOMERS

**Form Name**: `Customers`  
**Form Link Name**: `Customers`  
**Default Report**: `All_Customers`  
**Purpose**: Stores institutional clients (hospitals, nursing homes) and homecare individuals. Links seamlessly to Zoho Books customer contacts.

---

## 1. Design Principle: Zero-Typing Customer Intake
The user enters **ONLY ONE FIELD**: `Customer Name` selected from a dynamic Zoho Books customer dropdown!  
No manual typing of IDs, customer types, addresses, or operational statuses. All hardware counters roll up automatically via transactional modules.

---

## 2. Field Specifications

### A. User Interactive Field (The ONLY field user enters):
1. **Customer Name** `[Dropdown]` (Mandatory, dynamically populated with active Zoho Books customers)
   - Field Link Name: `Customer_Name`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.

### B. Auto-Derived Context Fields (Disabled - Provide Visual Context without Editing):
2. **Customer ID** `[Single Line - Disabled Context]` (Primary Key / Zoho Books Contact ID, Mandatory, Unique)
   - Field Link Name: `Customer_ID`
   - Form Behavior: Visible & Disabled on Add and Edit.
3. **Customer Type** `[Dropdown - Disabled Context]` (Auto-derived: `"Business"` or `"Individual"`)
   - Field Link Name: `Customer_Type`
   - Options: `"Business"`, `"Individual"`
   - Form Behavior: Visible & Disabled on Add and Edit.
4. **Billing Address** `[Multi-Line - Disabled Context]` (Auto-populated from Books billing address)
   - Field Link Name: `Billing_Address`
   - Form Behavior: Visible & Disabled on Add and Edit.

### C. Metric & Lifecycle Fields (Hidden on Add - For Reports & Status Tracking Only):
*Note: These fields make no sense when creating a new customer record. They are completely hidden on intake and utilized solely for reports and automated ledger rollups.*
5. **Customer Status** `[Dropdown - Hidden on Add]` (Default: `"Active"`. Options: `"Active"`, `"Inactive"`)
   - Field Link Name: `Customer_Status`
   - Form Behavior: Hidden on Add; Visible & Locked in Edit mode; shown in reports.
6. **Active Control Unit #** `[Number - Hidden on Add]` (Live count of CUs deployed/active with this customer)
   - Field Link Name: `Active_Control_Unit_Count`
   - Form Behavior: Hidden on Add; Visible & Locked in Edit mode; shown in reports.
7. **Active Overlay #** `[Number - Hidden on Add]` (Live count of Overlays deployed/active with this customer)
   - Field Link Name: `Active_Overlay_Count`
   - Form Behavior: Hidden on Add; Visible & Locked in Edit mode; shown in reports.
8. **Active Bed Stations #** `[Number - Hidden on Add]` (Live count of complete paired bed stations active on site)
   - Field Link Name: `Active_Bed_Stations_Count`
   - Form Behavior: Hidden on Add; Visible & Locked in Edit mode; shown in reports.
9. **Spare Units #** `[Number - Hidden on Add]` (Count of emergency spare CUs/Overlays held on site)
   - Field Link Name: `Spare_Units_Count`
   - Form Behavior: Hidden on Add; Visible & Locked in Edit mode; shown in reports.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/02_customers/1_Customers_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - Step 1: Disable context fields (`Customer_ID`, `Customer_Type`, `Billing_Address`) so user sees fetched details without editing.
  - Step 2: In Add mode (`input.ID == null`), enable `Customer_Name` for user selection, and hide all metric/status fields (`Customer_Status`, `Active_Control_Unit_Count`, `Active_Overlay_Count`, `Active_Bed_Stations_Count`, `Spare_Units_Count`).
  - Step 3: In Add mode, query Zoho Books API for active contacts, filter out already onboarded customers, sort alphabetically, and populate `Customer_Name` dropdown.
  - Step 4: In Edit mode (`input.ID != null`), lock `Customer_Name` and lock all status and metric counters.

### Workflow 2: On User Input (Step 1: Customer Name)
- **File**: `scripts/02_customers/2_1_Customers_onuserinput_customername.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Customer_Name -> On User Input
- **Plain English Logic**:
  - Step 1: If cleared, reset auto-derived fields and exit.
  - Step 2: Duplicate check: Check if `Customer_Name` already exists in `Customers`. If found, alert user and clear selection.
  - Step 3: Query Zoho Books contact profile using fast `search_text` and retrieve full profile via `getRecordsByID`.
  - Step 4: Auto-populate `Customer_ID` with the Books contact ID.
  - Step 5: Auto-populate `Customer_Type` (`"Business"` if `customer_sub_type == 'business'`, else `"Individual"`).
  - Step 6: Format and populate full `Billing_Address`, appending statutory GSTIN and contact details (Phone & Email) cleanly formatted under a divider.
  - Step 7: Auto-populate `Customer_Status = "Active"`.

### Workflow 3: On Form Validation & Safety Guard
- **File**: `scripts/02_customers/3_Customers_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory check: Ensure `Customer_Name` is selected and `Customer_ID` was retrieved.
  - Step 2: Duplicate check: Query `Customers` for existing `Customer_ID` or `Customer_Name` on other records (`ID != input.ID`). Alert and cancel submit if duplicate.
  - Step 3: Default `Customer_Status = "Active"` and rollup counters to 0 if null.

### Workflow 4: On Form Submission Success
- **File**: `scripts/02_customers/4_Customers_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Redirect cleanly to `All_Customers` report.
