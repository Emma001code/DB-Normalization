# Stage 3: Third Normal Form (3NF) — Group 8

## What we did

2NF stopped columns from depending on **half** of a composite key. **3NF** fixes a different problem: columns that depend on **other non-key columns** instead of depending directly on the primary key.

Simple picture: **Project → Site Address → City/State**

The city and state really belong to the **address**, not to the project ID itself. If we store city and state on the project row, we are going through an extra hop. In 3NF, we give the address its own table.

Most of our other “A → B → C” chains were already fixed in 2NF when we made separate client, worker, and supplier tables. Stage 3 mainly adds the **sites** table and double-checks everything else.

---

## Which transitive dependencies did we find?

### Fixed in this stage — site location on projects

| Chain | What it means |
|-------|---------------|
| ProjectID → SiteAddress → **SiteCity** | City follows from the address, not from the project code |
| ProjectID → SiteAddress → **SiteState** | Same for state |

Every project in our file has its own address, and each address has exactly one city and state:

| Address | City | State |
|---------|------|-------|
| 123 Main St | Boston | MA |
| 78 Harbor Rd | Boston | MA |
| 45 River Dr | Providence | RI |
| 900 Innovation Way | New York | NY |
| 300 Sunset Blvd | Los Angeles | CA |
| 88 Lakeview Rd | Orlando | FL |
| 1 Airport Way | Chicago | IL |

**Fix:** New file `sites_3nf.csv`. Removed SiteCity and SiteState from `projects_3nf.csv`.

---

### Already fixed in 2NF (we verified they stay correct)

These chains were in the **raw mess**, but we broke them when we split entity tables:

| Chain in raw data | Where it lives now |
|-------------------|-------------------|
| Project → ClientName → phone, email, city | `clients_3nf` |
| Project → SupervisorName → phone | `supervisors_3nf` |
| WorkerName → phone, hourly rate | `workers_3nf` |
| SupplierName → city | `suppliers_3nf` |

**Example:** Metro Corp is always `617-555-1000` / `contact@metrocorp.com`. We no longer copy that onto every assignment row — we store it once in `clients_3nf` and link with ClientName.

---

### Checked — no transitive problem found

| Table | Why it is fine |
|-------|----------------|
| `project_worker_assignments_3nf` | Only extra column is SupplierName (a link to suppliers) |
| `supplier_materials_3nf` | Concrete costs 120 on P001 but 110 on P002 — price is not a simple “material name → price” rule |
| `project_equipment_3nf` | Crane rental varies by project (5000 up to 6500) |
| Skills, certs, supplier phones | Key columns only |

---

## Which non-key attributes depended on other non-key attributes?

| Where | The chain | What we did |
|-------|-----------|-------------|
| `projects_2nf` | **SiteAddress → SiteCity, SiteState** | Moved city/state to `sites_3nf` |
| Raw / 1NF rows | ClientName → phone, email, city | Already in `clients_3nf` (Stage 2) |
| Raw / 1NF rows | SupervisorName → phone | Already in `supervisors_3nf` (Stage 2) |
| Raw / 1NF rows | WorkerName → phone, rate | Already in `workers_3nf` (Stage 2) |
| Raw / 1NF rows | SupplierName → city | Already in `suppliers_3nf` (Stage 2) |

On `projects_3nf`, ClientName and SupervisorName are just **pointers** — they do not carry phone numbers anymore, so there is no chain on that table except the site fix above.

---

## Which new tables did we create?

**New in Stage 3:**

| Table | Purpose | Rows |
|-------|---------|------|
| `sites_3nf.csv` | Address + city + state in one place | 7 |

**Updated:**

| Table | Change |
|-------|--------|
| `projects_3nf.csv` | Dropped SiteCity and SiteState; kept SiteAddress as a link |

Everything else is the same structure as 2NF, copied forward as `_3nf` files.

---

## How did we use foreign keys to preserve relationships?

| Table | Foreign key | Links to |
|-------|-------------|----------|
| `projects_3nf` | SiteAddress | `sites_3nf` — where is this project built? |
| `projects_3nf` | ClientName | `clients_3nf` — who hired us? |
| `projects_3nf` | SupervisorName | `supervisors_3nf` — who manages the site? |
| `project_worker_assignments_3nf` | ProjectID, WorkerName, SupplierName | projects, workers, suppliers |
| Skills, certs, materials, equipment tables | (ProjectID, WorkerName) | assignment row |

**Nothing was lost.** To get P001’s city, join projects to sites:

```
projects_3nf:  P001 | Downtown Plaza | ... | 123 Main St | Metro Corp | John Carter
sites_3nf:     123 Main St | Boston | MA
```

---

## Why this is in 3NF now

1. Still in 2NF — no partial dependencies.
2. **No non-key column depends on another non-key column** within the same table.
3. Facts about clients, workers, suppliers, and sites are stored **once** and linked, not copied through a chain.

**Next:** In BCNF we look at trickier rules — like whether material price really needs the worker’s name in the key (spoiler: it does not always).

---

## Files in this folder

```
3NF/
  sites_3nf.csv                 ← NEW
  projects_3nf.csv              ← updated
  clients_3nf.csv
  supervisors_3nf.csv
  workers_3nf.csv
  suppliers_3nf.csv
  project_worker_assignments_3nf.csv
  worker_skills_3nf.csv
  worker_certifications_3nf.csv
  supplier_phones_3nf.csv
  supplier_materials_3nf.csv
  project_equipment_3nf.csv
  3NF_explanation.md
```
