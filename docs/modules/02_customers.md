# MODULE 2: CUSTOMERS

**Form Name**: `Customers`  
**Form Link Name**: `Customers`  
**Default Report**: `All_Customers`  
**Purpose**: Stores institutional clients (hospitals, nursing homes) and homecare individuals. Links seamlessly to Zoho Books customer contacts.

---

## 1. Design Principle: Zero-Typing Customer Intake
The user enters **ONLY ONE FIELD**: `Customer Name` selected from a dynamic Zoho Books customer dropdown!  
No manual typing of IDs, customer types, or addresses. All customer asset summaries and device counts roll up dynamically via Zoho Creator Pages, Grouped Reports, and Detail View Related Lists (ensuring 100% live reporting without static counter columns).

---

## 2. Field Specifications

### A. User Interactive Field (The ONLY field user enters):
1. **Customer Name** `[Dropdown]` (Mandatory, dynamically populated with active Zoho Books customers)
   - Field Link Name: `Customer_Name`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.

### B. Auto-Derived Context & System Fields:
2. **Customer ID** `[Number - Hidden on Form]` (Primary Key / Zoho Books Contact ID, Mandatory, Unique)
   - Field Link Name: `Customer_ID`
   - Form Behavior: Hidden on Add and Edit (used purely for background Zoho Books API invoicing and uniqueness checks).
3. **Customer Type** `[Dropdown - Disabled Context]` (Auto-derived: `"Business"` or `"Individual"`)
   - Field Link Name: `Customer_Type`
   - Options: `"Business"`, `"Individual"`
   - Form Behavior: Visible & Disabled on Add and Edit.
4. **Billing Address** `[Multi-Line - Disabled Context]` (Auto-populated from Books billing address)
   - Field Link Name: `Billing_Address`
   - Form Behavior: Visible & Disabled on Add and Edit.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/02_customers/1_Customers_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - Step 1: Hide `Customer_ID` on form (internal Books Contact ID, zero UI clutter); disable context fields (`Customer_Type`, `Billing_Address`) so user sees fetched details without editing.
  - Step 2: In Add mode (`input.ID == null`), enable `Customer_Name` for user selection, and show context fields (`Customer_Type`, `Billing_Address`).
  - Step 3: In Add mode, query Zoho Books API for active contacts, filter out already onboarded customers, sort alphabetically, and populate `Customer_Name` dropdown.
  - Step 4: In Edit mode (`input.ID != null`), lock `Customer_Name`.

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

### Workflow 3: On Form Validation & Safety Guard
- **File**: `scripts/02_customers/3_Customers_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory check: Ensure `Customer_Name` is selected and `Customer_ID` was retrieved.
  - Step 2: Duplicate check: Query `Customers` for existing `Customer_ID` or `Customer_Name` on other records (`ID != input.ID`). Alert and cancel submit if duplicate.

### Workflow 4: On Form Submission Success
- **File**: `scripts/02_customers/4_Customers_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Redirect cleanly to `All_Customers` report.
