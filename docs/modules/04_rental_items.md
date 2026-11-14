# MODULE 4: RENTAL ITEMS

**Form Name**: `Rental Items`  
**Form Link Name**: `Rental_Items`  
**Default Report**: `All_Rental_Items`  
**Purpose**: Rental service items, commercial pricing rules, and hardware catalog mapping.

---

## 1. Design Principle: Guided Intake with User Control
1. **Interactive Field**: The user selects `Item Name` from a dropdown (populated with standard options or active items).
2. **Auto-Derived & Editable**:
   - `Billing Unit`: Auto-derived from the name ("Day" or "Month") and left editable by the user.
   - `Product Model`: Auto-derived hardware model lookup (`Altern4` or `Eturnal`) and left editable by the user.

---

## 2. Field Specifications

### User Interactive Fields:
1. **Item Name** `[Dropdown]` (Mandatory, Unique)
   - Field Link Name: `Item_Name`
   - Common Options:
     - `Rental - Altern4™ - Daily`
     - `Rental - Altern4™ - Monthly`
     - `Rental - ETURNAL HOME- Monthly`
     - `Rental - ETURNAL Hospital™ - Daily`
     - `Rental - Eturnal One™ - Daily`
     - `Rental - Eturnal One™ - Monthly`
2. **Product Model** `[Lookup -> Product_Model, displaying Product_Model_Name]` (Auto-derived: `Altern4` or `Eturnal`, User-Editable)
   - Field Link Name: `Product_Model`
3. **Billing Unit** `[Dropdown]` (Auto-derived from Item Name: `"Day"` | `"Month"`, User-Editable)
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
  - Auto-fills from `Rental_Agreement.Rental_Item -> Rental_Items.Item_Name`.
- **In `Rental_Logs` (Module 10)**:
  - Context field `Rental Item` [Lookup -> `Rental_Items`, displaying `Item_Name`].
  - Auto-fills from `Bed_Station -> Rental_Agreement.Rental_Item`.

---

## 4. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/04_rental_items/1_Rental_Items_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - Step 1: Ensure `Billing_Unit` and `Product_Model` are visible and enabled (editable by user).
  - Step 2: In Edit mode, lock `Item_Name` to prevent altering existing items.
  - Step 3: In Add mode, query active rental items (fallback to standard 6 options if offline/empty), filter out already onboarded items, sort alphabetically, and populate `Item_Name` choices.

### Workflow 2: On User Input (Step 1: Item Name)
- **File**: `scripts/04_rental_items/2_1_Rental_Items_onuserinput_itemname.deluge`
- **Trigger**: Created or Edited -> Field Actions -> Item_Name -> On User Input
- **Plain English Logic**:
  - Step 1: If selection is cleared, reset dependent fields.
  - Step 2: Parse "Daily" / "Day" -> `"Day"`, "Monthly" / "Month" -> `"Month"` into `Billing_Unit`.
  - Step 3: Parse "Altern4" / "Altern" -> `"Altern4"`, "Eturnal" -> `"Eturnal"`, query `Product_Model` master, and assign lookup ID to `Product_Model`.
  - Step 4: Ensure `Billing_Unit` and `Product_Model` remain editable.
  - Step 5: Duplicate check: If `Item_Name` already exists in `Rental_Items`, alert user.

### Workflow 3: On Form Validation & Safety Guard
- **File**: `scripts/04_rental_items/3_Rental_Items_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory check: Ensure `Item_Name` is present.
  - Step 2: Duplicate check: Query `Rental_Items` to ensure `Item_Name` is unique.
  - Step 3: Safety fallback: Ensure `Product_Model` and `Billing_Unit` are populated before saving.

### Workflow 4: On Form Submission Success
- **File**: `scripts/04_rental_items/4_Rental_Items_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Redirect cleanly to `All_Rental_Items` report.
