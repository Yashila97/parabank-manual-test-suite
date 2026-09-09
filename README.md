# Manual Test Suite — ParaBank

Manual test design and execution for [ParaBank](https://parabank.parasoft.com/parabank/index.htm),
Parasoft's public demo online bank. Registration, account opening, fund transfer, bill
pay, loan request and transaction history — the same journeys I test professionally on
regulated money-handling platforms.

Public demo application. No employer systems, data or documentation appear anywhere in
this repository.

## Why manual, and why say so

Most QA portfolios are automation frameworks. This one is test design, executed by hand,
documented properly — which is what the majority of QA work in regulated industries
actually is, and what an automated suite depends on somebody having done first.

A framework tells you someone can write code. A test suite like this tells you whether
they know *what to test*, which is the harder and less common skill. Every case here
names the technique that produced it, so you can see the reasoning rather than take my
word for the coverage.

## Contents

| File | What it is |
|---|---|
| [`test-approach.md`](test-approach.md) | Scope, risk-based prioritisation, environment notes, entry and exit criteria |
| [`test-cases/01-registration.md`](test-cases/01-registration.md) | 24 cases — field validation, boundaries, duplicate handling, security |
| [`test-cases/02-fund-transfer.md`](test-cases/02-fund-transfer.md) | 21 cases — the money-movement journey, including the ones that matter |
| [`test-cases/03-bill-pay.md`](test-cases/03-bill-pay.md) | 16 cases — payee validation, amount boundaries, confirmation |
| [`test-cases/04-loan-request.md`](test-cases/04-loan-request.md) | 12 cases — decision-table coverage of approval logic |
| [`artefacts/test-conditions.md`](artefacts/test-conditions.md) | The analysis that produced the cases — BVA, equivalence partitions, decision tables, state transitions |
| [`artefacts/traceability-matrix.csv`](artefacts/traceability-matrix.csv) | Requirements mapped to cases, technique and priority |
| [`artefacts/execution-log.md`](artefacts/execution-log.md) | Run results — fill in as you execute |
| [`artefacts/defect-reports.md`](artefacts/defect-reports.md) | Defects found, in the format I use professionally |

## How to use this repository

The test cases are the deliverable and they stand on their own — test design happens
before execution. To make it a complete record, work through one execution cycle
(roughly three hours for the whole suite), record actual results in the execution log,
and write up anything you find in the defect reports file.

**Do not publish invented results.** An execution log with results you did not observe
is the one thing here that could genuinely damage you, because an interviewer will ask
about a specific row and expect you to remember it.

## Author

Naga Yashila Araveti — QA Engineer, regulated payments and gaming platforms.
[LinkedIn](https://www.linkedin.com/in/naga-araveti)
