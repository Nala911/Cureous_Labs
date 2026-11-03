# Cureous Labs — Patient Repositioning System (PRS)

This repository hosts the Zoho Creator application architecture, schema definitions, master data dictionary, and production Deluge scripts for the Cureous Patient Repositioning System (PRS).

## Directory Structure

```text
Cureous_Labs/
├── docs/
│   ├── ARCHITECTURE.md          # System overview, lifecycle, and integration roles
│   ├── DATA_DICTIONARY.md       # Master field, link name, and dependency matrix
│   ├── modules/                 # Module specifications (01 to 11)
│   └── wizards/                 # Stateless bedside action workflows
├── scripts/                     # Standardized Deluge scripts organized by module
│   ├── 01_lots/
│   ├── 02_customers/
│   ├── 03_locations/
│   ├── 04_product_catalog/
│   ├── 05_devices/
│   ├── 06_rental_agreements/
│   ├── 07_stock_transfers/
│   ├── 08_bed_stations/
│   ├── 09_bed_station_logs/
│   ├── 10_rental_log/
│   ├── 11_qc_sanitization/
│   ├── wizards/
│   └── functions/
├── GEMINI.md                    # Core architectural protocol and development rules
├── .gitignore
└── README.md
```

## Daily Workflow for Multi-Device Collaboration

1. **Before starting work:**
   ```bash
   git pull origin main
   ```
2. **After making updates:**
   ```bash
   git add .
   git commit -m "Brief summary of changes"
   git push origin main
   ```
