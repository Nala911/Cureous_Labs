# WIZARD 2: STATELESS BATCH MONTHLY USAGE GENERATOR

**Form Name**: `Batch_Usage_Generator_Wizard`  
**Form Type**: Stateless Form  
**Trigger**: Button `[ ⚡ Batch Generate Monthly Usage ]` on Customer or Agreement view.  
**Purpose**: Solves the administrative burden of manually creating individual usage logs for a hospital facility operating 5 to 20+ beds on Pay-Per-Use at month-end.

---

## 1. Fields on the Stateless Form

1. **Customer** `[Lookup -> Customers]`
2. **Rental Agreement** `[Lookup -> Rental_Agreements]` (Filtered to Pay-Per-Use contracts for this customer)
3. **Billing Month / Year** `[Dropdown]` (e.g. October 2026)
4. **Default Usage Start Date** `[Date]` (e.g. 2026-10-01)
5. **Default Usage End Date** `[Date]` (e.g. 2026-10-31)
6. **Active Bed Stations Dynamic Grid**:
   - `Bed Station Name` `[Single Line - Read Only]`
   - `Control Unit Serial` `[Single Line - Read Only]`
   - `Overlay Serial` `[Single Line - Read Only]`
   - `Active Days` `[Number]` (Defaults to calendar days in month, editable if a bed had patient downtime)
   - `Include in Billing` `[Checkbox]` (Defaults to Checked)
7. **Submit Button**: `"Generate All Usage Logs"`

---

## 2. Execution Outcome
Iterates across all checked bed stations in the subform, creates individual, itemized `Rental_Log` entries with exact device serial snapshots, calculated days, taxable amounts, and 5% GST in 3 seconds, ready for Accounts verification and Books batch invoicing.
