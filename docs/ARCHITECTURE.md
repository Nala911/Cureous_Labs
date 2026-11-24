# CUREOUS PATIENT REPOSITIONING SYSTEM (PRS)
## Master Architecture & System Overview

**Domain**: Medical Device Manufacturing & Commercial Rentals (Dynamic Alternating Pressure Air Mattresses & Control Units)  
**Platform**: Zoho Creator (Enterprise/Professional) + Zoho Books Integration  
**Version**: 3.1.0 (Unit-Level Direct Intake Architecture)  

---

## 1. System Overview & Physical Product Architecture

The company manufactures and rents Patient Repositioning Systems (PRS)—dynamic alternating pressure air mattresses for bedridden patients to prevent pressure injuries and bedsores.

A clinically functioning system consists of **TWO independent physical components**:
1. **Control Unit (CU)**: The electronic pump module containing dual compressor pumps, micro-solenoid valves, a micro-controller PCB, alternating pressure sensors, and audible/visual clinical alarm systems.
2. **Overlay (Mattress)**: The clinical patient surface containing polyurethane (PU) air cells arranged transversely, an antimicrobial wipe-down waterproof cover, and a quick-deflate CPR emergency valve.

### Manufacturing & Lifecycle Independence
- **Independent Manufacturing**: Control Units and Overlays are manufactured, assembled, tested, and inventoried completely independently.
- **Dynamic Bedside Pairing**: They are **never** manufactured in fixed pairs. They are paired dynamically at the patient bedside during field deployment via Module 07 (`Deployments`).
- **Independent Maintenance & Swaps**: If a CU pump develops a valve failure on site, only the CU is replaced; the mattress remains untouched under the patient. `Deployments` accommodates single-component swaps seamlessly.

---

## 2. Clientele & Commercial Architecture

### Clientele
- **Hospitals / Institutions**: Bulk fleet rentals (e.g., 5 to 20+ systems deployed across ICUs and wards).
- **Homecare Patients**: Single system rental for home-bound individuals.

### Commercial Deployment & Lifecycle Tracking
- **Deployments (Module 07)**: Manages dynamic bedside pairing, customer allocation, rental/demo/idle states, and automatically synchronizes physical device state in `Devices` (`Device_State` and `Current_Customer`).

---

## 3. Hardware Product Models & Families (Module 08: Product Models)

Managed centrally through the **`Product Models`** (`Product_Models`) master catalog, providing a normalized relational source of truth for physical devices (`Devices`):
- **Altern4**: Standard alternating pressure 4-inch system (basic institutional tier).
- **Altern8**: Advanced alternating pressure 8-inch system.
- **Eturnal One**: Advanced micro-climate alternating pressure system with dynamic cycle timing.
- **Eturnal One with foam**: Premium tier featuring a safety foam underlay base to prevent patient bottoming-out during sudden power cuts.
- Hardware models are auto-derived dynamically from Serial Number prefixes during device intake by resolving against `Product_Models`, while remaining accessible as lookup choices for technicians.

---

## 4. Zoho Books Accounting & Invoicing Integration

The architecture maintains clean boundaries between Zoho Creator (operational hardware & customer allocation) and Zoho Books (financial ledger & statutory compliance):

1. **Customers Onboarding**:
   - Creator pulls active customer contacts from Zoho Books via native API.
   - Creator stores `Customer_ID` (Books Contact ID), locks financial details, and prevents manual data entry errors.

---

## 5. User Roles & Permission Matrix

| Role | Customers | Devices | Books Sync |
| :--- | :--- | :--- | :--- |
| **Administrator** | Full | Full | Full |
| **Production / Assembly** | Read | Stages | No Access |
| **Warehouse Manager** | Read | Full | No Access |
| **Biomedical QC Engineer** | Read | Read | No Access (Full QC) |
| **Field Service Tech** | Read | Read | No Access |
| **Accounts Executive** | Create/Edit | Read | Sync to Books |
