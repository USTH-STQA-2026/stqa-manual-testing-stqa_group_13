# Summary Report — STQA Group 13

| Field | Value |
|-------|-------|
| **Group** | Group 13 |
| **Assignment** | A1 — Manual Black-Box Testing |
| **System Under Test** | https://stqa.rbc.vn (Library Management System) |
| **Testing Type** | Manual Black-Box Testing |
| **Date** | 27/05/2026 |
| **Reference** | SRS v1.0, test-cases.md, test-execution.md, bug-reports.md |

---

## 1. Overview

This report summarizes the results of manual black-box testing performed on the Library Management System (LMS) at https://stqa.rbc.vn. Testing covered all 8 functional requirements defined in SRS v1.0 (REQ-01 through REQ-08) using Equivalence Partitioning (EP), Boundary Value Analysis (BVA), and Decision Table testing techniques.

A total of **30 test cases** were designed and executed. **4 defects** were identified, including one critical defect that directly violates a stated business rule.

---

## 2. Testing Scope

| Requirement | Description | TCs Designed | TCs Executed |
|-------------|-------------|:------------:|:------------:|
| REQ-01 | User Login | 5 | 5 |
| REQ-02 | View Book List | 3 | 3 |
| REQ-03 | Search & Filter Books | 4 | 4 |
| REQ-04 | Borrow Book | 6 | 6 |
| REQ-05 | Return Book | 2 | 2 |
| REQ-06 | Overdue Handling | 2 | 2 |
| REQ-07 | Member Management | 5 | 5 |
| REQ-08 | Borrow Record Lookup | 3 | 3 |
| **Total** | | **30** | **30** |

---

## 3. Test Results Summary

| Metric | Value |
|--------|-------|
| Total Test Cases | 30 |
| Passed | 26 |
| Failed | 4 |
| Pass Rate | **86.7%** |
| Requirements Covered | 8 / 8 (100%) |
| Defects Found | 4 |
| Critical Defects | 1 (DEFECT-01) |
| High Defects | 1 (DEFECT-02) |
| Medium Defects | 1 (DEFECT-03) |
| Low Defects | 1 (DEFECT-04) |

### Pass/Fail by Requirement

| REQ | Pass | Fail | Result |
|-----|:----:|:----:|--------|
| REQ-01 | 5 | 0 | ✅ All Pass |
| REQ-02 | 3 | 0 | ✅ All Pass |
| REQ-03 | 4 | 0 | ✅ All Pass |
| REQ-04 | 5 | 1 | ⚠️ DEFECT-01 (Critical) |
| REQ-05 | 1 | 1 | ⚠️ DEFECT-03 (Medium) |
| REQ-06 | 2 | 0 | ✅ All Pass |
| REQ-07 | 3 | 2 | ⚠️ DEFECT-02 (High) + DEFECT-04 (Low) |
| REQ-08 | 3 | 0 | ✅ All Pass |

---

## 4. Defects Found

| Defect ID | Title | Severity | REQ | Found in TC |
|-----------|-------|----------|-----|-------------|
| DEFECT-01 | System allows a member to borrow more than 3 books simultaneously | **Critical** | REQ-04 | TC-04-04 |
| DEFECT-02 | Email address without dot in domain (e.g., `user@nodot`) is accepted | **High** | REQ-07 | TC-07-02 |
| DEFECT-03 | No overdue warning shown when returning a late book | **Medium** | REQ-05 | TC-05-02 |
| DEFECT-04 | Duplicate email rejection uses wrong error message "Email không hợp lệ." | **Low** | REQ-07 | TC-07-03 |

---

## 5. Testing Techniques Applied

### Equivalence Partitioning (EP)

EP was applied to all 8 requirements. Each input field or system condition was divided into valid and invalid equivalence classes, with one representative test case per class, ensuring broad coverage without redundant repetition.

**Example — REQ-01 Login:** partitioned into (valid credentials), (non-existent email), (wrong password), (empty fields) → 5 test cases covering all major input classes without over-testing.

**Example — REQ-07 Member Management:** email input partitioned into (valid format), (missing dot in domain), (missing @ symbol), (already registered) → distinct invalid classes, each revealing different validation failures.

### Boundary Value Analysis (BVA)

BVA was applied to REQ-04 (Borrow Book) to test the 3-book borrow limit boundary. The test sequence used MEM006 (biet.hoang@email.com), starting from 0 active borrows:

| TC | Borrow Count | Book | Boundary Position | Result |
|----|:------------:|------|:-----------------:|--------|
| TC-04-01 | 0 → 1 | BOOK002 | Below limit | PASS |
| TC-04-02 | 1 → 2 | BOOK005 | Within range | PASS |
| TC-04-03 | 2 → 3 | BOOK008 | At upper limit | PASS |
| TC-04-04 | 3 → 4 | BOOK010 | **Over boundary** | **FAIL → DEFECT-01** |

BVA was also applied to REQ-07 email validation to test the boundary between acceptable and unacceptable email formats (e.g., `user@nodot` vs. `user@nodot.com`).

### Decision Table (REQ-04 — Borrow Book)

A decision table was constructed for the borrow-book feature with 3 conditions:

- **C1**: Book status is "Available"
- **C2**: Member's active borrow count is < 3
- **C3**: Member account status is "Active"

| Rule | C1 (Available) | C2 (< 3 borrows) | C3 (Active) | Expected Action |
|------|:--------------:|:----------------:|:-----------:|-----------------|
| DT-1 | T | T | T | ✅ Allow borrow |
| DT-2 | F | T | T | ❌ Reject — book not available |
| DT-3 | T | F | T | ❌ Reject — **borrow limit reached** |
| DT-4 | T | T | F | ❌ Reject — account suspended/expired |
| DT-5 | F | F | T | ❌ Reject — multiple conditions fail |
| DT-6 | F | T | F | ❌ Reject — multiple conditions fail |
| DT-7 | T | F | F | ❌ Reject — multiple conditions fail |
| DT-8 | F | F | F | ❌ Reject — all conditions fail |

TC-04-04 tests Rule DT-3. The system incorrectly handles it as DT-1 (allows the borrow), confirming that C2 is never evaluated. This is the root cause of **DEFECT-01**.

---

## 6. Quality Assessment

### System Strengths

- **Authentication** (REQ-01) is fully correct — empty fields, unregistered email, and wrong password are all handled with clear, accurate messages.
- **Role-based access control** (REQ-02, REQ-07, REQ-08) is properly enforced — members cannot access librarian-only screens or member management functions.
- **Search and filter** (REQ-03) works correctly and supports case-insensitive title, author, and genre search.
- **Overdue detection** (REQ-06) reliably flags records past their due date and updates statuses correctly.
- **Record isolation** (REQ-08) is correctly implemented — each member can only view their own borrow history.
- **UI-level enforcement** for suspended accounts (REQ-04) and already-borrowed books works correctly.

### System Weaknesses

- **Borrow limit enforcement** (REQ-04) is absent at the backend — the most critical business rule has no server-side validation.
- **Email domain validation** (REQ-07) is incomplete — the server only verifies the presence of `@`, not the full domain format (missing dot check).
- **Error message specificity** (REQ-07) is poor — duplicate email conflicts return the same message as format validation failures, reducing diagnostic clarity for librarians.
- **Overdue notification on return** (REQ-05) is missing — the return workflow does not alert librarians when a book is returned late.

---

## 7. Recommendations

### Priority 1 — Fix Immediately

**DEFECT-01: Enforce 3-Book Borrow Limit (Critical)**
The absence of a server-side borrow count check allows unlimited borrowing by any member. This directly contradicts REQ-04 and the SRS. Recommended fix: before creating a new borrow record, query the count of active (non-returned) records for the requesting member. If count ≥ 3, return an appropriate error — *"You have reached the maximum borrow limit of 3 books."*

**DEFECT-02: Enforce Full Email Format on Server-Side (High)**
The current regex only checks for `@`. A proper validation pattern (requiring at least one dot in the domain segment) must be applied on the backend. Invalid addresses stored in the database will silently break notification delivery and account recovery. Recommended fix: apply a stricter regex such as `^[^@\s]+@[^@\s]+\.[^@\s]+$` at the API layer.

### Priority 2 — Fix in Next Sprint

**DEFECT-03: Add Overdue Warning on Book Return (Medium)**
When the return handler processes a record with status "Quá hạn", it should include an additional notification line (e.g., *"Note: This book was overdue."*) alongside the success message. This informs librarians at the point of action without requiring separate manual record checking.

### Priority 3 — Fix When Convenient

**DEFECT-04: Clarify Duplicate Email Error Message (Low)**
Separate the unique-constraint violation from the format validation error at the API layer and return a distinct, accurate message: *"This email address is already registered."* This is a UX improvement with no functional impact.

---

## 8. Conclusion

The Library Management System is functionally sound in authentication, role separation, search, overdue detection, and record isolation. However, the **complete absence of borrow limit enforcement** (DEFECT-01) is a critical gap that violates a core business rule and makes library inventory policy unenforceable. Combined with the email domain validation gap (DEFECT-02), the system carries data integrity risks that should be resolved before any production deployment.

**Recommendation: do not release the system until DEFECT-01 and DEFECT-02 are fixed.** DEFECT-03 and DEFECT-04 may be addressed in subsequent patches.

| Quality Dimension | Assessment |
|---|---|
| Functional Correctness | ⚠️ 86.7% (26/30 TCs pass) |
| Business Rule Compliance | ❌ Fails — critical borrow limit not enforced |
| Data Integrity | ❌ At risk — invalid email addresses accepted and stored |
| Error Message Clarity | ⚠️ Needs improvement — duplicate vs. format errors conflated |
| Release Readiness | ❌ Not recommended without DEFECT-01 and DEFECT-02 fixes |
