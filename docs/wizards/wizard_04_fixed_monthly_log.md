# WIZARD 4: STATELESS FIXED MONTHLY RENTAL LOG GENERATOR

**Form Name**: `Generate_Fixed_Monthly_Rental_Log_Wizard`  
**Form Type**: Stateless Modal  
**Trigger**: Custom Action Button `[ ⚡ Generate Monthly Rental Log ]` on `Rental_Agreements` record row.  
**Purpose**: Generates fixed monthly rental charges directly from the contract record on-demand (either in advance or at month-end). Auto-attaches the assigned Bed Station and active CU/Overlay serials, suggests the next unbilled cycle, computes flat monthly fee + 5% GST, and creates the `Rental_Log` record ready for invoicing.

---

## 1. Eligibility Guard
1. Agreement `Billing_Unit` must be `"Month"`.
2. Agreement must have an active deployed Bed Station (`Deployment_Status == "Active on Patient Bed"`). If no station is deployed, execution is blocked with an alert.

---

## 2. Fields on the Stateless Modal

1. **Agreement ID** `[Hidden / Single Line]`
2. **Agreement Title** `[Single Line - Read Only]`
3. **Customer** `[Single Line - Read Only]`
4. **Deployed Bed Station** `[Single Line - Read Only]` (Displays assigned `Bed_Station_Title`)
5. **Rental Item** `[Single Line - Read Only]`
6. **Active CU Serial** `[Single Line - Read Only]`
7. **Active Mattress Serial** `[Single Line - Read Only]`
8. **Agreed Monthly Rate (₹)** `[Currency - Read Only]`
9. **Billing Period Start Date** `[Date]` (Mandatory; editable)
10. **Billing Period End Date** `[Date]` (Mandatory; auto-calculated: Start Date + 1 Month - 1 Day)
11. **Discount / Adjustment (₹)** `[Currency]` (Default: 0.00)
12. **Taxable Value (₹)** `[Currency - Read Only]` (Agreed Rate - Discount)
13. **GST 5% (₹)** `[Currency - Read Only]` (Taxable Value * 0.05)
14. **Total Invoice Value (₹)** `[Currency - Read Only]` (Taxable Value + GST)
15. **Billing Notes** `[Multi-Line]` (e.g. "Advance monthly rental for November 2026")
16. **Submit Button**: `"Confirm & Generate Rental Log"`

---

## 3. Deluge Execution Logic

```deluge
void fn_generate_fixed_monthly_rental_log(int agreementId, date startDate, date endDate, decimal discount, string notes)
{
    agrRec = Rental_Agreements[ID == agreementId];
    
    // Safety check: Bed Station must exist
    activeStation = Bed_Station[
        Rental_Agreement == agreementId && 
        Deployment_Status in {"Active on Patient Bed", "Standby at Customer Site"}
    ];
    
    if(activeStation.count() == 0)
    {
        alert "Cannot generate Rental Log: No active Bed Station found for this agreement!";
        return;
    }
    
    stationRec = activeStation.get(0);
    rate = agrRec.Agreed_Rate;
    taxable = rate - ifNull(discount, 0.0);
    if(taxable < 0.0)
    {
        taxable = 0.0;
    }
    gstAmt = (taxable * 0.05).round(2);
    invoiceVal = taxable + gstAmt;
    daysCount = (endDate.daysBetween(startDate)) + 1;
    monthStr = startDate.toString("MMMM yyyy");
    
    // Insert permanent Rental_Log record
    newLog = insert into Rental_Log
    [
        Bed_Station = stationRec.ID,
        Customer = agrRec.Customer,
        Rental_Agreement = agrRec.ID,
        Rental_Item = agrRec.Rental_Item,
        Rental_Item = agrRec.Rental_Item,
        Rental_Mode = "Fixed Monthly Rental",
        Control_Unit_Serial_No = stationRec.Control_Unit_Serial_No,
        Overlay_Serial_No = stationRec.Overlay_Serial_No,
        Control_Unit_Asset = stationRec.Current_Control_Unit_Asset,
        Overlay_Asset = stationRec.Current_Overlay_Asset,
        Billing_Month = monthStr,
        Usage_Start_Date = startDate,
        Usage_End_Date = endDate,
        Number_of_Days = daysCount,
        Agreed_Rate = rate,
        Discount_Amount = ifNull(discount, 0.0),
        Taxable_Value = taxable,
        GST_Rate = 5.0,
        GST_Amount = gstAmt,
        Invoice_Value = invoiceVal,
        Billing_Status = "Pending Invoice",
        Billing_Notes = notes
    ];
    
    alert "SUCCESS: Fixed Monthly Rental Log generated for " + monthStr + " (Invoice Value: ₹" + invoiceVal + "). Ready for invoicing!";
}
```
