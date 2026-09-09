# Test Cases — Bill Pay

**Journey:** Pay an external payee
**Priority:** P1 — money leaves the customer's control entirely
**URL:** `/parabank/billpay.htm`

Bill pay carries a risk that internal transfer does not: the money goes to a third
party, so an error is not recoverable by an internal correction. Payee detail validation
therefore matters more here than anywhere else in the application.

| ID | Technique | Steps | Expected result |
|---|---|---|---|
| TC-BP-001 | — | Complete all payee fields with valid data and a valid amount, submit | Payment accepted. Confirmation states payee name, amount and source account |
| TC-BP-002 | NEG | Submit with Payee Name blank | Rejected, field named |
| TC-BP-003 | NEG | Submit with Address blank | Rejected, field named |
| TC-BP-004 | NEG | Submit with Account Number blank | Rejected, field named |
| TC-BP-005 | NEG | Enter a Verify Account value that differs from Account Number | Rejected with a mismatch message. **This check is the only thing standing between a typo and money sent to a stranger** |
| TC-BP-006 | EG | Enter Account Number, then paste a different value into Verify Account | Still rejected. The mismatch check must not be bypassable by paste |
| TC-BP-007 | BVA | Amount `0` | Rejected |
| TC-BP-008 | BVA | Amount `0.01` | Accepted, exactly 0.01 debited |
| TC-BP-009 | BVA | Amount equal to the full source balance | Accepted, source becomes 0.00 |
| TC-BP-010 | BVA | Amount one penny over the source balance | Rejected. Source must not go negative |
| TC-BP-011 | NEG | Amount `-100` | Rejected. Must not credit the customer |
| TC-BP-012 | — | Complete a payment, then check Account Activity on the source | Exactly one debit, matching the confirmed amount |
| TC-BP-013 | EG | Double-click Send Payment | Exactly one payment. A duplicate here is unrecoverable |
| TC-BP-014 | EG | Complete a payment, press Back, re-submit | No second payment |
| TC-BP-015 | SEC | Payee Name = `<script>alert(1)</script>`, submit, then view Account Activity and the transaction detail | Rendered as text on every page it appears |
| TC-BP-016 | EP | Payee Account Number = alphabetic characters | Rejected, or the accepted format documented |
