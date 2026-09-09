# Test Cases — Transfer Funds

**Journey:** Transfer money between the customer's own accounts
**Priority:** P1 — money moves
**URL:** `/parabank/transfer.htm`
**Technique key:** BVA = boundary value analysis · EP = equivalence partitioning ·
NEG = negative · ST = state transition · EG = error guessing

**Preconditions for the whole suite:** a registered customer with two accounts, the
source holding a known balance established by the tester at the start of the cycle.
Record the customer and both account numbers in the execution log.

---

## Amount field — boundaries and validation

| ID | Technique | Precondition | Steps | Expected result |
|---|---|---|---|---|
| TC-FT-001 | BVA | Source balance 1000.00 | Enter amount `0`, submit | Rejected. Validation message shown. No movement on either account |
| TC-FT-002 | BVA | Source balance 1000.00 | Enter amount `0.01`, submit | Accepted. Source debited 0.01, destination credited 0.01 |
| TC-FT-003 | BVA | Source balance 1000.00 | Enter amount `999.99`, submit | Accepted. Balances reflect exactly 999.99 |
| TC-FT-004 | BVA | Source balance 1000.00 | Enter amount `1000.00`, submit | Accepted. Source balance becomes 0.00 |
| TC-FT-005 | BVA | Source balance 1000.00 | Enter amount `1000.01`, submit | Rejected with an insufficient-funds message. **Source must not go negative** |
| TC-FT-006 | NEG | — | Enter amount `-50`, submit | Rejected. A negative transfer must not reverse the direction of the movement |
| TC-FT-007 | NEG | — | Leave amount blank, submit | Rejected. Validation message names the amount field |
| TC-FT-008 | NEG | — | Enter `abc`, submit | Rejected. No server error, no stack trace |
| TC-FT-009 | BVA | — | Enter `10.555`, submit | Either rejected, or rounded to 2dp with the rounding applied consistently to both sides. Record which |
| TC-FT-010 | NEG | — | Enter `1,000.00` with a thousands separator | Behaviour recorded. If accepted, verify it moved 1000.00 and not 1.00 |
| TC-FT-011 | NEG | — | Enter `1e3` | Rejected, or moved as 1000.00. Must not be interpreted inconsistently between validation and processing |
| TC-FT-012 | EG | — | Enter amount with a leading/trailing space | Trimmed and accepted, or rejected. Must not silently move a different figure |
| TC-FT-013 | EG | — | Paste an amount rather than typing it | Accepted. Paste must not be blocked and must not bypass validation |

## Both sides of the movement — the cases that matter most

| ID | Technique | Precondition | Steps | Expected result |
|---|---|---|---|---|
| TC-FT-014 | — | Source 500.00, destination 200.00 | Transfer 100.00. Record both balances before and after from Accounts Overview | Source exactly 400.00, destination exactly 300.00. **The sum of both accounts is unchanged** |
| TC-FT-015 | — | As above | After the transfer, open Account Activity for the source | A single debit of 100.00, dated today, no duplicate |
| TC-FT-016 | — | As above | Open Account Activity for the destination | A single credit of 100.00 matching the debit exactly |
| TC-FT-017 | — | As above | Open the transaction detail for both entries | Amount, date and type consistent between the two records and with the confirmation screen |
| TC-FT-018 | EG | Source 500.00 | Transfer 100.00, then transfer 100.00 again | Two separate debits totalling 200.00. Running balance correct after each, not just at the end |
| TC-FT-019 | EG | — | Submit the transfer form, then immediately click Transfer a second time (double-click) | **Exactly one transfer occurs.** A duplicate debit here is the highest-severity defect in this suite |
| TC-FT-020 | EG | — | Complete a transfer, then press the browser Back button and re-submit the form | No second transfer, or an explicit rejection. Re-posting must not silently move money again |
| TC-FT-021 | EG | — | Complete a transfer, note the confirmation URL, then reload that URL directly | No additional movement |

## Account selection

| ID | Technique | Precondition | Steps | Expected result |
|---|---|---|---|---|
| TC-FT-022 | NEG | Two accounts exist | Select the same account as both source and destination | Rejected, or a no-op with the balance unchanged. Must not produce a debit without a matching credit |
| TC-FT-023 | EP | Customer A logged in | Inspect the destination account dropdown | Only accounts belonging to customer A are listed |
| TC-FT-024 | NEG | Customer A logged in; an account number belonging to customer B is known | Using dev tools, alter the destination account id in the form to customer B's account and submit | **Rejected.** A customer must not be able to transfer into another customer's account by tampering with the request. This is an authorisation check, not a validation one |
| TC-FT-025 | ST | Customer logged in | Log out. Press Back to return to the transfer form and submit | Rejected, redirected to login. The session must be dead |

## Reporting

| ID | Technique | Steps | Expected result |
|---|---|---|---|
| TC-FT-026 | — | Complete a transfer and read the confirmation screen | States the amount, the source and the destination unambiguously. A customer should not have to guess which way the money went |
| TC-FT-027 | — | Request a statement for the source account after several transfers | All movements present, in order, with a running balance that reconciles to the current balance |

---

## Notes on TC-FT-019 and TC-FT-024

These two are the reason this suite exists in this shape.

**TC-FT-019 (double submission).** Nobody writes this case from a requirements
document, because no requirement says "the customer must not be able to pay twice by
double-clicking". It comes from having watched it happen. On a payment journey it is
routinely the highest-severity defect present and it is invisible to a test that only
checks the confirmation page rendered.

**TC-FT-024 (authorisation via tampering).** Field validation and authorisation are
different things, and a UI dropdown that only shows your own accounts proves nothing
about what the server accepts. Changing the value in dev tools and re-submitting takes
about thirty seconds and is the only way to find out whether the check exists on the
server or only in the browser.
