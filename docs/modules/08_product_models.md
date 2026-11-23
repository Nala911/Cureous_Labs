# MODULE 08: PRODUCT MODELS (HARDWARE CATALOG MASTER)

**Form Name**: `Product Models`  
**Form Link Name**: `Product_Models`  
**Default Report**: `All_Product_Models`  
**Purpose**: Central master catalog for medical device hardware product lines and models. Provides the relational source of truth for categorizing physical inventory in `Devices` and associating equipment with commercial rental item tiers.

---

## 1. Core Principles & Architecture
- **Master Model Catalog**: Defines standard hardware families manufactured and distributed by Cureous Labs (e.g. `Altern4`, `Altern8`, `Eturnal One`, `Eturnal One with foam`).
- **Relational Integrity**: Hardware assets in `Devices` reference this module via a single-select lookup link, eliminating typographical errors and standardizing fleet analytics.
- **Auto-Derivation Compatibility**: Serial numbers on intake auto-resolve to their respective `Product_Models` record ID while preserving technician flexibility to select model variants.

---

## 2. Field Specifications

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Product Model Name** | `Product_Model_Name` | Single Line | **User Entry** (Mandatory, Unique, Primary Identifier) | — | `scripts/05_devices/2_1_Devices_onuserinput_serialnumber.deluge`<br>`scripts/05_devices/3_Devices_onvalidate.deluge` | Target display field for `Devices.Product_Model` lookup |

---

## 3. Workflows & Deluge Scripts
- Standard native field properties enforce Mandatory and Unique constraints on `Product_Model_Name`.
- No custom form workflows currently required for basic catalog entry.
