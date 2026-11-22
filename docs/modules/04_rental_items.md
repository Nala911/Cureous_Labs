# MODULE 4: RENTAL ITEMS

**Form Name**: `Rental Items`  
**Form Link Name**: `Rental_Items`  
**Default Report**: `All_Rental_Items`  
**Purpose**: Catalog of rental service items synced with Zoho Books and mapping their Books Item ID for automated downstream invoicing in deployment logs.

---

## 1. Design Principle: Lean Integration Catalog
1. **Interactive Field**: The user selects `Item Name` from a dropdown (populated dynamically with active, un-onboarded rental items from Zoho Books).
2. **System-Managed Integration ID**:
   - `Rental_Item_ID`: Automatically resolved and populated from Zoho Books upon selecting `Item_Name`. Always disabled/hidden from standard users.
   - Provides downstream modules (such as Deployment Logs) with the exact Books `item_id` for invoice generation with zero API overhead.

---

## 2. Field Specifications

### Fields:
1. **Item Name** `[Dropdown]` (Mandatory, Unique)
   - Field Link Name: `Item_Name`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.
   - Common Options fetched from Zoho Books:
     - `Rental - Altern4™ - Daily`
     - `Rental - Altern4™ - Monthly`
     - `Rental - ETURNAL HOME- Monthly`
     - `Rental - ETURNAL Hospital™ - Daily`
     - `Rental - Eturnal One™ - Daily`
     - `Rental - Eturnal One™ - Monthly`
2. **Rental item id** `[Single Line]` (Zoho Books Item ID)
   - Field Link Name: `Rental_Item_ID`
   - Form Behavior: Disabled & Hidden on Add and Edit (System-managed).
   - Stores the exact Books item ID (e.g. `460000000098765`).

---

## 3. Downstream Linking Rules
- Downstream deployment logs and commercial billing functions query `Rental_Items` to retrieve `Rental_Item_ID`.
- When generating invoices via `zoho.books.createRecord("invoices", ...)`, `Rental_Item_ID` is passed directly as `item_id` in `line_items`.

---

## 4. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/04_rental_items/1_Rental_Items_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - Step 1: Keep `Rental_Item_ID` disabled and hidden.
  - Step 2: In Edit mode, lock `Item_Name` to prevent altering existing records.
  - Step 3: In Add mode, query active rental items from Zoho Books, filter out already onboarded items, sort alphabetically, and populate `Item_Name` choices (fallback to standard options if offline).

### Workflow 2: On User Input (Step 1: Item Name)
- **File**: `scripts/04_rental_items/2_1_Rental_Items_onuserinput_itemname.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Item_Name -> On User Input
- **Plain English Logic**:
  - Step 1: If selection is cleared, reset `Rental_Item_ID`.
  - Step 2: Duplicate check: If `Item_Name` already exists in `Rental_Items`, alert user.
  - Step 3: Query Zoho Books to retrieve the matching `item_id` for the selected `Item_Name` and populate `Rental_Item_ID`.

### Workflow 3: On Form Validation & Safety Guard
- **File**: `scripts/04_rental_items/3_Rental_Items_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory check: Ensure `Item_Name` is present.
  - Step 2: Duplicate check: Query `Rental_Items` to ensure `Item_Name` is unique.
  - Step 3: Safety fallback: Ensure `Rental_Item_ID` is populated from Books before saving.

### Workflow 4: On Form Submission Success
- **File**: `scripts/04_rental_items/4_Rental_Items_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Redirect cleanly to `All_Rental_Items` report.
