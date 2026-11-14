# MODULE 6: RENTAL AGREEMENTS

**Form Name**: `Rental_Agreements`  
**Form Link Name**: `Rental_Agreements`  
**Default Report**: `All_Rental_Agreements`  
**Purpose**: Commercial rental master contracts executed with hospitals, institutions, or homecare patients.

---

## 1. Design Principle: 3-Field Streamlined Intake
The user enters/selects **ONLY THREE FIELDS**:
1. `Customer`
2. `Rental Item`
3. `Agreed Rate`  
All identifiers, modes, titles, and dates are auto-generated or derived by the system.
`Billing Unit` is auto-derived in the background and hidden from the form.
Active deployed bed stations are tracked natively via the live `Bed_Stations` relation in reports and detail views.

---

## 2. Field Specifications

### A. User Interactive Fields (The ONLY fields user enters / edits):
1. **Customer** `[Lookup -> Customers, displaying Customer_Name]` (Mandatory)
   - Field Link Name: `Customer`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.
2. **Rental Item** `[Lookup -> Rental_Items, displaying Item_Name]` (Mandatory)
   - Field Link Name: `Rental_Item`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.
3. **Agreed Rate** `[Currency / Decimal]` (Mandatory commercial rate)
   - Field Link Name: `Agreed_Rate`
   - Form Behavior: Visible & Enabled on Add and Edit.
4. **Agreement Title** `[Single Line]` (Auto-filled on Customer + Item selection, fully editable by user)
   - Field Link Name: `Agreement_Title`
   - Form Behavior: Visible & Enabled on Add and Edit.
5. **Agreement Start Date** `[Date]` (Auto-filled with today's date, fully editable by user)
   - Field Link Name: `Agreement_Start_Date`
   - Form Behavior: Visible & Enabled on Add and Edit.

### B. Auto-Derived Background Field (Hidden from User UI):
6. **Billing Unit** `[Dropdown - Hidden Background Field]` (Auto-derived from Rental Item: `"Day"` | `"Month"`)
   - Field Link Name: `Billing_Unit`
   - Options: `"Day"`, `"Month"`
   - Form Behavior: Hidden on Add and Edit (auto-populated silently in background from `Rental_Items`). Preserved for downstream billing logic and report grouping.

### C. System & Lifecycle Fields:
7. **Rental Agreement ID** `[Auto-Number - Native Auto-Hidden on Add]` (Native primary key, Prefix: `RA-`, e.g. `RA-0001`)
   - Field Link Name: `Rental_Agreement_ID`
   - Form Behavior: Auto-Hidden on Add; Visible & Locked in Edit mode.
8. **Security Deposit** `[Currency / Decimal - Hidden on Add]` (Reserved for future deposit management)
   - Field Link Name: `Security_Deposit`
   - Form Behavior: Hidden on Add; Visible & Locked in Edit mode.
9. **Agreement End Date** `[Date - Hidden on Add]` (Blank on creation; populated upon contract termination / closure)
   - Field Link Name: `Agreement_End_Date`
   - Form Behavior: Hidden on Add; Visible & Enabled in Edit mode upon termination.
10. **Agreement Status** `[Dropdown - Hidden on Add]` (Default: `"Active"`, Options: `"Draft"`, `"Active"`, `"Suspended"`, `"Closed"`)
    - Field Link Name: `Agreement_Status`
    - Form Behavior: Hidden on Add (defaults to "Active" in background); Visible & Enabled in Edit mode.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable user entry fields (`Customer`, `Rental_Item`, `Agreed_Rate`, `Agreement_Title`, `Agreement_Start_Date`). Default `Agreement_Start_Date = zoho.currentdate`. Hide `Billing_Unit` (managed in background). Hide irrelevant intake fields (`Agreement_Status`, `Agreement_End_Date`, `Security_Deposit`). Initialize background default `Agreement_Status = "Active"`.
  - In EDIT Mode: Lock immutable fields (`Rental_Agreement_ID`, `Customer`, `Rental_Item`). Keep `Billing_Unit` hidden. Allow updating `Agreed_Rate`, `Agreement_Title`, `Agreement_Start_Date`, `Agreement_Status`, and `Agreement_End_Date`. Reveal lifecycle tracking fields.

### Workflow 2: On User Input (Step 1: Rental Item)
- **File**: `scripts/06_rental_agreements/2_1_Rental_Agreements_onuserinput_rentalitem.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Rental_Item -> On User Input
- **Plain English Logic**:
  - Step 1: Ensure `Agreement_Start_Date` is set to today if blank.
  - Step 2: Query linked `Rental_Items` record and set `Billing_Unit` ("Day" or "Month") silently in background.
  - Step 3: If `Customer` is also selected, dynamically auto-populate `Agreement_Title`: `[Customer Name] - [Item Name]`, keeping it user-editable.

### Workflow 3: On User Input (Step 2: Customer)
- **File**: `scripts/06_rental_agreements/2_2_Rental_Agreements_onuserinput_customer.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Customer -> On User Input
- **Plain English Logic**:
  - Step 1: If `Rental_Item` is also selected, dynamically auto-populate `Agreement_Title`: `[Customer Name] - [Item Name]`, keeping it user-editable.

### Workflow 4: On Form Validation & Safety Guard
- **File**: `scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory validation: Ensure `Customer`, `Rental_Item`, and `Agreed_Rate` are provided.
  - Step 2: Rate validation: Ensure `Agreed_Rate > 0`.
  - Step 3: Safety check: Ensure `Agreement_Title`, `Agreement_Start_Date`, and `Billing_Unit` are populated.
  - Step 4: Duplicate warning: Check if active Daily contract already exists for this exact Customer and Rental Item combination.
  - Step 5: Default `Agreement_Status = "Active"` if null.

### Workflow 5: On Form Submission Success
- **File**: `scripts/06_rental_agreements/4_Rental_Agreements_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Append generated `(RA-XXXX)` sequence to `Agreement_Title` if not already present.
  - Step 2: Redirect cleanly to `All_Rental_Agreements` report.
