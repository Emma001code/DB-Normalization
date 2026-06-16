# Stage 1: First Normal Form (1NF), Group 8

## What we did

**Group 8:** Bonae Ineza, Emmanuel Ngwoke, Alice Uwase, Veronicah Wanjuu

Our raw file was one huge spreadsheet where a single cell sometimes held **several values at once**, separated by a pipe character (`|`). That breaks the basic rule of a proper table: **one fact per cell**.

In Stage 1, we fixed that. We did **not** try to make the design perfect yet, we only made every value **atomic** (single and separate). Repeated project names and client phone numbers are still there on purpose; we clean that up in Stages 2 and 3.

Description: imagine Bonae unpacking a shopping list that says `eggs|milk|bread`; she would write **three separate lines**, one item each, instead of three items crammed into one cell. That is exactly what we did with every `|` column in the raw CSV.

### What we looked at as a group (and why we kept 1NF simple)

Bonae and Alice asked: *why are client columns still on `project_worker_assignments_1nf.csv`?* Veronicah asked: *why not put `SupplierCity` in `supplier_phones_1nf.csv`?* Emmanuel asked: *why not remove `SupplierName` from the assignment table?*

We discussed both options and agreed on this:


| Idea                                   | Our decision            | Why                                                                                                                                 |
| -------------------------------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Move client to its own CSV now         | **No, wait for 2NF**    | Splitting clients fixes **repetition**, not `|` lists. That is Stage 2 work.                                                        |
| Put `SupplierCity` in the phones file  | **No, wait for 2NF**    | City is one value per supplier; phones are a list. Mixing them repeats Boston on every BuildPro phone row.                          |
| Remove `SupplierName` from assignments | **No, keep it for now** | On P001, Mike uses BuildPro and Harvey uses SteelWorks. We need that link on the assignment row until 2NF splits entities properly. |


**Bottom line (all four of us):** 1NF = make cells atomic. 2NF = stop copying project, client, worker, and supplier data everywhere. Show the design improving **one stage at a time**.

---

# Question and Answers

## Which columns violated 1NF?

Seven columns had multiple values stuffed into one cell:


| Column                 | What was wrong                | Real example from our file  |
| ---------------------- | ----------------------------- | --------------------------- |
| `WorkerSkills`         | Two skills in one cell        | `Carpentry|Framing`         |
| `WorkerCertifications` | Two certs in one cell         | `OSHA|First Aid`            |
| `SupplierPhones`       | Two phone numbers in one cell | `617-555-9000|617-555-9001` |
| `MaterialSupplied`     | Two materials in one cell     | `Concrete|Steel`            |
| `MaterialUnitCost`     | Two prices in one cell        | `120|300`                   |
| `EquipmentUsed`        | Two machines in one cell      | `Crane|Bulldozer`           |
| `EquipmentRentalCost`  | Two rental fees in one cell   | `5000|3000`                 |


The other 20 columns were already fine: one value per cell.

---

## How did we make the values atomic?

We used three simple rules:

**Rule 1: Keep one main table for everything that is already single-valued.**

File: `project_worker_assignments_1nf.csv`  
Same 15 rows as the raw file, but we removed all seven `|` columns.

**Rule 2: When two columns were paired, split them together.**

`MaterialSupplied` and `MaterialUnitCost` belong together: the first price goes with the first material:

- Raw: `Concrete\|Steel` and `120\|300`
- Becomes: Concrete → 120, Steel → 300 (in `supplier_materials_1nf.csv`)

Same idea for `EquipmentUsed` and `EquipmentRentalCost` in `project_equipment_1nf.csv`.

**Rule 3: Give each list its own table.**

- Skills → `worker_skills_1nf.csv`
- Certifications → `worker_certifications_1nf.csv`
- Supplier phones → `supplier_phones_1nf.csv`

We deliberately **did not** put skills and certifications in one big table. If we did, we would accidentally pair every skill with every cert (e.g. “Carpentry + PMP”) even when that is not true. We fixed that properly in Stage 5 (4NF).

---

## Did you create new rows, new tables, or both?

**Both.**

- **New tables:** 5 extra files for the list-type data (skills, certs, phones, materials, equipment).
- **New rows:** 15 raw rows became **133 rows** total across all 1NF files.


| File                                 | Rows | What it holds                          |
| ------------------------------------ | ---- | -------------------------------------- |
| `project_worker_assignments_1nf.csv` | 15   | Main row minus the `                   |
| `worker_skills_1nf.csv`              | 30   | One skill per row                      |
| `worker_certifications_1nf.csv`      | 30   | One certification per row              |
| `supplier_phones_1nf.csv`            | 8    | One phone per row (duplicates removed) |
| `supplier_materials_1nf.csv`         | 25   | One material + cost per row            |
| `project_equipment_1nf.csv`          | 25   | One equipment + rental cost per row    |


---

## What key or combination of keys identifies each row?


| Table                            | Primary key                                | What one row means                        |
| -------------------------------- | ------------------------------------------ | ----------------------------------------- |
| `project_worker_assignments_1nf` | **(ProjectID, WorkerName)**                | Mike Ross working on project P001         |
| `worker_skills_1nf`              | **(ProjectID, WorkerName, Skill)**         | One of Mike’s skills on P001              |
| `worker_certifications_1nf`      | **(ProjectID, WorkerName, Certification)** | One of Mike’s certs on P001               |
| `supplier_phones_1nf`            | **(SupplierName, Phone)**                  | One phone number for BuildPro             |
| `supplier_materials_1nf`         | **(ProjectID, WorkerName, Material)**      | One material on that assignment           |
| `project_equipment_1nf`          | **(ProjectID, WorkerName, Equipment)**     | One piece of equipment on that assignment |


---

## Walk-through: raw row 2 → 1NF

Raw row 2 is **P001, Mike Ross** with lists everywhere.


| Goes into                        | What we wrote                               |
| -------------------------------- | ------------------------------------------- |
| `project_worker_assignments_1nf` | One clean row for P001 + Mike Ross (no `|`) |
| `worker_skills_1nf`              | P001, Mike Ross, Carpentry                  |
|                                  | P001, Mike Ross, Framing                    |
| `worker_certifications_1nf`      | P001, Mike Ross, OSHA                       |
|                                  | P001, Mike Ross, First Aid                  |
| `supplier_phones_1nf`            | BuildPro Supplies, 617-555-9000             |
|                                  | BuildPro Supplies, 617-555-9001             |
| `supplier_materials_1nf`         | P001, Mike Ross, Concrete, 120              |
|                                  | P001, Mike Ross, Steel, 300                 |
| `project_equipment_1nf`          | P001, Mike Ross, Crane, 5000                |
|                                  | P001, Mike Ross, Bulldozer, 3000            |


---

## Why this is in 1NF now

1. Every cell has **exactly one value**: no more `|` anywhere.
2. Repeating groups (lists inside cells) are **gone**.
3. Every row can be **uniquely identified** with the keys above.

We still have repeated data (e.g. “Downtown Plaza” copied on three P001 rows). That is normal at this stage and gets fixed next.

---

## Files in this folder

```
1NF/
  project_worker_assignments_1nf.csv
  worker_skills_1nf.csv
  worker_certifications_1nf.csv
  supplier_phones_1nf.csv
  supplier_materials_1nf.csv
  project_equipment_1nf.csv
  1NF_explanation.md
```

