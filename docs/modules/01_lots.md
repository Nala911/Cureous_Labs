# MODULE 1: LOTS

**Form Name**: `Lots`  
**Form Link Name**: `Lots`  
**Default Report**: `All_Lots`  
**Purpose**: Central manufacturing run ledger and batch traceability master. Governs the production batch lifecycle and links physical devices to their parent production run.

---

## 1. Design Principle: Ultra-Lean Form
The form contains **ONLY ONE FIELD**: `Lot ID`!  
Batch identification is kept purely focused on intake and traceability. Aggregated progress reporting and batch metrics are handled via report views and analytics rather than form fields.

### Multi-Product / Mixed Lot Architecture
A lot does not restrict product models. A single lot (e.g. `26HB04`) can contain Altern4™ CUs, Eturnal One™ CUs, and Overlays simultaneously.

---

## 2. Field Specifications

### User Interactive Field:
1. **Lot ID** `[Single Line]` (Mandatory, Unique, Primary Key, Barcode/QR capable)
   - Field Link Name: `Lot_ID`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/01_lots/1_Lots_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In Add mode (`input.ID == null`), enable and display `Lot_ID` as the single user entry field.
  - In Edit mode (`input.ID != null`), lock `Lot_ID` to preserve primary key batch identity.

### Workflow 2: On User Input (Lot ID)
- **File**: `scripts/01_lots/2_1_Lots_onuserinput_lotid.deluge`
- **Trigger**: Field Actions -> Lot_ID -> On User Input
- **Plain English Logic**:
  - No script required. Eliminating client-server roundtrips guarantees instantaneous typing and barcode scanning.

### Workflow 3: On Form Validation & Safety Guard
- **File**: `scripts/01_lots/3_Lots_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory check: Ensure `Lot_ID` is entered.
  - Step 2: Sanitize input: Trim whitespace and force uppercase.
  - Step 3: Minimum length guard: Enforce minimum of 3 characters.
  - Step 4: Duplicate check: Query `Lots` for matching `Lot_ID`. Alert and cancel submit if duplicate.

### Workflow 4: On Form Submission Success
- **File**: `scripts/01_lots/4_Lots_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Redirect cleanly to `All_Lots` report, dismissing the submit loading spinner.
