# QA Test Strategy Note

## 1. Strategy Overview

This test plan follows the **MFQ framework** (Models → Functional → Quality).

The core principle: **test depth follows business risk, not feature count**.

Inventory consistency and order/payment state correctness receive the deepest testing. Lower-risk UI and basic validation receive lighter coverage.

> Detailed model definitions → `MFQ-Test-Plan.md` M1–M5
> Detailed functional cases → `MFQ-Test-Plan.md` F1–F10
> Detailed quality objectives → `MFQ-Test-Plan.md` Q1–Q5

---

## 2. Why Prioritize Inventory and State Consistency

Inventory is the highest-risk domain because:

- A single bug can cause **overselling** — direct financial loss.
- Inventory state spans **multiple systems** (Order, Inventory, WMS, SC2P) — any link can fail.
- **Concurrency** is unavoidable (flash sales, last-item purchase) — correctness depends on locking/contention strategy.
- **Message-based synchronization** introduces duplicate, lost, and out-of-order risks.

The key release risk is not whether an API returns 200, but whether the **final business state** (inventory + order + payment) remains correct after failures, retries, and concurrent operations.

---

## 3. Prioritization and Trade-offs

Given the limited test window, the trade-off is **depth of high-risk coverage** over **breadth of all features**.

### Testing priority (high → low):

1. **Inventory consistency** — reservation, deduction, restoration, idempotency, concurrency
2. **Order/payment state correctness** — state machine transitions, payment callback handling
3. **Idempotency and message safety** — duplicate messages, out-of-order messages, lost messages
4. **Concurrent purchase scenarios** — last-item, flash-sale, concurrent reservation
5. **External system failure and compensation** — WMS/SC2P timeout, retry, reconciliation
6. **Recovery and reconciliation** — stuck orders, inventory difference detection

### What receives less depth:

- Basic UI/function validation (P2)
- Simple parameter validation
- Non-critical boundary conditions

This is a deliberate choice: identifying one overselling bug before release is worth more than passing 50 low-risk UI cases.

---

## 4. Exception and Compensation Approach

For every distributed operation, we test not only the happy path but also the **recovery path**:

- Operation → Partial Success / Failure → Retry → Compensation → Reconciliation → Consistent State


Key requirements:
- Compensation must be **idempotent** (no duplicate inventory changes).
- Failed compensation must be **retryable** and **observable**.
- Unresolved inconsistencies must enter **reconciliation** with alerting.

---

## 5. Release Decision

**Block release** when:
- Any confirmed overselling
- Critical inventory inconsistency
- Unrecoverable order/payment state inconsistency
- Open P0/Blocker defects

**Accept lower-priority defects** only when:
- Business risk is documented
- Stakeholder explicitly approves