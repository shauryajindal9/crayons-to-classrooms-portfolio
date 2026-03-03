# Decisions, Tradeoffs, and Constraints

This document explains the product + system decisions behind the inventory/workflow design, including the constraints that shaped the solution (time, staffing, and operational realities).

---

## 1) Core constraints (what shaped the design)

### 1.1 Resource allocation (people + time)
The nonprofit context required assuming limited engineering capacity and limited operational bandwidth:
- **Small team / part-time availability:** designs needed to be implementable without a large dedicated engineering team.
- **Low tolerance for training overhead:** warehouse staff and volunteers needed to adopt the workflow quickly.
- **High impact per feature:** prioritized changes that removed the most manual work / reduced mistakes, rather than “nice-to-have” polish.
- **Maintenance matters more than novelty:** designs aim to prevent long-term complexity and reduce future bug surface area.

### 1.2 Operational reality
- Inventory updates happen under time pressure (receiving, packing, distribution).
- Volunteer turnover is high; workflows must be **self-explanatory**.
- The system must support **auditability** (who changed what, when) to resolve mismatches and disputes.

### 1.3 Data constraints
- Data is often inconsistent when coming from manual processes (spreadsheets, handwritten logs, partial item metadata).
- The design assumes **imperfect inputs**, so validation and item states must be explicit.

---

## 2) Guiding principles (how decisions were made)

1. **Reduce high-frequency friction:** optimize the “everyday” actions (search, update quantity, view status).
2. **Make the system hard to misuse:** avoid flows where a volunteer can accidentally corrupt inventory state.
3. **Preserve an audit trail:** reconcile errors by showing history rather than guessing.
4. **Design for incremental rollout:** allow shipping value in phases without breaking operations.
5. **Favor clarity over cleverness:** the UI should explain the warehouse, not require learning a new mental model.

---

## 3) Key design decisions

### 3.1 Inventory status model (explicit states)
**Decision:** use explicit statuses (e.g., in_stock, low_stock, reserved, distributed, damaged, pending_review) instead of relying on quantity alone.

**Why:**
- Quantity alone doesn’t explain why an item can’t be allocated.
- Status drives operational decisions (e.g., “pending review” shouldn’t be distributed).

**Tradeoff:**
- Adds conceptual complexity (more states to manage), but reduces ambiguity and prevents operational mistakes.

---

### 3.2 Transaction / history-first thinking (auditability)
**Decision:** treat inventory changes as events (receipts/distributions/adjustments), not just overwriting current quantity.

**Why:**
- Resolves mismatch disputes quickly (you can see the chain of changes).
- Supports accountability and reconciliation.

**Tradeoff:**
- More implementation effort than simple CRUD, but much better for long-term correctness.

---

### 3.3 Role-based permissions (reduce risk)
**Decision:** differentiate permissions by role (Admin vs Staff vs Volunteer).

**Why:**
- Volunteers often need read-only access, not write access.
- Prevents accidental edits that lead to major reconciliation issues.

**Tradeoff:**
- Requires up-front design for authorization, but pays off by reducing human-error cost.

---

### 3.4 UI: “Table-first” for high-volume tasks
**Decision:** prioritize an inventory table/list view with fast search/filter + inline status clarity.

**Why:**
- Warehouse tasks are repetitive and high-volume.
- Table view minimizes clicks and supports scanning.

**Tradeoff:**
- Less visually “pretty” than card-based UI, but dramatically faster for daily operations.

---

### 3.5 Edit flows: reduce irreversible mistakes
**Decision:** edits happen through constrained fields + confirmations for high-risk actions (e.g., bulk updates, status changes).

**Why:**
- One wrong edit can impact dozens of downstream distributions.
- Confirmation patterns reduce accidental damage.

**Tradeoff:**
- Adds an extra step for some actions, but improves safety and trust.

---

## 4) Resource allocation plan (how I’d phase this)

If engineering resources are limited, the design is intended to ship in **phases**:

### Phase 1 — Highest ROI (minimum viable operational system)
- Inventory list + search + filters
- Item detail view
- Basic role permissions
- Manual adjustment with reason field

**Goal:** eliminate the most painful manual reconciliation + provide visibility.

### Phase 2 — Audit + workflow hardening
- Transaction/history view (receipts/distributions/adjustments)
- Better validation and state constraints
- Alerts for anomalies (negative qty, sudden drops)

**Goal:** reduce errors and make mismatches explainable.

### Phase 3 — Efficiency + scale
- Bulk actions (import/export, batch status updates)
- Performance optimizations (pagination, indexing, caching)
- Dashboards for category-level visibility

**Goal:** improve throughput and enable growth.

---

## 5) Risks and mitigations

### Risk: messy item data (duplicates, inconsistent naming)
**Mitigation:**
- Introduce category standards and controlled vocabularies
- Deduping process + “merge items” flow (admin-only)

### Risk: permission complexity causes confusion
**Mitigation:**
- Keep roles minimal
- Clear UI cues (read-only badges; disabled edit controls)

### Risk: staff adoption / training burden
**Mitigation:**
- Use familiar patterns (table, filters)
- Tooltips and quick-start onboarding page
- Keep the “happy path” simple

---

## 6) What I would improve with more time

- Stronger reconciliation tooling (flag suspicious adjustments)
- Better analytics (usage logs for operations optimization)
- A more formalized taxonomy for item categories and attributes
- Integration points (barcode scanning, supplier imports) depending on constraints
