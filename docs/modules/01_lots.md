# MODULE 1: LOTS

**Form Name**: `Lots`  
**Form Link Name**: `Lots`  
**Default Report**: `All_Lots`  
**Purpose**: Central manufacturing run ledger and batch traceability master. Governs the production batch lifecycle and provides a live, aggregated stage-by-stage ledger of all Control Units and Overlays manufactured under each lot.

---

## 1. Design Principle: Ultra-Lean UX
The user enters **ONLY ONE FIELD**: `Lot ID`!  
All inventory counters and stage breakdowns roll up automatically via Asset Master (`Devices`) and Quality Control automated workflows. No unnecessary date pickers, target numbers, supervisors, or manual notes.

### Multi-Product / Mixed Lot Architecture
A lot does not restrict product models. A single lot (e.g. `26HB04`) can contain Altern4™ CUs, Eturnal One™ CUs, and Overlays simultaneously.

---

## 2. Field Specifications

### A. User Interactive Field (The ONLY field user enters):
1. **Lot ID** `[Single Line]` (Mandatory, Unique, Primary Key, Barcode/QR capable)
   - Field Link Name: `Lot_ID`
   - Form Behavior: Visible & Enabled on Add; Locked in Edit mode.

### B. Rollup Metrics (Strictly Read-Only, Updated via Asset Master & QC Workflows, Hidden on Create):
*Note: All metrical rollup fields are completely hidden during record creation to keep the intake form ultra-lean and clutter-free. They are visible in reports and locked in edit mode.*
2. **Total CUs Count** `[Number - Read Only - Hidden on Add]` (Total Control Units registered under this lot)
   - Field Link Name: `Total_CUs_Count`
3. **Total Overlays Count** `[Number - Read Only - Hidden on Add]` (Total Overlays registered under this lot)
   - Field Link Name: `Total_Overlays_Count`
4. **Total Units Count** `[Number - Read Only - Hidden on Add]` (Sum: Total CUs + Total Overlays)
   - Field Link Name: `Total_Units_Count`
5. **CUs In Progress** `[Number - Read Only - Hidden on Add]` (CUs currently in manufacturing/assembly/testing)
   - Field Link Name: `CUs_In_Progress`
6. **CUs Rent Ready** `[Number - Read Only - Hidden on Add]` (CUs that reached "Manufacturing Complete")
   - Field Link Name: `CUs_Rent_Ready`
7. **Overlays In Progress** `[Number - Read Only - Hidden on Add]` (Overlays currently in cutting/welding/testing)
   - Field Link Name: `Overlays_In_Progress`
8. **Overlays Rent Ready** `[Number - Read Only - Hidden on Add]` (Overlays that reached "Manufacturing Complete")
   - Field Link Name: `Overlays_Rent_Ready`

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/01_lots/1_Lots_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - Step 1: In Add mode (`input.ID == null`), enable `Lot_ID` as the single user entry field, and hide all rollup metric fields (`Total_CUs_Count`, `Total_Overlays_Count`, `Total_Units_Count`, `CUs_In_Progress`, `CUs_Rent_Ready`, `Overlays_In_Progress`, `Overlays_Rent_Ready`).
  - Step 2: In Edit mode (`input.ID != null`), lock `Lot_ID` to protect primary key batch integrity, and display all metric rollup counters locked as read-only.

### Workflow 2: On User Input (Lot ID)
- *No script required.* Eliminating client-server roundtrips guarantees instantaneous typing and barcode scanning.

### Workflow 3: On Form Validation & Safety Guard
- **File**: `scripts/01_lots/3_Lots_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory check: Ensure `Lot_ID` is not empty.
  - Step 2: Sanitize input: Trim whitespace and force uppercase.
  - Step 3: Minimum length guard: Minimum 3 characters.
  - Step 4: Duplicate check: Query `Lots` for matching `Lot_ID`. Alert and cancel submit if duplicate.
  - Step 5: Default numeric counters to 0 if null.

### Workflow 4: On Form Submission Success
- **File**: `scripts/01_lots/4_Lots_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Redirect cleanly to `All_Lots` report, dismissing the submit loading spinner.
