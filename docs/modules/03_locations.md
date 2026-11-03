# MODULE 3: LOCATIONS

**Form Name**: `Locations`  
**Form Link Name**: `Locations`  
**Default Report**: `All_Locations`  
**Purpose**: Master registry for company GST-registered physical branch offices and operational hubs (Bangalore HQ, Kerala Office / Palakkad, and Nagpur Office). Provides clean location selection for device manufacturing custody and Delivery Challan dispatch branch mapping for Zoho Books & E-Way Bill generation (Rule 55 CGST). Zero virtual warehouses.

---

## 1. Field Specifications

### User Interactive Fields:
1. **Location's Name** `[Single Line]` (Mandatory, Unique)
   - Field Link Name: `Location_s_Name`
   - Purpose: Primary display label used across all system dropdowns and lookups (e.g., "Bangalore HQ", "Palakkad Branch", "Nagpur Branch").
2. **Location ID** `[Single Line]` (Mandatory, Unique)
   - Field Link Name: `Location_ID`
   - Purpose: Exact Zoho Books Location/Warehouse ID (e.g. `2938472000000123456`). Locked in Edit mode to ensure audit integrity.
3. **Is Primary Hub** `[Decision (Checkbox)]` (Optional, Default: false)
   - Field Link Name: `Is_Primary`
   - Purpose: Checked strictly for central manufacturing headquarters (Bangalore HQ). Used to auto-assign intake warehouse for newly manufactured devices in Module 5.

---

## 2. Downstream Usage & Cross-Module Architecture
- **In Module 5 (`Devices`)**: Field `Current_Location` [Lookup -> `Locations`, displaying `Location_s_Name`].
- **In Module 7 (`Stock_Transfers`)**: Field `Source_Office` and `Destination_Office` [Lookup -> `Locations`, displaying `Location_s_Name`].
- **Zoho Books API Push**: `challan_payload.put("location_id", input.Source_Office.Location_ID)`. Zoho Books automatically issues the Delivery Challan from that branch's registered GSTIN and address.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/03_locations/1_Locations_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - Step 1: Zero external API calls: form loads instantly in < 5ms.
  - Step 2: In Add mode: Enable all fields and set `Is_Primary = false`.
  - Step 3: In Edit mode: Lock `Location_ID` as read-only. Keep `Location_s_Name` and `Is_Primary` editable.

### Workflow 2: On Form Validation & Safety Guard
- **File**: `scripts/03_locations/3_Locations_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory and whitespace trim check on `Location_s_Name` and `Location_ID`.
  - Step 2: Duplicate check: Query `Locations` to ensure `Location_ID` or `Location_s_Name` does not already exist. Cancel submit if duplicate.

### Workflow 3: On Form Submission Success
- **File**: `scripts/03_locations/4_Locations_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Single Primary Hub enforcement: If `input.Is_Primary == true`, set any other location with `Is_Primary == true` to `false`.
  - Step 2: Redirect cleanly to `All_Locations` report.
