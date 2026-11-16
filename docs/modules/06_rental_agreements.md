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
All identifiers, titles, and dates are auto-generated or derived by the system.
Billing cadence ("Day" or "Month") is inherited directly from the linked `Rental_Item` (eliminating redundant fields).
Active deployed bed stations are tracked natively via the live `Bed_Stations` relation in reports and detail views.

---

## 2. Field Specifications

### A. User Interactive Fields (The ONLY fields user enters / edits):
1. **Customer** `[Lookup -> Customers, displaying Customer_Name]` (Mandatory)
   - Field Link Name: `Customer`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.
2. **Rental Item** `[Lookup -> Rental_Items, displaying Item_Name]` (Mandatory)
   - Field Link Name: `Rental_Item`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode. Inherits billing cadence (`Billing_Unit`) downstream.
3. **Agreed Rate** `[Currency / Decimal]` (Mandatory commercial rate)
   - Field Link Name: `Agreed_Rate`
   - Form Behavior: Visible & Enabled on Add and Edit.
4. **Agreement Title** `[Single Line]` (Business Unique PK; Auto-filled on Customer + Item selection as `[First 3 words of Customer] - [Clean Item] #[Count+1]`, fully editable by user, uniqueness validated on submit)
   - Field Link Name: `Agreement_Title`
   - Form Behavior: Visible & Enabled on Add and Edit.
5. **Agreement Start Date** `[Date]` (Auto-filled with today's date, fully editable by user)
   - Field Link Name: `Agreement_Start_Date`
   - Form Behavior: Visible & Enabled on Add and Edit.

### B. System & Lifecycle Fields:
6. **Security Deposit** `[Currency / Decimal - Hidden on Add]` (Reserved for future deposit management)
   - Field Link Name: `Security_Deposit`
   - Form Behavior: Hidden on Add; Visible & Locked in Edit mode.
7. **Agreement End Date** `[Date - Hidden on Add]` (Blank on creation; populated upon contract termination / closure)
   - Field Link Name: `Agreement_End_Date`
   - Form Behavior: Hidden on Add; Visible & Enabled in Edit mode upon termination.
8. **Agreement Status** `[Dropdown - Hidden on Add]` (Default: `"Active"`, Options: `"Draft"`, `"Active"`, `"Suspended"`, `"Closed"`)
   - Field Link Name: `Agreement_Status`
   - Form Behavior: Hidden on Add (defaults to "Active" in background); Visible & Enabled in Edit mode.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/06_rental_agreements/1_Rental_Agreements_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In ADD Mode: Enable user entry fields (`Customer`, `Rental_Item`, `Agreed_Rate`, `Agreement_Title`, `Agreement_Start_Date`). Default `Agreement_Start_Date = zoho.currentdate`. Hide irrelevant intake fields (`Agreement_Status`, `Agreement_End_Date`, `Security_Deposit`). Initialize background default `Agreement_Status = "Active"`.
  - In EDIT Mode: Lock immutable fields (`Customer`, `Rental_Item`). Allow updating `Agreed_Rate`, `Agreement_Title`, `Agreement_Start_Date`, `Agreement_Status`, and `Agreement_End_Date`. Reveal lifecycle tracking fields.

### Workflow 2: On User Input (Step 1: Rental Item)
- **File**: `scripts/06_rental_agreements/2_1_Rental_Agreements_onuserinput_rentalitem.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Rental_Item -> On User Input
- **Plain English Logic**:
  - Step 1: Ensure `Agreement_Start_Date` is set to today if blank.
  - Step 2: Fetch clean item name from `Rental_Items`.
  - Step 3: If `Customer` is also selected, dynamically auto-populate `Agreement_Title`: extracts first 3 words of Customer, clean item model/cadence (last 3 words/clean model, e.g. `Eturnal One - Monthly`), and appends `#[Count+1]` based on existing agreements for that customer. Leaves title fully user-editable.

### Workflow 3: On User Input (Step 2: Customer)
- **File**: `scripts/06_rental_agreements/2_2_Rental_Agreements_onuserinput_customer.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Customer -> On User Input
- **Plain English Logic**:
  - Step 1: If `Rental_Item` is also selected, dynamically auto-populate `Agreement_Title` with the first 3 words of Customer, clean item model, and `#[Count+1]`, keeping it fully user-editable.

### Workflow 4: On Form Validation & Safety Guard
- **File**: `scripts/06_rental_agreements/3_Rental_Agreements_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory validation: Ensure `Customer`, `Rental_Item`, and `Agreed_Rate` are provided.
  - Step 2: Rate validation: Ensure `Agreed_Rate > 0`.
  - Step 3: Title Uniqueness Check: Query `Rental_Agreements` to guarantee no duplicate `Agreement_Title` exists (excluding current record ID in Edit mode). Cancels submission with alert if duplicate found.
  - Step 4: Safety check: Ensure `Agreement_Title` and `Agreement_Start_Date` are populated (applying fallback generation if title is blank).
  - Step 5: Duplicate warning: Check if active Daily contract already exists for this exact Customer and Rental Item combination (by checking `Rental_Item.Billing_Unit == "Day"`).
  - Step 6: Default `Agreement_Status = "Active"` if null.

### Workflow 5: On Form Submission Success
- **File**: `scripts/06_rental_agreements/4_Rental_Agreements_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Redirect cleanly to `All_Rental_Agreements` report.
