Please refer to the following test cases, or you can directly view the XMind documentation; the test case content is the same.

# 1. Priority Definition

| Priority | Definition |
| -------- | ---------- |
| P0 | Critical business risk. Failure may cause overselling, financial loss, incorrect order state, data inconsistency or major business interruption. |
| P1 | Important business function. Failure significantly impacts users or business operations but does not necessarily cause critical data/financial loss. |
| P2 | Normal functional or boundary scenario with relatively lower business risk. |

---

# 2. Test Case Tree

## 2.1 Product / SKU

### TC-PRODUCT

| ID | Test Scenario | Priority | Expected Result |
| ---- | ---- | -------- | ---- |
| P001 | View product | P1 | Product information is displayed correctly. |
| P002 | Select valid SKU | P1 | Correct SKU is selected and corresponding information is displayed. |
| P003 | Select unavailable SKU | P1 | Unavailable SKU cannot be purchased. |
| P004 | SKU with zero inventory | P0 | SKU cannot be successfully purchased. |
| P005 | Invalid SKU | P1 | Invalid SKU is rejected and no order/inventory change occurs. |
| P006 | Quantity exceeds available inventory | P0 | Purchase is rejected and inventory remains consistent. |

---

## 2.2 Cart

### TC-CART

| ID | Test Scenario | Priority | Expected Result |
| ---- | ---- | -------- | ---- |
| C001 | Add SKU to cart | P1 | SKU is successfully added to cart. |
| C002 | Update cart quantity | P1 | Cart quantity and amount are updated correctly. |
| C003 | Remove SKU | P2 | SKU is removed successfully. |
| C004 | Add same SKU repeatedly | P1 | Cart quantity follows the defined business rule without incorrect duplication. |
| C005 | Add zero quantity | P2 | Invalid quantity is rejected. |
| C006 | Add negative quantity | P2 | Invalid quantity is rejected. |
| C007 | Cart SKU becomes unavailable | P1 | User cannot successfully place an invalid order. |
| C008 | Cart inventory changes before checkout | P0 | Final checkout validates current inventory and prevents overselling. |

---

## 2.3 Order

### TC-ORDER

| ID | Test Scenario | Priority | Expected Result |
| ---- | ---- | -------- | ---- |
| O001 | Create normal order | P0 | Order is created successfully and required inventory is reserved. |
| O002 | Create order with insufficient inventory | P0 | Order is rejected and inventory remains correct. |
| O003 | Create order with zero inventory | P0 | Order cannot be successfully created. |
| O004 | Invalid SKU in order | P1 | Order creation is rejected. |
| O005 | Invalid quantity | P1 | Order creation is rejected. |
| O006 | Submit order repeatedly | P0 | Duplicate submission does not create duplicate orders or inventory reservations. |
| O007 | Inventory reservation succeeds but order creation fails | P0 | Reserved inventory is released through compensation. |
| O008 | Order cancellation before payment | P0 | Order is cancelled and reserved inventory is restored. |
| O009 | Cancel already completed order | P1 | Invalid state transition is rejected. |

---

## 2.4 Payment

### TC-PAYMENT

| ID | Test Scenario | Priority | Expected Result |
| ---- | ---- | -------- | ---- |
| PAY001 | Successful payment | P0 | Payment succeeds, order becomes Paid, inventory transitions correctly. |
| PAY002 | Payment failure | P0 | Order does not incorrectly become Paid and inventory follows the defined release/compensation flow. |
| PAY003 | Payment timeout | P0 | Timeout is handled safely and does not create inconsistent order/inventory state. |
| PAY004 | Duplicate payment callback | P0 | Duplicate callback does not cause duplicate payment or inventory deduction. |
| PAY005 | Payment succeeds but order update fails | P0 | System retries/reconciles and eventually restores a consistent state. |
| PAY006 | Payment callback arrives late | P1 | Late callback is processed according to valid order state rules. |
| PAY007 | Invalid payment callback | P1 | Invalid callback is rejected and does not modify business state. |

---

## 2.5 Shipment

### TC-SHIPMENT

| ID | Test Scenario | Priority | Expected Result |
| ---- | ---- | -------- | ---- |
| S001 | Normal shipment | P0 | Shipment succeeds and order enters the correct state. |
| S002 | Shipment failure | P1 | Failure is recorded and order remains in a valid state. |
| S003 | WMS request timeout | P0 | Timeout is handled through retry/failure handling without duplicate shipment. |
| S004 | Duplicate shipment request | P0 | Duplicate request does not create duplicate shipment. |
| S005 | Duplicate shipment callback | P0 | Duplicate callback does not cause invalid state transition. |
| S006 | Shipment callback out of order | P1 | Invalid/out-of-order event does not corrupt order state. |

---

## 2.6 Partial / Split Shipment

### TC-SPLIT-SHIPMENT

| ID | Test Scenario | Priority | Expected Result |
| ---- | ---- | -------- | ---- |
| SS001 | Multi-SKU order split into multiple shipments | P0 | Each shipment contains the correct SKU and quantity. |
| SS002 | Only part of order is shipped | P0 | Order is not incorrectly marked as fully completed. |
| SS003 | Multiple shipments complete sequentially | P1 | Final order state is correct after all shipments are completed. |
| SS004 | Shipment quantity exceeds ordered quantity | P0 | Excess shipment is rejected. |
| SS005 | Duplicate partial shipment notification | P0 | Duplicate notification does not duplicate shipment quantity. |
| SS006 | One shipment fails while another succeeds | P0 | Successful and failed portions maintain correct states and can be recovered. |

---

## 2.7 Signed / Completed

### TC-SIGNED

| ID | Test Scenario | Priority | Expected Result |
| ---- | ---- | -------- | ---- |
| SG001 | Normal signing | P1 | Order enters Signed state. |
| SG002 | Signed after shipment | P1 | Valid state transition succeeds. |
| SG003 | Duplicate signed notification | P1 | Duplicate notification is idempotent. |
| SG004 | Sign before shipment | P1 | Invalid state transition is rejected or safely handled. |
| SG005 | Signed order becomes completed | P1 | Order reaches the correct final state. |

---

## 2.8 Return / Refund

### TC-RETURN

| ID | Test Scenario | Priority | Expected Result |
| ---- | ---- | -------- | ---- |
| R001 | Normal return | P1 | Return request is processed successfully. |
| R002 | Refund after valid return | P0 | Refund is processed correctly. |
| R003 | Return restores inventory | P0 | Inventory is restored correctly when applicable. |
| R004 | Duplicate return request | P1 | Duplicate request does not create duplicate return. |
| R005 | Duplicate refund | P0 | Duplicate refund is prevented. |
| R006 | Return quantity exceeds purchased quantity | P1 | Request is rejected. |
| R007 | Invalid order state for return | P1 | Invalid return is rejected. |
| R008 | Inventory restoration fails | P0 | Failure enters retry/compensation/reconciliation flow. |

---

## 2.9 Inventory Synchronization

> This is the highest-risk functional area.

### TC-INVENTORY-SYNC

| ID | Test Scenario | Priority | Expected Result |
| ---- | ---- | -------- | ---- |
| I001 | Inventory reservation succeeds | P0 | Available decreases and Reserved increases correctly. |
| I002 | Inventory reservation fails | P0 | No incorrect inventory deduction occurs. |
| I003 | Inventory deduction succeeds | P0 | Reserved decreases and Sold increases correctly. |
| I004 | Inventory deduction fails | P0 | System handles failure without losing inventory. |
| I005 | Cancel order restores inventory | P0 | Reserved inventory returns to Available correctly. |
| I006 | Return restores inventory | P0 | Applicable inventory is restored correctly. |
| I007 | Duplicate reservation message | P0 | Duplicate message does not reserve inventory twice. |
| I008 | Duplicate deduction message | P0 | Duplicate message does not deduct inventory twice. |
| I009 | Duplicate restoration message | P0 | Inventory is restored only once. |
| I010 | Out-of-order inventory messages | P0 | Invalid message ordering does not corrupt inventory state. |
| I011 | Inventory = 0 | P0 | Purchase/reservation is rejected. |
| I012 | Concurrent purchase of last item | P0 | Only valid requests succeed; overselling does not occur. |
| I013 | Concurrent reservation | P0 | Total reserved quantity does not exceed available inventory. |
| I014 | Reservation succeeds but order fails | P0 | Reserved inventory is compensated/released. |
| I015 | Deduction succeeds but response is lost | P0 | Retry does not cause duplicate deduction. |
| I016 | Inventory synchronization timeout | P1 | Retry/reconciliation eventually restores consistency. |
| I017 | WMS inventory differs from local inventory | P0 | Difference is detected and enters reconciliation flow. |
| FS001 | Flash sale inventory preload | P0 | Flash sale inventory is correctly allocated; available inventory reflects the allocation. |
| FS002 | Flash sale start inventory display | P0 | Correct flash sale inventory count is displayed; no overselling before start time. |
| FS003 | Concurrent purchase of last flash sale item | P0 | Only one request succeeds; no overselling occurs; final inventory = 0. |
| FS004 | Purchase exceeds per-user flash sale limit | P0 | Second request is rejected; inventory unchanged. |
| FS005 | Flash sale inventory deduction consistency | P0 | Order state, inventory state and flash sale remaining count are all consistent. |
| FS006 | Flash sale ends — unsold inventory restored | P0 | Unsold inventory is released back to general available inventory. |
| FS007 | Flash sale order payment timeout | P0 | Order is auto-cancelled; reserved inventory is released back to flash sale pool. |
| FS008 | Duplicate flash sale order under high concurrency | P0 | Only one order is created; inventory deducted only once. |
| FS009 | Flash sale page load under high traffic | P1 | Page loads within acceptable latency; inventory count is accurate. |
| FS010 | Flash sale countdown accuracy across time zones | P1 | Countdown is accurate for all time zones; sale starts/ends at the correct absolute time. |


---

## 2.10 API Validation

### TC-API

| ID | Test Scenario | Priority | Expected Result |
| ---- | ---- | -------- | ---- |
| API001 | Missing required parameter | P1 | Request is rejected with structured error. |
| API002 | Invalid parameter type | P1 | Request is rejected. |
| API003 | Invalid parameter range | P1 | Request is rejected. |
| API004 | Invalid SKU | P1 | Request is rejected. |
| API005 | Invalid quantity | P1 | Request is rejected. |
| API006 | Unauthorized request | P0 | Request is rejected. |
| API007 | User accesses another user's order | P0 | Access is rejected. |
| API008 | Invalid order state operation | P0 | Operation is rejected without changing business data. |
| API009 | Invalid request does not modify inventory | P0 | Inventory remains unchanged. |

---

## 2.11 Exception / Compensation

### TC-COMPENSATION

| ID | Test Scenario | Priority | Expected Result |
| ---- | ---- | -------- | ---- |
| EC001 | Inventory reservation succeeds but order creation fails | P0 | Reserved inventory is released. |
| EC002 | Payment succeeds but order update fails | P0 | Retry/reconciliation restores consistency. |
| EC003 | Inventory deduction fails after payment | P0 | System enters retry/compensation flow. |
| EC004 | WMS request timeout | P0 | Retry is performed according to policy. |
| EC005 | External system unavailable | P1 | Failure is handled without corrupting local state. |
| EC006 | Compensation request repeated | P0 | Compensation is idempotent. |
| EC007 | Compensation fails repeatedly | P0 | Failure is observable and enters reconciliation/manual repair flow. |
| EC008 | Message lost | P0 | Missing business event can be detected and recovered. |
| EC009 | Message duplicated | P0 | Duplicate message does not create duplicate business effects. |
| EC010 | Message out of order | P0 | Invalid state transition is prevented and recoverable. |