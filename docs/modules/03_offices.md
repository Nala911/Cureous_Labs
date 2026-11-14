# MODULE 3: OFFICES

**Form Name**: `Offices`  
**Form Link Name**: `Offices`  
**Default Report**: `All_Offices`  
**Purpose**: Master registry for company GST-registered physical branch offices and operational hubs (Bangalore HQ, Kerala Office / Palakkad, and Nagpur Office). Provides clean office selection for device manufacturing custody and branch transfers.

---

## 1. Field Specifications

### User Interactive Fields:
1. **Office Name** `[Single Line]` (Mandatory, Unique)
   - Field Link Name: `Office_Name`
   - Purpose: Primary display label used across all system dropdowns and lookups (e.g., "Bangalore HQ", "Kerala Office / Palakkad", "Nagpur Office").
   - Behavior: Enabled and editable on Add and Edit modes.
2. **Is Primary Hub** `[Decision (Checkbox)]` (Optional, Default: false)
   - Field Link Name: `Is_Primary`
   - Purpose: Checked strictly for central manufacturing headquarters (Bangalore HQ). Used to auto-assign intake warehouse for newly manufactured devices in Module 5.

---

## 2. Downstream Usage & Cross-Module Architecture
- **In Module 5 (`Devices`)**: Field `Current_Office` [Lookup -> `Offices`, displaying `Office_Name`].
- **In Module 7 (`Stock_Transfers`)**: Fields `Source_Office` and `Destination_Office` [Lookup -> `Offices`, displaying `Office_Name`].

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/03_offices/1_Offices_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In Add mode: Enable `Office_Name` and set `Is_Primary = false`.
  - In Edit mode: Keep `Office_Name` and `Is_Primary` editable.

### Workflow 2: On Form Validation & Safety Guard
- **File**: `scripts/03_offices/3_Offices_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory and whitespace trim check on `Office_Name`.
  - Step 2: Duplicate check: Query `Offices` to ensure `Office_Name` does not already exist on another record.

### Workflow 3: On Form Submission Success
- **File**: `scripts/03_offices/4_Offices_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Single Primary Hub enforcement: If `input.Is_Primary == true`, set any other office with `Is_Primary == true` to `false`.
  - Step 2: Redirect cleanly to `All_Offices` report.
