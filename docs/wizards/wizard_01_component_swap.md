# WIZARD 1: STATELESS COMPONENT SWAP WIZARD

**Form Name**: `Swap_Component_Wizard`  
**Form Type**: Stateless Form (Popup Dialog, Height: 520px, Width: 650px)  
**Trigger**: Custom Action Button `[ 🔄 Swap Component ]` on `Bed_Station` report or detail view.  
**URL Invocation**:
```deluge
openUrl("#Form:Swap_Component_Wizard?Bed_Station_ID=" + input.ID, "popupWindow", "height=520,width=650");
```

---

## 1. Operational Purpose
Enables a field service technician at the patient bedside to replace a single failing component (e.g. broken Control Unit) without disturbing the intact component (e.g. clean Overlay mattress). Maintains continuous patient billing and atomic historical pairing snapshots in `Bed_Station_Log`.

---

## 2. Fields on the Stateless Form

1. **Target Bed Station ID** `[Hidden / Single Line]` (Passed via URL parameter)
2. **Bed Station Display** `[Single Line - Read Only]` (Auto-loaded: e.g. "Apollo Hospital - ICU Bed 04 (BST-2026-0042)")
3. **Component to Swap** `[Dropdown]` (Options: `"Control Unit"`, `"Overlay"`, `"Both"`)
4. **Current Control Unit** `[Single Line - Read Only]` (Auto-loaded serial)
5. **Current Overlay** `[Single Line - Read Only]` (Auto-loaded serial)
6. **Reason for Replacement** `[Dropdown]` (Mandatory: "Low Pressure Pump Failure", "PCB / Alarm Malfunction", "Mattress Cell Puncture", "Cover Fluid Ingress / Biological Contamination", "Routine Preventive Maintenance")
7. **Replacement CU Asset** `[Lookup -> Devices]` (Shown if Component is "Control Unit" or "Both"; Filtered: Device_Type=Control Unit, Customer Site)
8. **Replacement Overlay Asset** `[Lookup -> Devices]` (Shown if Component is "Overlay" or "Both"; Filtered: Device_Type=Overlay, Customer Site)
9. **Technician Notes** `[Multi-Line]` (Fault codes or bedside remarks)
10. **Submit Button**: `"Confirm & Execute Swap"`

---

## 3. Deluge Execution Logic (On Submit)

```deluge
swapTime = zoho.currenttime;
bedRec = Bed_Station[ID == input.Target_Bed_Station_ID.toLong()];
customerID = bedRec.Customer;

// 1. Close current open snapshot in Bed_Station_Log
openLog = Bed_Station_Log[Bed_Station == bedRec.ID && Period_End_Date == null];
if(openLog.count() > 0)
{
    update Bed_Station_Log[ID == openLog.ID] [ Period_End_Date = swapTime ];
}

// 2. Resolve Old vs New Asset Selections
oldCU = bedRec.Current_Control_Unit_Asset;
oldOV = bedRec.Current_Overlay_Asset;
newCU = oldCU;
newOV = oldOV;

if(input.Component_to_Swap == "Control Unit" || input.Component_to_Swap == "Both")
{
    newCU = input.Replacement_CU_Asset;
    update Devices[ID == oldCU] [ Rental_Stage = "Defective at Customer Site" ];
    update Devices[ID == newCU] [ Rental_Stage = "Deployed - Active on Bed" ];
}
if(input.Component_to_Swap == "Overlay" || input.Component_to_Swap == "Both")
{
    newOV = input.Replacement_Overlay_Asset;
    update Devices[ID == oldOV] [ Rental_Stage = "Defective at Customer Site" ];
    update Devices[ID == newOV] [ Rental_Stage = "Deployed - Active on Bed" ];
}

// 3. Create NEW open snapshot in Bed_Station_Log
insert into Bed_Station_Log
[
    Bed_Station = bedRec.ID,
    Bed_Station_Title = bedRec.Bed_Station_Title,
    Customer = customerID,
    Rental_Agreement = bedRec.Rental_Agreement,
    Control_Unit_Asset = newCU,
    Overlay_Asset = newOV,
    Period_Start_Date = swapTime,
    Period_End_Date = null,
    Trigger_Event = "Component Swap",
    Component_Swapped = input.Component_to_Swap,
    Swap_Reason = input.Reason_for_Replacement,
    Technician_Name = zoho.loginuser,
    Technician_Notes = input.Technician_Notes
];

// 4. Update parent Bed_Station live pairing & Last_Changed_Date in-place
update Bed_Station[ID == bedRec.ID]
[
    Current_Control_Unit_Asset = newCU,
    Current_Overlay_Asset = newOV,
    Last_Changed_Date = swapTime
];

alert "SUCCESS: " + input.Component_to_Swap + " swapped successfully on " + bedRec.Bed_Station_Title + "! New pairing log recorded.";
```
