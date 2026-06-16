# Stage 4: Boyce-Codd Normal Form (BCNF), Group 8

## What we did

BCNF is stricter than 3NF: if columns X determine Y, then X must be a **candidate key**, not just part of the key.

We checked all 12 tables from 3NF. Ten were fine. Two were not: `supplier_materials_3nf` and `project_equipment_3nf` had the worker’s name in the key, but price and rental cost did not really depend on the worker.

### What we looked at as a group

Bonae asked: *on P001, Mike and Rachel both get Concrete at $120. Why was WorkerName in the key at all?*

Alice asked: *why split into four new files instead of fixing the two old ones in place?*

Veronicah asked: *could we drop WorkerName from the key and keep one materials table?*

Emmanuel asked: *same material name, different prices on different projects. Does that mean we need ProjectID in the price key?*

| Question | Our decision | Why |
|----------|--------------|-----|
| Worker in the key? | **No, remove it from pricing** | (ProjectID, Material) decides UnitCost; worker only decides *who uses* the material. |
| Four new CSVs? | **Yes: price + usage for each** | One table for the rate, one for who uses it. Join on ProjectID + Material (or Equipment). |
| One materials table? | **No, split in two** | Mixing price and usage in one row hid the BCNF problem. |
| ProjectID in price key? | **Yes** | Concrete is 120 on P001 and 110 on P002. Price follows project + material, not material name alone. |

**Bottom line:** Only two tables broke BCNF. We split each into a **pricing/rental** file and a **worker usage** file.

---
# Question and Answers

## Which functional dependencies did we check?

For each table we asked:

1. What is the primary key?
2. Are there any non-key columns?
3. Could a **smaller** set of columns still determine a non-key value?
4. Is every “determinant” actually a candidate key?

### Summary of what we found

| Table | Result |
|-------|--------|
| `sites_3nf`, `projects_3nf`, `clients_3nf`, `supervisors_3nf`, `workers_3nf`, `suppliers_3nf` | ✓ Fine: key determines everything |
| `project_worker_assignments_3nf` | ✓ Fine: (ProjectID, WorkerName) → SupplierName |
| `worker_skills_3nf`, `worker_certifications_3nf`, `supplier_phones_3nf` | ✓ Fine: no extra columns |
| **`supplier_materials_3nf`** | ✗ **Problem found** |
| **`project_equipment_3nf`** | ✗ **Problem found** |

### Rules we tested but that do **not** hold in our data

| We tested | Result | Example |
|-----------|--------|---------|
| (SupplierName, Material) → UnitCost | **No** | Steel Beams from SteelWorks: $500 on P004, $520 on P007 |
| Material → UnitCost | **No** | “Concrete” ranges from $110 to $140 |
| Equipment → RentalCost | **No** | “Crane” ranges from $5000 to $6500 |
| ProjectID → SupplierName | **No** | P001 uses both BuildPro and SteelWorks |

---

## Did you find any determinant that was not a candidate key?

**Yes: two cases.**

### 1. Material prices (`supplier_materials_3nf`)

**Key was:** (ProjectID, WorkerName, Material) → UnitCost

**But really:** On P001, Concrete is **$120** for both Mike Ross and Rachel Zane. The worker’s name does not matter; only **project + material** matter.

```
P001, Mike Ross,   Concrete, 120
P001, Rachel Zane, Concrete, 120   ← same price, different worker
```

So **(ProjectID, Material) → UnitCost**, but (ProjectID, Material) was **not** the primary key. That breaks BCNF.

### 2. Equipment rental (`project_equipment_3nf`)

**Key was:** (ProjectID, WorkerName, Equipment) → RentalCost

**But really:** On P001, Crane rental is **$5000** for Mike, Rachel, and Harvey.

```
P001, Mike Ross,      Crane, 5000
P001, Rachel Zane,    Crane, 5000
P001, Harvey Specter, Crane, 5000   ← same rental, different worker
```

So **(ProjectID, Equipment) → RentalCost**, but that pair was not the primary key. BCNF violation again.

---

## Which tables violated BCNF?

| Table | What was wrong |
|-------|----------------|
| `supplier_materials_3nf` | UnitCost decided by project + material, not worker |
| `project_equipment_3nf` | RentalCost decided by project + equipment, not worker |

---

## How did you decompose them?

We split each problem table into **“the price/rate”** and **“who uses it”**.

### Materials

**`project_material_pricing_bcnf.csv`**: the price list per project  
*(ProjectID + Material → UnitCost)*

```
P001 | Concrete | 120
P001 | Steel    | 300
```

**`project_worker_materials_bcnf.csv`**: which worker uses which material (no price column)

```
P001 | Mike Ross   | Concrete
P001 | Rachel Zane | Concrete
```

Join both on ProjectID + Material to get the full picture.

### Equipment

**`project_equipment_rental_bcnf.csv`**: rental rate per project  
*(ProjectID + Equipment → RentalCost)*

```
P001 | Crane     | 5000
P001 | Bulldozer | 3000
```

**`project_worker_equipment_bcnf.csv`**: which worker uses which machine

```
P001 | Mike Ross      | Crane
P001 | Harvey Specter | Crane
```

---

## What candidate keys exist after decomposition?

| Table | Primary key |
|-------|-------------|
| `project_material_pricing_bcnf` | **(ProjectID, Material)**: now matches what actually determines price |
| `project_worker_materials_bcnf` | **(ProjectID, WorkerName, Material)** |
| `project_equipment_rental_bcnf` | **(ProjectID, Equipment)**: now matches what actually determines rental |
| `project_worker_equipment_bcnf` | **(ProjectID, WorkerName, Equipment)** |

All other tables kept the same keys as 3NF (see `sites_bcnf`, `projects_bcnf`, etc.).

---

## Why this is in BCNF now

1. Still in 3NF.
2. **Every** “X → Y” rule has X as a candidate key; we fixed the two places where that was not true.
3. Joins get the data back exactly; nothing deleted, just reorganized.
4. Foreign keys tie usage tables to pricing/rental tables.

**Example: one 3NF row split into two BCNF rows:**

```
Before:  P001 | Mike Ross | Crane | 5000

After:
  project_equipment_rental_bcnf:   P001 | Crane | 5000
  project_worker_equipment_bcnf:   P001 | Mike Ross | Crane
```

---

## Files in this folder

```
BCNF/
  sites_bcnf.csv
  projects_bcnf.csv
  clients_bcnf.csv
  supervisors_bcnf.csv
  workers_bcnf.csv
  suppliers_bcnf.csv
  project_worker_assignments_bcnf.csv
  worker_skills_bcnf.csv
  worker_certifications_bcnf.csv
  supplier_phones_bcnf.csv
  project_material_pricing_bcnf.csv      ← NEW
  project_worker_materials_bcnf.csv      ← NEW
  project_equipment_rental_bcnf.csv      ← NEW
  project_worker_equipment_bcnf.csv      ← NEW
  BCNF_explanation.md
```

`supplier_materials_3nf.csv` and `project_equipment_3nf.csv` are replaced by the four files above.

---

## What is next (4NF)?

Some facts are **lists that do not depend on each other**: skills vs certifications, phones vs materials. Stage 5 separates those so we never invent false combinations.
