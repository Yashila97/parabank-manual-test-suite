# Test Cases — New Customer Registration

**Journey:** Register a new customer
**Priority:** P1 — gateway to every other journey, and where authentication weaknesses live
**URL:** `/parabank/register.htm`
**Technique key:** BVA = boundary value analysis · EP = equivalence partitioning ·
NEG = negative · EG = error guessing · SEC = security-adjacent

---

## Mandatory field validation

| ID | Technique | Steps | Expected result |
|---|---|---|---|
| TC-RG-001 | — | Submit the form completely empty | Every mandatory field flagged individually. Not one generic "please complete the form" |
| TC-RG-002 | NEG | Complete everything except First Name, submit | Rejected. Message identifies First Name specifically |
| TC-RG-003 | NEG | Complete everything except Last Name | Rejected, Last Name named |
| TC-RG-004 | NEG | Complete everything except Address | Rejected, Address named |
| TC-RG-005 | NEG | Complete everything except Username | Rejected, Username named |
| TC-RG-006 | NEG | Complete everything except Password | Rejected, Password named |
| TC-RG-007 | NEG | Enter Password and a different Confirm value | Rejected with a mismatch message. The customer must be told which field to correct |
| TC-RG-008 | — | Complete all fields with valid data, submit | Account created. Customer logged in, welcome message shown, account number issued |

## Field boundaries and formats

| ID | Technique | Steps | Expected result |
|---|---|---|---|
| TC-RG-009 | BVA | First Name = 1 character | Accepted |
| TC-RG-010 | BVA | First Name = 255 characters | Accepted or rejected with a stated limit. Must not truncate silently |
| TC-RG-011 | BVA | First Name = 256+ characters | Rejected cleanly. No server error |
| TC-RG-012 | EP | First Name = `O'Brien` | Accepted. An apostrophe is a normal name character and must not break the insert |
| TC-RG-013 | EP | First Name = `José-María` | Accepted and stored with accents intact. Check it displays correctly after login, not just that it submitted |
| TC-RG-014 | NEG | First Name = `12345` | Behaviour recorded. Numeric names are unusual but not invalid; note whether validation is stricter than it should be |
| TC-RG-015 | BVA | Phone = 1 digit | Rejected, or accepted with the format documented |
| TC-RG-016 | EP | Phone = `+44 7721 547967` | Accepted. International format must not be rejected |
| TC-RG-017 | EP | Zip Code = `CV1 2TT` (UK format, contains a space and letters) | Accepted. A five-digit-only validation rule is a defect for a non-US customer |
| TC-RG-018 | NEG | SSN field = alphabetic characters | Behaviour recorded |
| TC-RG-019 | EG | Enter values with leading and trailing whitespace throughout | Trimmed on save. Verify by checking the profile page afterwards |

## Duplicate and uniqueness handling

| ID | Technique | Precondition | Steps | Expected result |
|---|---|---|---|---|
| TC-RG-020 | NEG | Username `ntest010101` already registered | Register again with the same username | Rejected with a clear message. No second account created |
| TC-RG-021 | EP | As above | Register with the same first name, last name and address but a new username | Accepted. Personal details are not required to be unique |
| TC-RG-022 | EG | As above | Register with the same username in different case (`NTest010101`) | Behaviour recorded. If accepted, two accounts now differ only by case, which will confuse login and support |

## Security-adjacent

| ID | Technique | Steps | Expected result |
|---|---|---|---|
| TC-RG-023 | SEC | Password = `a` (single character) | Behaviour recorded. A bank with no password policy is worth raising even on a demo |
| TC-RG-024 | SEC | Observe the Password and Confirm fields while typing | Characters masked |
| TC-RG-025 | SEC | Submit the form with dev tools Network open, inspect the request | Sent over HTTPS. Password not present in the URL or in a GET query string |
| TC-RG-026 | SEC | First Name = `<script>alert(1)</script>`, register, then view the welcome message and profile | Rendered as literal text, never executed. Check every page the value appears on, not just the first |
| TC-RG-027 | SEC | Last Name = `Smith'; DROP TABLE users;--` | Accepted as text or rejected. No server error, no evidence of query interference |
| TC-RG-028 | EG | Register, log out, then attempt login with the correct username and wrong password twice | Rejected. Note whether the message distinguishes "no such user" from "wrong password" — enumeration risk |
