# MODULE 3: OFFICES

**Form Name**: `Offices`  
**Form Link Name**: `Offices`  
**Default Report**: `All_Offices`  
**Purpose**: Master registry for company branch offices, warehouses, and physical locations. Used to track where devices are situated or transferred across branches.

---

## 1. Field Specifications

### User Interactive Field:
1. **Office Name** `[Single Line]` (Mandatory, Unique)
   - Field Link Name: `Office_Name`
   - Form Behavior: Visible & Enabled on Add and Edit.
   - Purpose: Display name for physical branch/office (e.g., "Bangalore HQ", "Kerala Office / Palakkad", "Nagpur Office").

---

## 2. Downstream Usage & Cross-Module Architecture
- **In Module 5 (`Devices`)**: Field `Current_Office` [Lookup -> `Offices`, displaying `Office_Name`].
- **In Transfers / Deployments**: Tracks source, destination, and present location custody.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/03_offices/1_Offices_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - Ensures `Office_Name` is visible and enabled on both Add and Edit modes.

### Workflow 2: On Form Validation & Safety Guard
- **File**: `scripts/03_offices/3_Offices_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory and whitespace trim check on `Office_Name`.
  - Step 2: Duplicate check: Query `Offices` to ensure `Office_Name` is unique.

### Workflow 3: On Form Submission Success
- **File**: `scripts/03_offices/4_Offices_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Redirect cleanly to `All_Offices` report.
