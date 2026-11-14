# MODULE: PRODUCT MODEL

**Form Name**: `Product Model`  
**Form Link Name**: `Product_Model`  
**Default Report**: `All_Product_Models`  
**Purpose**: Central master registry of physical hardware device models (e.g., Altern4, Altern8, EturnalOne, Eturnal One with foam). Governs product model identification and provides an authoritative relational lookup for physical devices and rental items.

---

## 1. Field Specifications

### User Interactive Field:
1. **Product Model Name** `[Single Line]` (Mandatory, Unique)
   - Field Link Name: `Product_Model_Name`
   - Purpose: Display name used across all system lookups (e.g., "Altern4", "Altern8", "EturnalOne", "Eturnal One with foam").
   - Behavior: Enabled and visible on Add and Edit modes.

---

## 2. Downstream Usage & Cross-Module Architecture

- **In Module 5 (`Devices`)**:
  - Field: `Product Model` (`Product_Model`) [Lookup -> `Product_Model`, displaying `Product_Model_Name`, Disabled Context].
  - On entering/scanning a serial number (e.g., `ALT4CU053`, `ALT8CU001`, `ETOCU065`), Deluge matches the serial prefix to the corresponding `Product_Model` record and auto-populates `input.Product_Model`.
- **In Module 4 (`Rental_Items`)**:
  - Field: `Product Model` (`Product_Model`) [Lookup -> `Product_Model`, displaying `Product_Model_Name`, Read-Only].
  - Auto-linked when mapping rental service items by matching model keywords.
- **In Module 8 (`Bed_Stations`)**:
  - Deluge checks `cu_rec.Product_Model == ov_rec.Product_Model` to enforce hardware family compatibility when pairing Control Units and Overlays at bedside.

---

## 3. Workflows & Deluge Scripts

### Workflow 1: On Form Load
- **File**: `scripts/product_models/1_Product_Model_onload.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Load
- **Plain English Logic**:
  - In Add and Edit modes: Enable `Product_Model_Name` for user entry.

### Workflow 2: On Form Validation & Safety Guard
- **File**: `scripts/product_models/3_Product_Model_onvalidate.deluge`
- **Trigger**: Created or Edited -> Form Actions -> On Validate
- **Plain English Logic**:
  - Step 1: Mandatory check: Ensure `Product_Model_Name` is populated.
  - Step 2: Sanitize input: Trim whitespace from `Product_Model_Name`.
  - Step 3: Duplicate check: Query `Product_Model` to ensure `Product_Model_Name` does not already exist on another record.

### Workflow 3: On Form Submission Success
- **File**: `scripts/product_models/4_Product_Model_onsuccess.deluge`
- **Trigger**: Created -> Form Actions -> On Success
- **Plain English Logic**:
  - Step 1: Redirect user cleanly to `All_Product_Models` report.
