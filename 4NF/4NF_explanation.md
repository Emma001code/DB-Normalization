# Stage 5: Fourth Normal Form (4NF), Group 8

## What we did

By BCNF, our tables were in good shape: one value per cell, sensible keys. But the **raw file** still mixed up **separate lists** on the same row.

Alice gave the simplest example from row 2 (Mike Ross on P001):
- Skills: `Carpentry|Framing`
- Certs: `OSHA|First Aid`

Those are **two different lists**. OSHA does not come with Carpentry. If we put both lists in one table and expand every skill with every cert, we invent rows we never had, like “Framing + PMP” when PMP was never on Mike’s row.

**4NF fix:** put each list in **its own table**. Join only when the raw data really connects them.

We already split `\|` lists in **1NF**. In **4NF** we finished by:
- keeping skills, certs, materials, and equipment in **separate** files
- splitting **who works on a project** from **which suppliers the project uses**
- splitting **supplier phones** from **supplier materials**

### What we looked at as a group (questions about our 4NF files)

Bonae asked: *why do we still have `project_worker_assignments_bcnf.csv` in BCNF but three files in 4NF (`project_workers_4nf`, `project_suppliers_4nf`, and `worker_supplier_assignments_4nf`)? Isn’t that the same information repeated?*

Veronicah asked: *we already have `assignment_materials_4nf`. Why add `supplier_materials_4nf` as well? Don’t both tables list materials?*

Alice asked: *why rename `worker_skills_bcnf` to `assignment_skills_4nf`? The data looks the same. Did anything actually change?*

Emmanuel asked: *could we put skills and certifications in one table now that every cell is single-valued? We’d just have more rows.*

We discussed each point and agreed on this:

| Question | Our decision | Why |
|----------|--------------|-----|
| One assignment file vs three project files | **Split into three** | P001 has 3 workers and 2 suppliers (separate lists). `worker_supplier_assignments_4nf` only stores real pairs (Mike→BuildPro, Harvey→SteelWorks), not all 6 combinations. |
| `assignment_materials_4nf` vs `supplier_materials_4nf` | **Keep both** | Assignment materials = what Mike used on P001. Supplier materials = what BuildPro can supply in general. Different questions, different tables. |
| Rename to `assignment_skills_4nf` | **Yes, rename** | Same rows as BCNF, but the name shows these skills belong to an **assignment**, not mixed with certs in one file. |
| One table for skills + certs | **No** | Cells would be atomic, but we’d still mix two **independent lists**. Joining them could invent pairs like “Framing + PMP” that never existed in the raw CSV. |

**Bottom line (all four of us):** 4NF is about **which files we keep separate**, not splitting `\|` again. If two lists don’t depend on each other, they stay in different CSVs.

---
# Questions and Answers

## Which multi-valued dependencies did you identify?

A **multi-valued dependency** (MVD) means: for one project/worker/supplier, there is a **set** of values, and that set does not determine another set.

| # | Multi-valued dependency | What it means in our data |
|---|-------------------------|---------------------------|
| 1 | **(ProjectID, WorkerName) →→ Skill** | Mike on P001 has multiple skills |
| 2 | **(ProjectID, WorkerName) →→ Certification** | Mike on P001 has multiple certs |
| 3 | **(ProjectID, WorkerName) →→ Material** | An assignment can use multiple materials |
| 4 | **(ProjectID, WorkerName) →→ Equipment** | An assignment can use multiple machines |
| 5 | **ProjectID →→ WorkerName** | P001 has multiple workers |
| 6 | **ProjectID →→ SupplierName** | P001 uses multiple suppliers |
| 7 | **SupplierName →→ Phone** | BuildPro has two phone numbers |
| 8 | **SupplierName →→ Material** | BuildPro supplies Concrete, Steel, Bricks, etc. |

**Independence examples (why 4NF is needed):**
- Skills ⊥ Certifications (raw row 2: `Carpentry|Framing` vs `OSHA|First Aid`)
- Materials ⊥ Equipment (raw row 2: `Concrete|Steel` vs `Crane|Bulldozer`)
- Workers ⊥ Suppliers on a project (P001: 3 workers, 2 suppliers, not a full cross-product)
- Phones ⊥ Materials for a supplier (raw row 2: two phones vs two materials)

---

## Which independent lists were mixed together?

| Where | Lists mixed together | Risk if left mixed |
|-------|----------------------|--------------------|
| Raw worker columns | Skills + Certifications | Fake skill–cert pairings |
| Raw supply columns | Materials + Equipment | Fake material–machine pairings |
| Raw supplier columns | Phones + Materials | Looks like phones “belong to” specific materials |
| Whole raw row | Workers + suppliers + materials + equipment | Explosion of false combinations if naively expanded |
| BCNF assignment table | Project workers + suppliers in one place | Hard to see workers list vs suppliers list independently |

---

## How did you separate them?

### Already separated earlier (kept and renamed in 4NF)

| Mixed in raw | 4NF tables |
|--------------|------------|
| Skills + Certs | `assignment_skills_4nf.csv` + `assignment_certifications_4nf.csv` |
| Materials + Equipment usage | `assignment_materials_4nf.csv` + `assignment_equipment_4nf.csv` |
| Prices / rentals (BCNF) | `project_material_pricing_4nf.csv` + `project_equipment_rental_4nf.csv` |

### New / clearer splits in 4NF

**1) Project workers ⊥ project suppliers**

Instead of only one assignment file, we use three tables:

`project_workers_4nf`: who works on the project:
```
P001 | Mike Ross
P001 | Rachel Zane
P001 | Harvey Specter
```

`project_suppliers_4nf`: which suppliers the project uses:
```
P001 | BuildPro Supplies
P001 | SteelWorks Inc
```

`worker_supplier_assignments_4nf`: which supplier each worker actually uses:
```
P001 | Mike Ross      | BuildPro Supplies
P001 | Rachel Zane    | BuildPro Supplies
P001 | Harvey Specter | SteelWorks Inc
```

We do **not** create `P001 | Mike Ross | SteelWorks Inc` because that never happened in the raw data.

**2) Supplier phones ⊥ supplier materials**

| Table | Holds |
|-------|--------|
| `supplier_phones_4nf.csv` | Phone numbers only |
| `supplier_materials_4nf.csv` | Materials each supplier can provide (catalog) |

Example:
```
supplier_phones_4nf:    BuildPro Supplies | 617-555-9000
supplier_materials_4nf: BuildPro Supplies | Concrete
```

---

## What are the primary keys of the new relationship tables?

| Table | Primary key | What one row means |
|-------|-------------|-------------------|
| `assignment_skills_4nf` | **(ProjectID, WorkerName, Skill)** | One skill on one assignment |
| `assignment_certifications_4nf` | **(ProjectID, WorkerName, Certification)** | One certification on one assignment |
| `assignment_materials_4nf` | **(ProjectID, WorkerName, Material)** | One material used on an assignment |
| `assignment_equipment_4nf` | **(ProjectID, WorkerName, Equipment)** | One machine used on an assignment |
| `project_workers_4nf` | **(ProjectID, WorkerName)** | Worker is assigned to this project |
| `project_suppliers_4nf` | **(ProjectID, SupplierName)** | Supplier is used on this project |
| `worker_supplier_assignments_4nf` | **(ProjectID, WorkerName)** | Which supplier that worker uses (one per assignment) |
| `supplier_materials_4nf` | **(SupplierName, Material)** | Supplier offers this material |
| `supplier_phones_4nf` | **(SupplierName, Phone)** | One supplier phone number |
| `project_material_pricing_4nf` | **(ProjectID, Material)** | Unit cost for that material on that project |
| `project_equipment_rental_4nf` | **(ProjectID, Equipment)** | Rental cost for that equipment on that project |

### Entity tables (unchanged logic from BCNF)

| Table | Primary key |
|-------|-------------|
| `sites_4nf` | SiteAddress |
| `projects_4nf` | ProjectID |
| `clients_4nf` | ClientName |
| `supervisors_4nf` | SupervisorName |
| `workers_4nf` | WorkerName |
| `suppliers_4nf` | SupplierName |

---

## Why is our final design in 4NF?

1. **Still in BCNF:** every functional dependency has a candidate-key determinant (Stage 4 fixes still apply).

2. **No table mixes two unrelated lists.** Skills are separate from certs; materials separate from equipment; workers/suppliers lists are separate and only linked through `worker_supplier_assignments_4nf`; phones separate from supplier materials.

3. **Joining does not invent fake rows.** If you join skills and certifications on (ProjectID, WorkerName), you only combine values that exist independently in each list; you don’t force every skill to pair with every cert.

4. **All raw facts can be recovered** by joining the 4NF tables; nothing was deleted, only reorganized.

---

## Walk-through: raw row 2 (P001, Mike Ross) → 4NF

Raw row 2 had lists everywhere. After 4NF, selected rows look like:

| Table | Row |
|-------|-----|
| `assignment_skills_4nf` | P001, Mike Ross, Carpentry |
| `assignment_skills_4nf` | P001, Mike Ross, Framing |
| `assignment_certifications_4nf` | P001, Mike Ross, OSHA |
| `assignment_certifications_4nf` | P001, Mike Ross, First Aid |
| `supplier_phones_4nf` | BuildPro Supplies, 617-555-9000 |
| `supplier_materials_4nf` | BuildPro Supplies, Concrete |
| `assignment_materials_4nf` | P001, Mike Ross, Concrete |
| `project_material_pricing_4nf` | P001, Concrete, 120 |
| `assignment_equipment_4nf` | P001, Mike Ross, Crane |
| `project_equipment_rental_4nf` | P001, Crane, 5000 |
| `worker_supplier_assignments_4nf` | P001, Mike Ross, BuildPro Supplies |

Each row says **one thing**.

---

## Our full normalization journey (Group 8)

| Stage | What we fixed |
|-------|----------------|
| **1NF** | Split `\|` cells: one value per cell |
| **2NF** | Stop copying project/client/worker/supplier facts on every row |
| **3NF** | Stop chaining facts (address → city; client name → phone) |
| **BCNF** | Price/rental keys match what actually determines them (not worker name) |
| **4NF** | Separate independent lists so joins never create fake combinations |

This completes our normalization of `big3_construction_raw_data.csv`.

---
