# Test Cases — Loan Request

**Journey:** Request a loan
**Priority:** P2
**URL:** `/parabank/requestloan.htm`

The approval outcome depends on three inputs together — loan amount, down payment and
the balance of the funding account — which makes this a decision table rather than a set
of independent field checks. Testing each field in isolation would miss the combinations,
which is exactly where approval logic tends to be wrong.

## Decision table

Let **A** = loan amount requested, **D** = down payment, **B** = balance of the funding
account. The rules below are the behaviour to confirm; record the actual outcome and note
any row where the application disagrees.

| Row | Condition | Expected outcome |
|---|---|---|
| 1 | D ≤ B, A within lending limit | Approved. Down payment debited from the funding account, loan account created |
| 2 | D > B | Denied. Insufficient funds for the down payment. **No debit, no loan account** |
| 3 | D = B exactly | Approved. Funding account left at 0.00 |
| 4 | D = 0 | Behaviour recorded — is a zero down payment permitted? |
| 5 | A = 0 | Denied or rejected at validation |
| 6 | A very large relative to D | Denied. Record the threshold if one is discoverable |
| 7 | D negative | Rejected at validation. Must not credit the funding account |
| 8 | A negative | Rejected at validation |

## Cases

| ID | Row | Technique | Steps | Expected result |
|---|---|---|---|---|
| TC-LR-001 | 1 | Decision table | Balance 1000, request 5000 with 200 down | Approved. Balance becomes 800. New loan account visible in Accounts Overview |
| TC-LR-002 | 2 | Decision table | Balance 100, request 5000 with 500 down | Denied. **Balance still exactly 100.** No loan account created |
| TC-LR-003 | 3 | BVA | Balance 200, request 5000 with 200 down | Approved. Balance 0.00 |
| TC-LR-004 | 2 | BVA | Balance 200, request 5000 with 200.01 down | Denied, balance unchanged |
| TC-LR-005 | 4 | BVA | Down payment 0 | Outcome recorded |
| TC-LR-006 | 5 | BVA | Loan amount 0 | Rejected |
| TC-LR-007 | 7 | NEG | Down payment -100 | Rejected. Confirm the balance did not increase |
| TC-LR-008 | 8 | NEG | Loan amount -5000 | Rejected |
| TC-LR-009 | — | NEG | Submit with all fields blank | Rejected, fields named individually |
| TC-LR-010 | — | NEG | Non-numeric values in both amount fields | Rejected cleanly, no server error |
| TC-LR-011 | 1 | — | After an approved loan, open the new loan account's activity | Opening entry present and consistent with the approved amount |
| TC-LR-012 | 2 | EG | After a denial, refresh the result page | No loan created retrospectively, no second debit |

## Note

TC-LR-002 and TC-LR-004 are the important pair. A denial that still debits the down
payment takes the customer's money and gives them nothing, and it produces a denial
screen that looks entirely normal. Asserting the balance after a *failed* request is the
only way to see it.
