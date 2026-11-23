# MODULE 06: LOTS (MANUFACTURING BATCH MASTER)

**Form Name**: `Lots`  
**Form Link Name**: `Lots`  
**Default Report**: `All_Lots`  
**Purpose**: Central master registry for manufacturing batch/lot tracking. Provides the relational source of truth for assigning physical hardware components (Control Units and Overlays) to production batches.

---

## 1. Core Principles & Traceability
- **Master Batch Identity**: Every manufacturing production run is tagged with a distinct Lot ID.
- **Relational Integrity**: Hardware units in `Devices` reference this module via a single-select lookup link, eliminating typographical discrepancies and facilitating lot-level quality tracking or recall isolation.
- **Lot ID Format Standard**: Standard alphanumeric batch code, typically decoded as `[YY][M][Seq]` (e.g. `26AB02`, `26FB03`).

---

## 2. Field Specifications

| Field Label | Field Link Name | Type | Form UI Behavior / Constraints | Target Lookup | Referenced in Scripts | Cross-Module Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Lot ID** | `Lot_ID` | Single Line | **User Entry** (Mandatory, Unique, Primary Identifier) | — | — | Target display field for `Devices.Lot` lookup |

---

## 3. Workflows & Deluge Scripts
- Standard native field properties enforce Mandatory and Unique constraints on `Lot_ID`.
- No custom form workflows currently required for basic batch entry.
