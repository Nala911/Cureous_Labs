# MODULE 4: RENTAL ITEMS

**Form Name**: `Rental Items`  
**Form Link Name**: `Rental_Items`  
**Default Report**: `All_Rental_Items`  
**Purpose**: Rental service items, commercial pricing rules, and Zoho Books service item mapping.

---

## 1. Design Principle: Zero-Typing Product Intake
The user selects **ONLY ONE FIELD**: `Item Name` from a dynamic Zoho Books service item dropdown!  
All item attributes (`Item_ID`, `Product_Model`, `Billing_Unit`) are derived automatically. Users only onboard new rental items; no editing of existing products.

---

## 2. Field Specifications

### A. User Interactive Field (The ONLY field user enters):
1. **Item Name** `[Dropdown]` (Mandatory, dynamically populated with active Zoho Books rental service items)
   - Field Link Name: `Item_Name`

### B. Auto-Derived Fields (System-Calculated from Zoho Books Item Profile, Read-Only):
2. **Item ID** `[Single Line - Read Only]` (Primary Key / Zoho Books Item ID, Mandatory, Unique)
   - Field Link Name: `Item_ID`
3. **Product Model** `[Single Line - Read Only]` (Auto-derived hardware model family)
   - Field Link Name: `Product_Model`
4. **Billing Unit** `[Dropdown - Read Only]` (Auto-derived from Item Name: `"Day"` | `"Month"`)
   - Field Link Name: `Billing_Unit`
   - Options: `"Day"`, `"Month"`

---

## 3. Downstream Linking Rules
- **In `Rental_Agreements` (Module 6)**:
  - User selects `Rental Item` [Lookup -> `Rental_Items`, displaying `Item_Name`].
  - On User Input of `Rental Item`:
    - `Rental_Agreement.Billing_Unit` auto-fills from `Rental_Items.Billing_Unit`.
    - `Rental_Agreement.Agreement_Title` auto-populates incorporating `Rental_Items.Item_Name`.
- **In `Bed_Stations` (Module 8)**:
  - Context field `Rental Item` (`Rental_Item` - Single Line, Disabled).
  - Auto-fills from `Rental_Agreement.Rental_Item -> Rental_Items.Item_Name` (or in Trial mode from `Devices.Device_Model`).
- **In `Rental_Logs` (Module 10) -> Zoho Books Invoice Push**:
  - Context field `Rental Item` [Lookup -> `Rental_Items`, displaying `Item_Name`].
  - Auto-fills from `Bed_Station -> Rental_Agreement.Rental_Item`.
  - Deluge directly pulls `Rental_Items.Item_ID` as the invoice line item ID.
  - Guaranteed zero item mapping discrepancy between Creator and Books!

---

## 4. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/04_rental_items/1_Rental_Items_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - Step 1: Lock auto-derived fields (`Item_ID`, `Product_Model`, `Billing_Unit`).
  - Step 2: In Edit mode, lock `Item_Name` to prevent altering existing items.
  - Step 3: Connect to Zoho Books via native API (`zoho.books.getRecords("items", ...)`), query active items starting with "Rental", filter out already onboarded items, sort alphabetically, and populate `Item_Name` choices.

### Workflow 2: On User Input (Step 1: Item Name)
- **File**: `scripts/04_rental_items/2_1_Rental_Items_onuserinput_itemname.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Item_Name -> On User Input
- **Plain English Logic**:
  - Step 1: If cleared, reset auto-derived fields.
  - Step 2: Duplicate check: If `Item_Name` exists in `Rental_Items`, alert and clear field.
  - Step 3: Match item in Books to retrieve `item_id` -> set `Item_ID`.
  - Step 4: Derive `Billing_Unit`: If name contains "Daily" -> `"Day"`; if name contains "Monthly" -> `"Month"`.
  - Step 5: Derive `Product_Model`: Match model keywords ("Altern4™", "Eturnal One™").

### Workflow 3: On Form Validation & Safety Guard
- **File**: `scripts/04_rental_items/3_Rental_Items_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory check: Ensure `Item_Name` and `Item_ID` are present.
  - Step 2: Duplicate check: Query `Rental_Items` for existing `Item_ID` or `Item_Name`.
  - Step 3: Safety fallback: Ensure `Product_Model` and `Billing_Unit` are populated.

### Workflow 4: On Form Submission Success
- **File**: `scripts/04_rental_items/4_Rental_Items_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Redirect cleanly to `All_Rental_Items` report.
