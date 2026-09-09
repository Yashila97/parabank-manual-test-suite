# Test Condition Analysis

The analysis that produced the test cases. Included because the cases on their own show
*what* was tested; this shows *why those and not others*.

---

## 1. Equivalence partitions — transfer amount

| Partition | Range | Representative value | Valid? |
|---|---|---|---|
| Zero | 0 | 0 | Invalid |
| Below minimum | 0 < a < 0.01 | 0.005 | Invalid — sub-penny |
| Valid, within balance | 0.01 ≤ a ≤ B | 100.00 | Valid |
| Exceeds balance | a > B | B + 0.01 | Invalid |
| Negative | a < 0 | -50 | Invalid |
| Non-numeric | — | "abc" | Invalid |
| Over-precise | more than 2dp | 10.555 | Boundary — behaviour must be documented |

One representative from each partition, plus every boundary between them. Seven
partitions and six boundaries gives thirteen amount cases (TC-FT-001 to TC-FT-013)
rather than an arbitrary number of "try some amounts".

## 2. Boundary values — transfer amount, source balance 1000.00

```
        invalid  |         valid          |  invalid
    ─────────────┼────────────────────────┼──────────
       0    0.01 │  0.02 ... 999.99  1000.00  1000.01
       ▲      ▲                        ▲        ▲
     TC-001  TC-002                  TC-004   TC-005
```

Four values, not four hundred. The values on either side of each boundary are where
off-by-one and comparison-operator errors live (`<` written where `<=` was meant), and
nothing in between adds information.

## 3. State transitions — session

```
    [Anonymous] ──login──▶ [Authenticated] ──logout──▶ [Ended]
         │                        │                       │
         │                   navigate to                   │
         └───rejected◀──── protected page ────rejected◀────┘
```

The transition worth testing is the invalid one: from **Ended**, can the browser Back
button reach a protected page and successfully post a transfer? That is TC-FT-025, and
it is a real defect class rather than a theoretical one, because the page is still in
the browser cache after logout.

## 4. Decision table — loan approval

Three inputs, so eight combinations. Reduced to the rows that produce distinct
behaviour, in `test-cases/04-loan-request.md`. The row that matters is **down payment
exceeds balance**: the requirement says deny, and the failure mode is that it denies the
loan *and* takes the down payment.

Testing loan amount, down payment and balance independently would never produce that
case, because it only exists in the interaction. This is the whole argument for decision
tables over field-by-field checking.

## 5. Error guessing — from experience, not from requirements

None of these appear in any specification. All of them are real defect classes on
payment journeys.

| Condition | Why it belongs | Case |
|---|---|---|
| Double-click submit | Duplicate debit. Highest severity available on this journey and invisible to a confirmation-page check | TC-FT-019, TC-BP-013 |
| Browser Back then re-submit | Re-posts the form. Same outcome as above, reached differently | TC-FT-020, TC-BP-014 |
| Reload the confirmation URL | Some implementations process on GET | TC-FT-021 |
| Same account as source and destination | Debit with no matching credit | TC-FT-022 |
| Tamper with the account id in the request | Authorisation checked in the browser only | TC-FT-024 |
| Assert balance after a *failed* transaction | The money-taken-nothing-given case | TC-LR-002, TC-LR-004 |
| Paste into a confirm-value field | Mismatch check bypassed by paste | TC-BP-006 |
| Apostrophe and accented characters in names | Breaks inserts and display encoding | TC-RG-012, TC-RG-013 |
| UK postcode in a US-shaped field | Format assumptions exclude real customers | TC-RG-017 |

## 6. What I deliberately did not test, and why

- **Cross-browser beyond one desktop and one mobile viewport.** Diminishing returns for
  a demo application; on a real platform this would be a matrix driven by analytics.
- **Load and concurrency.** Cannot be done responsibly on a shared public demo, and
  attempting it would affect other users.
- **The SOAP and REST services.** Different technique, different repository.
- **Anything requiring two customers simultaneously**, other than TC-FT-024, because
  the shared environment makes the results unreliable.

Stating the exclusions is part of the deliverable. A test plan that does not say what
it left out is not telling you its coverage.
