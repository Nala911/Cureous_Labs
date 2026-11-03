# WIZARD 3: STATELESS MOBILE DELIVERY HANDOVER WIZARD

**Form Name**: `Delivery_Handover_Wizard`  
**Form Type**: Stateless Form (Mobile Optimized)  
**Trigger**: Button `[ 📋 Complete Handover ]` on the mobile Stock Transfer view.  
**Purpose**: Enables the delivery driver on mobile to complete hospital handover in 30 seconds without navigating complex administrative menus.

---

## 1. Fields on the Stateless Form

1. **Stock Transfer Ref** `[Hidden / Lookup -> Stock_Transfers]`
2. **Hospital Receiving Person** `[Single Line]` (Name of nurse / biomedical staff)
3. **Contact Phone** `[Phone]`
4. **Digital Signature** `[Signature Field]` (Receiver signs with finger on screen)
5. **Delivery Photo** `[Upload Image]` (Optional photo of signed physical gate pass or equipment deployed in ICU)
6. **Submit Button**: `"Confirm Handover & Deliver Assets"`

---

## 2. Execution Outcome
- Updates parent `Stock_Transfers` record to `"Delivered"`.
- Captures legal proof of delivery (signature + photo).
- Automatically flips all line-item assets in `Devices` to customer custody (`Rental_Stage = "Deployed - Standby Buffer"` or active custody).
