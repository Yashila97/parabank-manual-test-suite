# Test Approach — ParaBank

**Application under test:** https://parabank.parasoft.com/parabank/index.htm
**Type:** Public demo online banking application (Parasoft). Not a real bank.
**Source:** Open source — https://github.com/parasoft/parabank
**Tested on:** _record your date and browser here_

---

## 1. Scope

**In scope**

- New customer registration
- Login, logout, forgotten login details
- Account opening (checking and savings)
- Transfer funds between own accounts
- Bill pay to an external payee
- Loan request
- Account overview, activity and transaction detail
- Update contact information

**Out of scope**

- The SOAP and REST web services (covered separately — see the API test design
  repository for how I approach API layers)
- Performance and load
- Browser compatibility beyond one desktop and one mobile viewport
- Anything requiring administrative access

## 2. Prioritisation

Effort follows financial exposure, not feature size. On a banking application the
severity of a defect has almost nothing to do with how visible it is.

| Priority | Area | Why |
|---|---|---|
| P1 | Fund transfer, bill pay | Money moves. A wrong amount or a duplicate debit is the worst outcome the system can produce, and neither necessarily shows an error |
| P1 | Registration, login | Gateway to everything; also where authentication and enumeration weaknesses live |
| P2 | Account opening, loan request | Creates records that later journeys depend on |
| P2 | Transaction history, statements | Wrong here means the customer cannot reconcile their own money |
| P3 | Contact information update | Data quality, no direct financial impact |

**The one that gets disproportionate attention:** transfer between accounts. It is the
only journey where an incorrect result can look completely normal. A transfer that
debits £100 and credits £10 produces two plausible-looking screens and no error, and the
customer finds it days later. Cases TC-FT-014 through TC-FT-021 exist specifically to
catch that class of failure by asserting both sides of the movement and the running
balance, rather than only checking that a confirmation page appeared.

## 3. Test design techniques used

Named per case in the suites so the reasoning is visible.

- **Equivalence partitioning** — account states, amount ranges, payee validity
- **Boundary value analysis** — amount fields (0, 0.01, minimum, maximum, over
  maximum), field lengths, phone and postcode formats
- **Decision tables** — loan approval, which depends on requested amount, down payment
  and the source account balance together
- **State transition testing** — the account lifecycle, and the session lifecycle after
  logout
- **Negative testing** — every input field, plus the journeys that should be refused
- **Error guessing** — informed by six years on payment and wallet systems: duplicate
  submission by double-click, browser back button after a completed transaction, session
  timeout mid-journey, transfer to the same account

## 4. Environment and data

ParaBank is a shared public demo. Two consequences that matter for how the suite is
written:

1. **Data is not yours alone.** Other people are registering and transferring at the
   same time. Every case that depends on a specific balance establishes that balance
   itself rather than assuming one, and no case asserts an absolute account number.
2. **The database is periodically reset.** Register a fresh customer at the start of
   each cycle rather than reusing one across days. Record the username you used in the
   execution log so a result can be re-checked.

Test data convention: username `ntest<ddmmyy><n>`, so a run is identifiable and
repeatable without collision.

## 5. Entry and exit criteria

**Entry.** Application reachable. A registered customer with at least one checking
account and a non-zero balance. Browser dev tools available for inspecting requests
where a UI result needs confirming at the network layer.

**Exit.** All P1 cases executed. No open critical or high defects in fund transfer or
bill pay. Every executed case has an actual result recorded and a pass or fail status.
Any case not executed is marked blocked with a reason, not left blank.

## 6. Risks to the testing itself

| Risk | Handling |
|---|---|
| Shared environment produces confusing results | Establish preconditions inside each case; never assume a starting balance |
| Demo application has known unfixed defects | Record them; do not treat a known defect as a blocker for unrelated cases |
| Database reset mid-cycle | Note the time in the log; re-establish data and re-run affected cases |
| Session timeout during a long cycle | Split execution into sessions per suite |
