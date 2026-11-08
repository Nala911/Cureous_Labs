# Cureous PRS — Antigravity Error Tracer & Debugging Guide

This guide explains how errors are captured across all Zoho Creator Deluge scripts and how to copy-paste them directly to Antigravity for instant fixes.

---

## 1. How It Works

Every Deluge script in the project is structured with **Execution Step Breadcrumbs** and a standardized **Antigravity Error Block**.

When an unexpected error occurs during testing:
- **During Form Validation (`On Validate`)**: An on-screen alert pops up with the exact error report.
- **After Form Submission (`On Success`)**: The exact error report is printed to Zoho Creator's `info` console log.

```text
=================== 🚨 ANTIGRAVITY ERROR REPORT ===================
File    : scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge
Step    : Step 3: Hardware Conflict Guard
Record  : 389472900000481234
Error   : Invalid argument type: NullPointerException
Context : {"Control_Unit": 389472900000112233, "Overlay_Asset": null}
==================================================================
```

---

## 2. How to Retrieve the Error Log

### Scenario A: Error on Form Submission (`On Validate`)
1. You fill a form and click **Submit**.
2. If an error occurs, an on-screen alert dialog pops up immediately.
3. **Select all text** inside the dialog (or press `Ctrl+A`, `Ctrl+C`).
4. Paste it directly into your Antigravity chat.

### Scenario B: Error in Post-Save Cascading Updates (`On Success`)
1. In Zoho Creator, navigate to **Settings -> Developer Space -> Logs** (or check the Deluge Script Builder test console).
2. Look for the latest execution log containing `🚨 ANTIGRAVITY ERROR REPORT`.
3. Copy the block and paste it directly into Antigravity.

---

## 3. What Antigravity Does Automatically

When you paste an `🚨 ANTIGRAVITY ERROR REPORT`:
1. **Finds File**: Reads `File` and opens the exact file in the workspace (e.g. `scripts/08_bed_stations/3_Bed_Stations_onvalidate.deluge`).
2. **Locates Line**: Jumps to `Step` in the code.
3. **Diagnoses Bug**: Inspects `Context` against `Error` to identify why the variable was null, mismatched, or broken.
4. **Applies Fix**: Directly edits the script file with the permanent fix.
5. **Provides Instructions**: Tells you where to copy the updated snippet back into Zoho Creator.

---

## 4. Standard Deluge Template

All Deluge scripts follow this lightweight pattern:

```deluge
current_step = "1. Parse & initialize";
debug_ctx = Map();
debug_ctx.put("Record_ID", input.ID);

try 
{
    // Step 1
    current_step = "1. Process specific step";
    // ... code ...
    
    // Step 2
    current_step = "2. Process next step";
    // ... code ...
}
catch (e)
{
    err_payload = "=================== 🚨 ANTIGRAVITY ERROR REPORT ===================\n" +
                  "File    : <scripts/.../filename.deluge>\n" +
                  "Step    : " + current_step + "\n" +
                  "Record  : " + ifnull(input.ID, "New_Record").toString() + "\n" +
                  "Error   : " + e.toString() + "\n" +
                  "Context : " + debug_ctx.toString() + "\n" +
                  "==================================================================";
    
    info err_payload;
    alert err_payload; // Included in On Validate
    cancel submit;     // Included in On Validate
}
```
