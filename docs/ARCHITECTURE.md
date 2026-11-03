# CUREOUS PATIENT REPOSITIONING SYSTEM (PRS)
## Master Architecture & System Overview

**Domain**: Medical Device Manufacturing & Commercial Rentals (Dynamic Alternating Pressure Air Mattresses & Control Units)  
**Platform**: Zoho Creator (Enterprise/Professional) + Zoho Books Integration  
**Version**: 2.7.0 (Modular Documentation & Live Data Matrix Architecture)  

---

## 1. System Overview & Physical Product Architecture

The company manufactures and rents Patient Repositioning Systems (PRS)—dynamic alternating pressure air mattresses for bedridden patients to prevent pressure injuries and bedsores.

A clinically functioning system consists of **TWO independent physical components**:
1. **Control Unit (CU)**: The electronic pump module containing dual compressor pumps, micro-solenoid valves, a micro-controller PCB, alternating pressure sensors, and audible/visual clinical alarm systems.
2. **Overlay (Mattress)**: The clinical patient surface containing polyurethane (PU) air cells arranged transversely, an antimicrobial wipe-down waterproof cover, and a quick-deflate CPR emergency valve.

### Manufacturing & Lifecycle Independence
- **Independent Manufacturing**: Control Units and Overlays are manufactured, assembled, tested, and inventoried completely independently.
- **Dynamic Bedside Pairing**: They are **never** manufactured in fixed pairs. They are paired dynamically at the patient bedside in Module 8 (`Bed_Stations`).
- **Independent Maintenance & Swaps**: If a CU pump develops a valve failure on site, only the CU is replaced via the Stateless Swap Wizard; the mattress remains untouched under the patient.

---

## 2. Clientele & Commercial Rental Types

### Clientele
- **Hospitals / Institutions**: Bulk fleet rentals (e.g., 5 to 20+ systems deployed across ICUs and wards).
- **Homecare Patients**: Single system rental for home-bound individuals.

### Commercial Rental Modes
1. **Pay-Per-Use (Daily)**:
   - Billed dynamically based on active patient usage days within the calendar month (`Usage_End_Date - Usage_Start_Date + 1`).
   - Multiple bed stations can be deployed flexibly under a single agreement.
2. **Fixed Monthly Rental**:
   - Flat agreed monthly fee regardless of how many days the bed was occupied.
   - Strict 1-to-1 relationship between Agreement and Bed Station (the agreement start date anchors recurring billing cycles).
3. **Clinical Trial / Demo**:
   - Temporary zero-cost evaluation period governed with full serial number tracking and delivery challans.

---

## 3. Product Tiers

Configured dynamically by administrators in `Product_Catalog`:
- **Altern4™**: Standard alternating pressure 4-inch system (basic institutional tier).
- **Eturnal One™**: Advanced micro-climate alternating pressure system with dynamic cycle timing.
- **Eturnal One with Foam**: Premium tier featuring a safety foam underlay base to prevent patient bottoming-out during sudden power cuts.
- Future models can be onboarded dynamically via Zoho Books item sync.

---

## 4. Manufacturing Lot Architecture

- Devices are manufactured in production runs designated by **Lot Numbers** (e.g. `26HB04`).
- **Mixed Product / Component Lots**: A single lot can contain any combination of Control Units and Overlays across multiple product models (Altern4™, Eturnal One™).
- **Independent Progress**: Units inside the same lot advance through assembly and testing stages at different speeds, but their progress rolls up automatically into the parent `Lots` record.
- **Gated Quality Certification**: No unit can be deployed to a customer or transferred to a branch until it passes Quality Control in Module 11 and receives a "Passed & Certified" clinical clearance.

---

## 5. Zoho Books Accounting & Invoicing Integration

The architecture maintains clean boundaries between Zoho Creator (operational hardware & bedside custody) and Zoho Books (financial ledger & statutory compliance):

1. **Customers Onboarding**:
   - Creator pulls active customer contacts from Zoho Books via native API.
   - Creator stores `Customer_ID` (Books Contact ID), locks financial details, and prevents manual data entry errors.
2. **Branch Offices & Locations (Rule 55 CGST Delivery Challans)**:
   - Creator `Locations` directly maps to the company's 3 GST-registered physical branch offices (Bangalore HQ, Kerala Office / Palakkad, and Nagpur Office).
   - Zero virtual warehouses: Delivery Challans and E-Way bills generated through Books API use the branch's registered `location_id`, with statutory GSTINs and branch addresses handled natively in Zoho Books.
3. **Product Catalog Sync**:
   - Creator rental items map 1:1 with Zoho Books rental service items, using `Books_Item_ID` as the billing line-item identifier.
4. **Automated Books Invoice Push**:
   - Accounts can push single or consolidated draft invoices directly from `Rental_Log` into Zoho Books.
   - Invoices include detailed line descriptions with Bed Station labels, serial numbers, and exact calendar dates.

---

## 6. User Roles & Permission Matrix

| Role | Lots | Customers | Devices | Stock Transfers | Bed Stations | Rental Logs | Books Invoicing |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Administrator** | Full | Full | Full | Full | Full | Full | Full |
| **Production / Assembly** | Create/Edit | Read | Stages | Read | No Access | No Access | No Access |
| **Warehouse Manager** | Read | Read | Full | Create/Edit | Read | Read | No Access |
| **Biomedical QC Engineer** | Read | Read | Read | Read | Read | No Access | No Access (Full QC) |
| **Field Service Tech** | Read | Read | Read | Create | Create/Swap | Create | No Access |
| **Accounts Executive** | Read | Create/Edit | Read | Read | Read | Full | Push to Books |
