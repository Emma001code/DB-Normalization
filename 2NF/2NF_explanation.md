# Stage 2: Second Normal Form (2NF) — Group 8

## What we did

After 1NF, every cell had one value, good. But our main table still had a problem: **the same information was copied over and over**.

Example: “Downtown Plaza” and Metro Corp’s phone number appeared on **every** P001 row, just because Mike, Rachel, and Harvey each had their own row. That happened because some columns only depend on **part** of the row’s ID (like ProjectID alone), not the full combination of ProjectID + WorkerName.

In 2NF, we moved those “partially dependent” facts into **their own tables** so each fact is stored **once**.

### What we discussed as a group (why these splits belong to 2NF)

After we finished 1NF, Bonae pointed out that our 1NF “assignment” table still looked like a copy‑paste sheet: P001’s project name, Metro Corp’s phone/email, and John Carter’s phone were repeated on multiple rows.

Emmanuel explained the key idea of 2NF in one sentence: **if our row key is (ProjectID, WorkerName), then any column that depends on only ProjectID or only WorkerName does not belong on the assignment table.**

Alice helped us list the “depends on ProjectID only” columns (project details, site, client, supervisor) and Veronicah helped us list the “depends on WorkerName only” columns (worker phone and hourly rate). That discussion is what drove our decompositions:

- We created **entity tables** (`projects_2nf`, `clients_2nf`, `supervisors_2nf`, `workers_2nf`, `suppliers_2nf`) so each fact is stored once.
- We kept one **relationship table** (`project_worker_assignments_2nf`) that holds only the assignment-level fact: which supplier each worker uses on a project.

Why we kept `SupplierName` on the assignment table:
- On project P001, Mike and Rachel use BuildPro but Harvey uses SteelWorks.
- So supplier is not just “per project”; it is tied to **the worker on that project**, which matches the composite key (ProjectID, WorkerName).

---
# Questions and Answer

## Which tables had composite keys?

A **composite key** means the row is identified by **two or more columns together**, not one column alone.

| Table | Composite key |
|-------|---------------|
| `project_worker_assignments_1nf` | **(ProjectID, WorkerName)** |
| `worker_skills_1nf` | **(ProjectID, WorkerName, Skill)** |
| `worker_certifications_1nf` | **(ProjectID, WorkerName, Certification)** |
| `supplier_materials_1nf` | **(ProjectID, WorkerName, Material)** |
| `project_equipment_1nf` | **(ProjectID, WorkerName, Equipment)** |
| `supplier_phones_1nf` | **(SupplierName, Phone)** — already fine |

The big problem was **`project_worker_assignments_1nf`**. It had 19 columns but the row key was only ProjectID + WorkerName.

---

## Which columns depended only on part of the composite key?

### Main table: `project_worker_assignments_1nf`

**Depends only on ProjectID** (not on which worker):
- Project name, type, dates, site address, city, state
- Client name, phone, email, city
- Supervisor name and phone

*Why?* P001 is always “Downtown Plaza” in Boston — that does not change because the worker is Mike or Rachel.

**Depends only on WorkerName** (not on which project):
- Worker phone and hourly rate

*Why?* Mike Ross is always `617-555-3001` at `$45/hr`, whether he is on P001, P002, or P004.

**Depends only on SupplierName**:
- Supplier city

*Why?* BuildPro Supplies is always in Boston.

**Depends on the full key (ProjectID + WorkerName)**:
- **SupplierName** on the assignment

*Why?* On P001, Mike and Rachel use BuildPro, but Harvey uses SteelWorks. So supplier is tied to **that worker on that project**, not to ProjectID alone.

### Also: `supplier_materials_1nf`

**SupplierName** depended on **(ProjectID, WorkerName)** only — not on the material. Every material row for Mike on P001 had BuildPro Supplies. We removed SupplierName from that table and look it up through the assignment instead.

### Already in 2NF (no changes needed)

- `worker_skills_1nf` and `worker_certifications_1nf` — only key columns, nothing extra
- `supplier_phones_1nf` — phone is part of the key
- `project_equipment_1nf` — rental cost really does depend on the full assignment + equipment

---

## Which data did we move into separate tables?

We broke the fat assignment table into **one slim link table** plus **five entity tables**:

| New table | What we moved there | Rows |
|-----------|---------------------|------|
| `projects_2nf.csv` | Project details + client & supervisor **names** | 7 |
| `clients_2nf.csv` | Client phone, email, city | 4 |
| `supervisors_2nf.csv` | Supervisor phone | 3 |
| `workers_2nf.csv` | Worker phone, hourly rate | 5 |
| `suppliers_2nf.csv` | Supplier city | 6 |
| `project_worker_assignments_2nf.csv` | Only ProjectID, WorkerName, SupplierName | 15 |

We also cleaned `supplier_materials_2nf.csv` by dropping the redundant SupplierName column.

---

## What primary keys and foreign keys did we introduce?

### Primary keys (what makes each row unique)

| Table | Primary key |
|-------|-------------|
| `projects_2nf` | ProjectID |
| `clients_2nf` | ClientName |
| `supervisors_2nf` | SupervisorName |
| `workers_2nf` | WorkerName |
| `suppliers_2nf` | SupplierName |
| `project_worker_assignments_2nf` | (ProjectID, WorkerName) |

### Foreign keys (how tables connect)

| Child table | Points to |
|-------------|-----------|
| `projects_2nf` → ClientName | `clients_2nf` |
| `projects_2nf` → SupervisorName | `supervisors_2nf` |
| `project_worker_assignments_2nf` → ProjectID | `projects_2nf` |
| `project_worker_assignments_2nf` → WorkerName | `workers_2nf` |
| `project_worker_assignments_2nf` → SupplierName | `suppliers_2nf` |
| Skills, certs, materials, equipment tables → (ProjectID, WorkerName) | `project_worker_assignments_2nf` |

---

## Before and after example

**Before (1NF) — one crowded row for Mike on P001:**

```
P001 | Downtown Plaza | Commercial | ... | Metro Corp | 617-555-1000 | ... | John Carter | ... 
     | Mike Ross | 617-555-3001 | 45 | BuildPro Supplies | Boston
```

**After (2NF) — split into focused tables:**

`projects_2nf`:
```
P001 | Downtown Plaza | Commercial | ... | 123 Main St | Boston | MA | Metro Corp | John Carter
```

`clients_2nf`:
```
Metro Corp | 617-555-1000 | contact@metrocorp.com | Boston
```

`workers_2nf`:
```
Mike Ross | 617-555-3001 | 45
```

`project_worker_assignments_2nf`:
```
P001 | Mike Ross | BuildPro Supplies
```

Metro Corp’s phone is stored **once**, not on every worker row.

---

## Why this is in 2NF now

1. Still in 1NF — one value per cell.
2. On tables with composite keys, **every non-key column depends on the whole key**, not just part of it.
3. Project, client, worker, and supplier facts live in **one place each**.

**Note:** `projects_2nf` still stores ClientName and SupervisorName as links. The client’s phone is not on the project row anymore — but site city/state still sit on the project row. We fix that chain in **3NF**.

---

## Files in this folder

```
2NF/
  projects_2nf.csv
  clients_2nf.csv
  supervisors_2nf.csv
  workers_2nf.csv
  suppliers_2nf.csv
  project_worker_assignments_2nf.csv
  worker_skills_2nf.csv
  worker_certifications_2nf.csv
  supplier_phones_2nf.csv
  supplier_materials_2nf.csv
  project_equipment_2nf.csv
  2NF_explanation.md
```
