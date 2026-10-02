# ShopEasy — Sprint 9 QA Report

**Project:** ShopEasy E-Commerce Application  
**Sprint:** 9  
**Sprint Duration:** 10 days  
**Team Size:** 2 QA Testers  
**Report Date:** Sprint 9 Closure  

---

## Table of Contents

1. [Part 1 — Test Metrics Dashboard](#part-1--test-metrics-dashboard)
2. [Part 2 — Risk Assessment: Guest Checkout (Sprint 10)](#part-2--risk-assessment-guest-checkout-sprint-10)
3. [Part 3 — SQL Validation Queries](#part-3--sql-validation-queries)
4. [Part 4 — Defect Analysis](#part-4--defect-analysis)

---

## Part 1 — Test Metrics Dashboard

### 1.1 Raw Data

| Data Point               | Value |
|--------------------------|-------|
| Total Tests Planned      | 180   |
| Tests Executed           | 172   |
| Tests Passed             | 155   |
| Tests Failed             | 17    |
| Tests Blocked            | 8     |
| Defects Found            | 22    |
| — Critical               | 4     |
| — Major                  | 7     |
| — Minor                  | 8     |
| — Trivial                | 3     |
| Production Bugs (prev.)  | 3     |
| Rejected Bugs            | 2     |
| Sprint Duration          | 10 days |
| Team Size                | 2 testers |

---

### 1.2 Calculated Metrics

#### Execution Rate
Measures how much of the planned test scope was actually run.

```
Execution Rate = (Tests Executed / Total Tests Planned) × 100
               = (172 / 180) × 100
               = 95.56%
```
> 8 tests remain unexecuted (all blocked). Target is typically ≥ 95% — we are right at the threshold.

---

#### Pass Rate
Proportion of executed tests that passed.

```
Pass Rate = (Tests Passed / Tests Executed) × 100
          = (155 / 172) × 100
          = 90.12%
```
> A pass rate above 90% is generally acceptable for a release candidate.

---

#### Failure Rate
```
Failure Rate = (Tests Failed / Tests Executed) × 100
             = (17 / 172) × 100
             = 9.88%
```

---

#### Defect Density
Defects found per test executed — indicates code quality.

```
Defect Density = Total Defects / Tests Executed
               = 22 / 172
               = 0.128 defects per test
```
> Higher density suggests areas needing rework. Cart and Checkout modules are the primary contributors (see Part 4).

---

#### Defect Removal Efficiency (DRE)
Measures how effective the team was at catching defects before production.

```
DRE = (Defects Found in Testing / (Defects Found in Testing + Production Bugs)) × 100
    = (22 / (22 + 3)) × 100
    = (22 / 25) × 100
    = 88.00%
```
> Industry benchmark is ≥ 85%. At 88%, the team is performing above the baseline, but 3 production escapes indicate room for improvement — particularly around checkout flows.

---

#### Defect Leakage Rate
Proportion of total defects that escaped to production.

```
Leakage Rate = (Production Bugs / (Defects Found in Testing + Production Bugs)) × 100
             = (3 / 25) × 100
             = 12.00%
```
> 12% leakage is above the ideal target of ≤ 10%. This warrants attention in Sprint 10.

---

#### Bug Rejection Rate
Proportion of raised defects that were rejected (i.e., not valid bugs).

```
Bug Rejection Rate = (Rejected Bugs / (Total Defects Raised + Rejected Bugs)) × 100
                   = (2 / (22 + 2)) × 100
                   = (2 / 24) × 100
                   = 8.33%
```
> A rejection rate under 10% is healthy. QA defect reporting quality is good.

---

#### Productivity
Average test cases executed per tester per day.

```
Productivity = Tests Executed / (Team Size × Sprint Duration)
             = 172 / (2 × 10)
             = 8.6 tests per tester per day
```

---

### 1.3 Metrics Summary Table

| Metric                          | Value   | Status         |
|---------------------------------|---------|----------------|
| Execution Rate                  | 95.56%  | ⚠️ At threshold (target ≥ 95%) |
| Pass Rate                       | 90.12%  | ✅ Acceptable  |
| Failure Rate                    | 9.88%   | ⚠️ Monitor     |
| Defect Density                  | 0.128   | ⚠️ Moderate    |
| Defect Removal Efficiency (DRE) | 88.00%  | ✅ Above benchmark (≥ 85%) |
| Defect Leakage Rate             | 12.00%  | ❌ Above target (≤ 10%) |
| Bug Rejection Rate              | 8.33%   | ✅ Healthy (< 10%) |
| Productivity                    | 8.6 tests/tester/day | ✅ Steady |

---

### 1.4 Severity Distribution

| Severity  | Count | % of Total |
|-----------|-------|------------|
| Critical  | 4     | 18.2%      |
| Major     | 7     | 31.8%      |
| Minor     | 8     | 36.4%      |
| Trivial   | 3     | 13.6%      |
| **Total** | **22** | **100%**  |

> 50% of defects are Critical or Major. This is the segment that must be resolved before release.

---

### 1.5 Release Recommendation

Sprint 9 delivered a **conditionally release-ready** build. The team executed 95.56% of planned tests and achieved a pass rate of 90.12%, with a Defect Removal Efficiency of 88% — above the industry baseline. However, three production escapes from the previous release have pushed the defect leakage rate to 12%, which exceeds the acceptable threshold of 10%, and all 4 Critical defects plus the 7 Major defects (totaling 50% of the defect pool) must be resolved and re-verified before the build is cleared for production. The 8 blocked tests — concentrated in the checkout and cart modules — represent an unacceptable coverage gap given that these modules carry the highest defect density. **Recommendation: do not release until all Critical/Major defects are closed, blocked tests are unblocked and re-executed, and a targeted regression cycle is completed on the Checkout and Cart modules.**

---

## Part 2 — Risk Assessment: Guest Checkout (Sprint 10)

**Feature:** Guest Checkout — allows users to complete purchases without creating an account.

| # | Risk | Likelihood | Impact | Risk Level | Mitigation |
|---|------|------------|--------|------------|------------|
| R1 | **Order-to-user linkage failure** — guest orders may not be persisted correctly without a `user_id`, causing orphaned records or broken order history | Medium | High | 🔴 High | Define a nullable `user_id` policy and a guest session token strategy early in design; validate DB integrity after each order placement test |
| R2 | **Payment data exposure** — guest checkout may bypass authentication guards protecting payment processing, increasing PCI-DSS surface area | Low | Critical | 🔴 High | Security review of the payment flow before sprint starts; add automated tests that verify no auth bypass is possible; involve security team in design sign-off |
| R3 | **Stock decrement race condition** — without a logged-in session, concurrent guest checkouts may not correctly serialize stock updates, leading to overselling | Medium | High | 🔴 High | Implement and test database-level locking or optimistic concurrency on the `stock` column; include load/concurrency tests in the sprint test plan |
| R4 | **Email confirmation delivery failure** — guest orders rely solely on email for order confirmation; if email is misconfigured or enters a spam folder, users have no recovery path | Medium | Medium | 🟡 Medium | Test email delivery across major providers (Gmail, Outlook, Yahoo); implement a guest order lookup page (order ID + email) as a fallback |
| R5 | **Analytics and reporting gaps** — revenue and "Active Users" dashboards are built on `user_id` joins; guest orders may be silently excluded from Total Revenue and Top Products calculations | High | Medium | 🟡 Medium | Audit all reporting queries before sprint close to ensure they handle NULL `user_id`; add a dedicated QA check for dashboard figures against raw DB totals (see Part 3, queries d–f) |

---

## Part 3 — SQL Validation Queries

All queries use Standard SQL (ANSI). Replace literal values (e.g., `'testuser@example.com'`) with the actual test data used during execution.

---

### a) After Registration — Verify New User Exists with Correct Role and Status

```sql
-- Replace the email value with the address used during registration
SELECT
    user_id,
    username,
    email,
    role,
    status,
    created_date
FROM users
WHERE email = 'testuser@example.com';

-- Expected: exactly 1 row returned
-- Expected role   : 'customer'  (or your application's default role)
-- Expected status : 'active'
-- Expected created_date : today's date
```

---

### b) After Placing an Order — Verify Order Total Equals Price × Quantity

```sql
-- Replace :order_id with the ID of the order just placed
SELECT
    o.order_id,
    o.quantity,
    o.total          AS stored_total,
    p.price          AS unit_price,
    (p.price * o.quantity)  AS expected_total,
    CASE
        WHEN o.total = (p.price * o.quantity) THEN 'PASS'
        ELSE 'FAIL'
    END AS validation_result
FROM orders o
JOIN products p ON o.product_id = p.product_id
WHERE o.order_id = :order_id;

-- Expected: validation_result = 'PASS'
```

---

### c) After Order — Verify Product Stock Decreased by Order Quantity

```sql
-- Record the stock BEFORE placing the order, then run this after.
-- Replace :product_id and :order_quantity with actual test values.
SELECT
    p.product_id,
    p.name,
    p.stock                            AS current_stock,
    :stock_before_order                AS stock_before,
    (:stock_before_order - :order_quantity) AS expected_stock,
    CASE
        WHEN p.stock = (:stock_before_order - :order_quantity) THEN 'PASS'
        ELSE 'FAIL'
    END AS validation_result
FROM products p
WHERE p.product_id = :product_id;

-- Expected: validation_result = 'PASS'
```

---

### d) Dashboard — Verify Total Revenue (Completed Orders Only)

```sql
-- This gives the ground-truth value to compare against the dashboard display
SELECT
    SUM(total) AS total_revenue
FROM orders
WHERE status = 'completed';

-- Compare this figure to what the dashboard shows.
-- Expected: dashboard value = query result (within any currency rounding rules)
```

---

### e) Dashboard — Verify Active Users Count

```sql
SELECT
    COUNT(*) AS active_users_count
FROM users
WHERE status = 'active';

-- Compare this figure to the "Active Users" count shown on the dashboard.
-- Expected: dashboard value = query result
```

---

### f) "Top 5 Products" Report — Verify Against Database

```sql
SELECT
    p.product_id,
    p.name,
    SUM(o.quantity) AS total_quantity_sold
FROM orders o
JOIN products p ON o.product_id = p.product_id
WHERE o.status = 'completed'   -- include only fulfilled orders
GROUP BY p.product_id, p.name
ORDER BY total_quantity_sold DESC
FETCH FIRST 5 ROWS ONLY;       -- ANSI SQL:2008; use LIMIT 5 if your DB supports it

-- Compare this ranked list to the "Top 5 Products" report in the application.
-- Expected: same product names in the same order with matching quantities
```

---

### g) Data Integrity Check — Orders Where Total ≠ Price × Quantity

```sql
-- Returns all orders with a calculation mismatch (should return 0 rows)
SELECT
    o.order_id,
    o.user_id,
    o.product_id,
    o.quantity,
    o.total                        AS stored_total,
    p.price                        AS unit_price,
    (p.price * o.quantity)         AS expected_total,
    (o.total - (p.price * o.quantity)) AS discrepancy
FROM orders o
JOIN products p ON o.product_id = p.product_id
WHERE o.total <> (p.price * o.quantity);

-- Expected: 0 rows returned
-- Any rows returned = data integrity defect; log with order_id and discrepancy amount
```

---

### h) Logic Bug Check — Users with Orders but Inactive Status

```sql
-- Returns users who have placed at least one order but are flagged as inactive
-- These accounts should either be active or their orders should not be processable
SELECT DISTINCT
    u.user_id,
    u.username,
    u.email,
    u.status,
    COUNT(o.order_id) AS order_count
FROM users u
JOIN orders o ON u.user_id = o.user_id
WHERE u.status = 'inactive'
GROUP BY u.user_id, u.username, u.email, u.status
ORDER BY order_count DESC;

-- Expected: 0 rows returned
-- Any rows returned = logic defect; an inactive user should not be able to place orders
-- Log each user_id found as a separate defect instance
```

---

## Part 4 — Defect Analysis

### 4.1 Defect Distribution by Module

| Module   | Defect Count | % of Total | Severity Breakdown (estimated) |
|----------|-------------|------------|-------------------------------|
| Cart     | 8           | 36.4%      | 1 Critical, 3 Major, 3 Minor, 1 Trivial |
| Checkout | 7           | 31.8%      | 2 Critical, 2 Major, 2 Minor, 1 Trivial |
| Search   | 4           | 18.2%      | 1 Critical, 1 Major, 1 Minor, 1 Trivial |
| Login    | 3           | 13.6%      | 0 Critical, 1 Major, 2 Minor, 0 Trivial |
| **Total**| **22**      | **100%**   | 4 Critical, 7 Major, 8 Minor, 3 Trivial |

> Note: severity-per-module breakdown above is estimated proportionally from the totals provided. Update with actual defect records from your tracking tool.

---

### 4.2 Module Priority for Increased Testing Focus

**Primary focus: Cart (8 defects, 36.4%)**  
The Cart module carries the highest absolute defect count and represents the most-used path in the purchase funnel. High defect density here directly amplifies the risk of checkout failures and revenue loss.

**Secondary focus: Checkout (7 defects, 31.8%, including 2 Critical)**  
Checkout has the highest concentration of Critical-severity defects relative to its size. Two critical defects in a payment-adjacent flow represent both a revenue risk and a potential data integrity issue.

Together, Cart and Checkout account for **68.2% of all defects** and are the pipeline to revenue — they must be the primary regression targets in Sprint 10, especially given the Guest Checkout feature incoming.

---

### 4.3 Root Cause Category Analysis

| Root Cause Category           | Description                                                                 | Modules Affected       | Defect Count (est.) |
|-------------------------------|-----------------------------------------------------------------------------|------------------------|---------------------|
| **Business Logic Errors**     | Incorrect implementation of business rules (e.g., wrong price calculation, discount misapplication, stock not decrementing) | Cart, Checkout         | ~7                  |
| **UI / UX Validation Gaps**   | Missing or incorrect client-side input validation (e.g., negative quantities accepted, empty fields submitted) | Cart, Search           | ~5                  |
| **State Management Issues**   | Session or cart state not persisting correctly across pages or after login/logout | Cart, Login            | ~4                  |
| **Integration / API Errors**  | Frontend and backend data contracts mismatched (e.g., field name differences, type mismatches in order payload) | Checkout, Search       | ~4                  |
| **Edge Case Handling**        | Unhandled edge cases (e.g., out-of-stock checkout attempt, special characters in search, duplicate order submission) | Search, Checkout, Cart | ~2                  |

> Estimated distribution based on module context and severity mix. Confirm against actual defect records in your issue tracker (Jira/Azure DevOps).

---

### 4.4 Mini Defect Analysis Report

**Sprint 9 Defect Summary — ShopEasy**

The sprint produced 22 defects across 4 modules. The Cart and Checkout modules together contribute 68% of all defects, which is consistent with the high complexity and business-rule density of the purchase funnel. The dominant root cause category is **Business Logic Errors** — primarily related to price calculation, discount application, and stock management — which accounts for an estimated ~32% of defects. This pattern typically indicates insufficient unit test coverage at the service layer and inadequate developer-level testing of boundary conditions before handoff to QA.

The second-largest category, **UI/UX Validation Gaps**, suggests that client-side validation is being added inconsistently or after the fact. Introducing a validation checklist as part of the Definition of Done for any user-input form would reduce this category significantly.

Three defects escaped to production (leakage rate: 12%), all of which are likely attributable to the Integration and Edge Case categories — scenarios that are difficult to reproduce in a staging environment. Adding a dedicated integration test suite and expanding the checkout regression suite to include concurrency and boundary scenarios are the highest-leverage actions for Sprint 10.

**Top 3 recommended actions:**
1. Add service-layer unit tests for all pricing, discount, and stock-update logic before QA handoff
2. Define and enforce a client-side validation standard for all form components in Cart and Checkout
3. Expand the regression suite to cover the 3 production-escape scenarios as permanent regression tests

---

*Report prepared by: QA Team — ShopEasy Sprint 9*  
*Classification: Internal*
