# Defect Reports

Defects found while executing the suite, in the format I use professionally.

**Write these up as you find them. Do not invent any.** Two real, well-written defects
are worth more than ten fabricated ones, and a fabricated one will not survive being
discussed.

---

## Template

**ID:** PB-001
**Title:** _One line. What happens, where. Specific enough to be searchable_
**Severity:** _S1 incorrect money movement or security · S2 journey blocked, no
workaround · S3 functional defect with a workaround · S4 cosmetic_
**Priority:** _Immediate · Before release · Scheduled · Backlog_
**Found by:** _Case ID_
**Environment:** _URL, browser and version, OS, date_

### Steps to reproduce
1.
2.
3.

### Expected result

### Actual result

### Evidence
_Screenshot filenames in `evidence/`, the relevant request or response from dev tools,
balances before and after. Redact nothing — this is a public demo — but do not paste
anything from your employer's systems._

### Impact
_Who is affected, in what circumstances, and what it costs them. This is the section
that decides whether the defect gets fixed, and it is the section most testers skip._

### Root cause hypothesis
_Optional, and only if you have evidence. A wrong guess stated confidently costs you
credibility with developers; "the check appears to be client-side only, since the
tampered request succeeded" is evidence-based and useful._

---

## What to look for while executing

Areas where this suite is most likely to find something, based on where these defect
classes usually live rather than on any knowledge of this particular application:

1. **Balance after a rejected transaction** — TC-LR-002, TC-LR-004, TC-FT-005. The
   money-taken-nothing-given class.
2. **Double submission** — TC-FT-019, TC-BP-013. Check the activity list, not the
   confirmation screen.
3. **Client-side-only authorisation** — TC-FT-024. Thirty seconds in dev tools.
4. **Input encoding on display** — TC-RG-026. Check *every* page the value appears on.
   Escaped on the welcome banner and unescaped in the account activity list is a common
   split.
5. **Non-US formats** — TC-RG-017. A five-digit zip validation excludes every UK
   customer, which on a real product is a revenue defect and not a cosmetic one.
6. **Rounding** — TC-FT-009. Whether both sides of the movement round the same way.

## Severity guidance for this application

| Finding | Severity | Reasoning |
|---|---|---|
| Duplicate debit from one submission | S1 | Incorrect money movement |
| Transfer into another customer's account by tampering | S1 | Authorisation failure |
| Denied loan still debits the down payment | S1 | Money taken, nothing given |
| Source account can go negative | S1 | Control failure |
| Stored script executes on any page | S1 | Security |
| Session survives logout | S2 | Security, but requires physical access to the browser |
| UK postcode rejected | S3 | Excludes real customers, has a workaround |
| Validation message does not name the field | S3 | Usability, accessibility-adjacent |
| Inconsistent date format between screens | S4 | Cosmetic |

Note that nothing in the S1 list produces an error page. That is the point of the
severity scale being calibrated on financial exposure rather than on visibility.
