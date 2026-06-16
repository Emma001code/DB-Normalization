# Stage 5: Fourth Normal Form (4NF) — Group 8

## What we did

**Group 8:** Bonae Ineza, Emmanuel Ngwoke, Alice Uwase, Veronicah Wanjuu

After BCNF, our tables were logically tight — but the **raw CSV still had a deeper problem** hiding in plain sight: **several independent lists were stored as if they belonged together**.

Emmanuel described it like this: imagine Alice writing on one line that a worker has skills `Carpentry|Framing` **and** certifications `OSHA|First Aid`. If you expand that into one table without thinking, you can accidentally create pairings that never existed — like “Framing + PMP” — even though the raw data never said that.

In 4NF, our job was to make sure **each independent list lives in its own table**, so joining tables never invents fake facts.

We started separating lists back in **1NF** (skills vs certs, materials vs equipment, phones vs materials). In **4NF**, we finished the job: we verified every BCNF table, renamed a few files for clarity, and added tables where two independent lists were still bundled together (especially **project workers** vs **project suppliers**).

### What we discussed as a group (why 4NF matters for our data)

After BCNF, Veronicah asked: *“Aren’t we done? Every cell is atomic and keys look correct.”*

Bonae answered with a raw-data example from P001:
- Workers on the project: Mike, Rachel, Harvey
- Suppliers on the project: BuildPro and SteelWorks
- Those are **two separate lists** — not every worker uses every supplier

Alice added another example from the same raw rows:
- `WorkerSkills` and `WorkerCertifications` are both lists, but **independent** (OSHA does not “belong to” Carpentry)
- `MaterialSupplied` and `EquipmentUsed` are also independent (Concrete does not determine Crane)

Emmanuel summarized the 4NF rule for our group:

> **If one thing has two unrelated multi-value lists, don’t store them in a way that forces combinations.**

That discussion drove our final splits:

| Mixed lists in raw / BCNF | Our 4NF separation |
|---------------------------|-------------------|
| Skills + Certifications | `assignment_skills_4nf` + `assignment_certifications_4nf` |
| Materials + Equipment usage | `assignment_materials_4nf` + `assignment_equipment_4nf` |
| Project workers + project suppliers | `project_workers_4nf` + `project_suppliers_4nf` + `worker_supplier_assignments_4nf` |
| Supplier phones + supplier materials | `supplier_phones_4nf` + `supplier_materials_4nf` |

**Bottom line (all four of us):** 4NF is about **independent lists**, not just “one value per cell.” Our final design stores each list once, then links them only where the raw data proves a real relationship.

---
# Questions and Answers

## Which multi-valued dependencies did you identify?

A **multi-valued dependency** (MVD) means: for one project/worker/supplier, there is a **set** of values — and that set does not determine another set.

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
- Workers ⊥ Suppliers on a project (P001: 3 workers, 2 suppliers — not a full cross-product)
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

`project_workers_4nf` — who works on the project:
```
P001 | Mike Ross
P001 | Rachel Zane
P001 | Harvey Specter
```

`project_suppliers_4nf` — which suppliers the project uses:
```
P001 | BuildPro Supplies
P001 | SteelWorks Inc
```

`worker_supplier_assignments_4nf` — which supplier each worker actually uses:
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

1. **Still in BCNF** — every functional dependency has a candidate-key determinant (Stage 4 fixes still apply).

2. **No table mixes two unrelated lists.** Skills are separate from certs; materials separate from equipment; workers/suppliers lists are separate and only linked through `worker_supplier_assignments_4nf`; phones separate from supplier materials.

3. **Joining does not invent fake rows.** If you join skills and certifications on (ProjectID, WorkerName), you only combine values that exist independently in each list — you don’t force every skill to pair with every cert.

4. **All raw facts can be recovered** by joining the 4NF tables — nothing was deleted, only reorganized.

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
| **1NF** | Split `\|` cells — one value per cell |
| **2NF** | Stop copying project/client/worker/supplier facts on every row |
| **3NF** | Stop chaining facts (address → city; client name → phone) |
| **BCNF** | Price/rental keys match what actually determines them (not worker name) |
| **4NF** | Separate independent lists so joins never create fake combinations |

This completes our normalization of `big3_construction_raw_data.csv`.

---

## Files in this folder

```
4NF/
  sites_4nf.csv
  projects_4nf.csv
  clients_4nf.csv
  supervisors_4nf.csv
  workers_4nf.csv
  suppliers_4nf.csv
  project_workers_4nf.csv
  project_suppliers_4nf.csv
  worker_supplier_assignments_4nf.csv
  assignment_skills_4nf.csv
  assignment_certifications_4nf.csv
  assignment_materials_4nf.csv
  assignment_equipment_4nf.csv
  project_material_pricing_4nf.csv
  project_equipment_rental_4nf.csv
  supplier_phones_4nf.csv
  supplier_materials_4nf.csv
  4NF_explanation.md
```
