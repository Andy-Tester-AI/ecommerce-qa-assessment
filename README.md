 
# E-commerce QA Test Plan Assessment

## 1. Overview

This repository contains the QA test plan and test cases for the E-commerce System assessment.

The test strategy follows the **MFQ framework**:

- **M — Models**
- **F — Functional**
- **Q — Quality**

The assessment focuses on the following business areas:

- Product / SPU / SKU
- Inventory
- Cart
- Order lifecycle
- Payment
- Shipment
- Partial / Split Shipment
- Return / Refund
- External synchronization with WMS / SC2P
- Exception handling and compensation
- Performance, reliability, observability, security, and recoverability

The primary testing objective is to identify high-risk business failures, especially:

- Inventory overselling
- Duplicate inventory deduction
- Incorrect order/payment states
- Inventory synchronization inconsistency
- Duplicate or out-of-order messages
- Concurrent purchase of the last item
- External system failures
- Unrecoverable data inconsistencies

---

## 2. Testing Approach

The test plan follows the **MFQ framework**: **M** (Models) to understand business entities and state transitions, **F** (Functional) to validate core flows, boundaries, inventory, API and compensation, and **Q** (Quality) to evaluate performance, reliability, observability, security and recoverability. Testing depth follows business risk — inventory consistency and order/payment correctness receive the highest priority.

> For detailed model definitions, functional test areas and quality objectives, see [MFQ-Test-Plan.md](MFQ-Test-Plan.md).

## 3. Risk-Based Prioritization

Testing priority is based on business risk rather than equal coverage across all functions.

### P0 — Critical

Examples:

- Inventory overselling
- Duplicate inventory deduction
- Incorrect payment/order state
- Critical inventory inconsistency
- Concurrent purchase of the last item
- Duplicate or out-of-order business messages
- Unrecoverable external synchronization failure
- Financial or data loss

### P1 — High

Examples:

- Partial / split shipment
- WMS / SC2P synchronization failure
- Timeout and retry
- Return and inventory restoration
- Compensation failure
- Important business rule failures

### P2 — Normal

Examples:

- Basic parameter validation
- Low-risk UI behavior
- Non-critical boundary scenarios

---

## 4. Inventory Consistency

Inventory is treated as a critical business domain.

The main inventory flow is:

```text
Available
    ↓
Reserved
    ↓
Deducted
```

Failure, cancellation, or return may require restoration:

```text
Reserved → Restored → Available

Deducted → Restored → Available
```

The test strategy focuses on:

* Reservation success / failure
* Deduction success / failure
* Cancellation and restoration
* Return and restoration
* Duplicate messages
* Out-of-order messages
* Concurrent requests
* Flash-sale scenarios
* Zero inventory
* Lost responses
* Timeout and retry
* External inventory differences
* Reconciliation

---

## 5. Release Decision

The release decision is based on functional correctness and overall business risk.

Release should be blocked when critical issues remain, especially:

* Confirmed overselling
* Incorrect inventory deduction
* Critical inventory inconsistency
* Payment/order state inconsistency
* Unrecoverable external synchronization failure
* P0 / Blocker defects

Lower-risk defects may be accepted only after risk assessment, documentation, and explicit approval.

---

## 6. Repository Structure

```text
ecommerce-qa-assessment/
│
├── README.md
├── MFQ-Test-Plan.md
├── Strategy-Note.md
│
└── test-cases/
    └── ecommerce-test-cases.md
```

### File Description

| File                                 | Description                                                    |
| ------------------------------------ | -------------------------------------------------------------- |
| `README.md`                          | Project overview and testing approach                          |
| `MFQ-Test-Plan.md`                   | Main MFQ-based test plan                                       |
| `Strategy-Note.md`                   | Testing strategy, priorities, trade-offs, and release decision |
| `test-cases/ecommerce-test-cases.md` | Structured test cases with priority and expected results       |

---

## 7. Key Testing Principles

### 7.1 State-Based Testing

Validate not only whether an API succeeds, but also whether the resulting business state is correct.

For example:

```text
Operation
    ↓
Expected State Transition
    ↓
Inventory / Order / Payment Consistency
```

### 7.2 Idempotency

Repeated requests or messages should not cause repeated business effects.

Examples:

* Duplicate payment callback
* Duplicate inventory deduction
* Duplicate shipment notification
* Duplicate restoration

### 7.3 Concurrency

Concurrency testing focuses on high-risk inventory scenarios, especially:

* Last-item purchase
* Flash-sale traffic
* Concurrent reservation
* Concurrent payment processing

### 7.4 Failure Recovery

When a distributed operation partially fails, the system should be able to:

```text
Detect
  ↓
Retry
  ↓
Compensate
  ↓
Reconcile
  ↓
Restore Consistency
```
