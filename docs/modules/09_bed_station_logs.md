# MODULE 9: BED STATION LOGS

**Form Name**: `Bed Station Logs`  
**Form Link Name**: `Bed_Station_Logs`  
**Default Report**: `All_Bed_Station_Logs`  
**Purpose**: Dedicated, continuous timeline and snapshot ledger for every device pairing at every Bed Station. Records the exact device combination active from Period Start Date to Period End Date. Populated exclusively by system triggers (Bed Station Initial Deployment, Stateless Swap Wizard, and Bed Station Closure). Strictly read-only and locked against manual user tampering.

---

## 1. Field Specifications

| # | Field Label | Field Link Name | Field Type | Key / Constraint | Relational Target & Behavior |
| :- | :--- | :--- | :--- | :--- | :--- |
| 1 | **Bed Station Log ID** | `Bed_Station_Log_ID` | Single Line / Auto-Number | Primary Key | e.g. `BSL-2026-0001`. Unique audit ledger identifier. |
| 2 | **Bed Station** | `Bed_Station` | Lookup | Mandatory Foreign Key | Target: `Bed_Stations` (Displaying `Bed_Station_Title`). Clickable navigation link to live station. |
| 3 | **Customer** | `Customer` | Lookup | Read-Only Context | Target: `Customers` (Displaying `Customer_Name`). Clickable link to customer account. |
| 4 | **Rental Agreement** | `Rental_Agreement` | Lookup | Read-Only Context | Target: `Rental_Agreements` (Displaying `Agreement_Title`). Clickable link to contract. |
| 5 | **Control Unit** | `Control_Unit` | Lookup | Read-Only Context | Target: `Devices` (Filtered to `Device_Type == "Control Unit"`, Displaying `Serial_Number`). Clickable link to physical pump asset. |
| 6 | **Overlay** | `Overlay` | Lookup | Read-Only Context | Target: `Devices` (Filtered to `Device_Type == "Overlay"`, Displaying `Serial_Number`). Clickable link to physical mattress asset. |
| 7 | **Period Start Date** | `Period_Start_Date` | Date-Time | Mandatory | Timestamp when this exact hardware pairing became active on the bed station. |
| 8 | **Period End Date** | `Period_End_Date` | Date-Time | Null while Active | Timestamp when unit was swapped or bed station closed; `null` while pairing is active. |
| 9 | **Duration Days** | `Duration_Days` | Decimal / Formula | Read-Only Metric | Operating days during this specific pairing period. |

---

## 2. Fields Removed & Rationale

- **Bed Station Title** (old field 3): Removed. The `Bed_Station` lookup field natively displays `Bed_Station_Title`, providing full visual context and a 1-click drill-down link without redundant data storage.
- **Log Status** (old field 9): Removed. Redundant status tracking. An active pairing is naturally represented by `Period_End_Date == null`, while a closed pairing has a populated `Period_End_Date`.
- **Trigger Event, Component Swapped, Swap Reason, Technician Name, Technician Notes** (old fields 11–15): Removed. Operational and clinical justifications are handled in service tickets/wizards. Eliminating these fields keeps the interval ledger lightweight, high-performance, and strictly focused on hardware timeline tracking.

---

## 3. Integrity & Lock Policy

- **Immutable Read-Only Ledger**: Direct manual record creation, editing, or deletion is completely prohibited in standard user roles.
- **System-Driven Population**: Records are created and closed exclusively via Deluge automations:
  1. `scripts/08_bed_stations/4_Bed_Stations_onsuccess.deluge`: Automatically creates initial active log record when a new Bed Station is deployed, and closes the active log when status is set to `"Closed"`.
  2. `scripts/wizards/wizard_01_component_swap.deluge`: Automatically stamps `Period_End_Date` on the outgoing pairing and inserts a new active log record with the replacement asset.

---

## 4. Workflows & Scripts

### 1. Form On Load Safety Lock
- **Script File**: `scripts/09_bed_station_logs/1_Bed_Station_Logs_onload.deluge`
- **Trigger**: User opens the form directly via URL.
- **Plain English Logic**:
  - Checks if user opened the form directly in the UI.
  - Automatically redirects the user to the report `#Report:All_Bed_Station_Logs` to prevent unauthorized blank record entry.

### 2. Form On Validate Guard
- **Script File**: `scripts/09_bed_station_logs/3_Bed_Station_Logs_onvalidate.deluge`
- **Trigger**: User or external API attempts to submit a record.
- **Plain English Logic**:
  - Enforces system-only creation guard; cancels submission if submitted manually through the web form without required script parameters.
