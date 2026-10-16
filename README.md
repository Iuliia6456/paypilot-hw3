# L04 · RAGAS Diagnostic Report - PayPilot

HW2 dataset source: https://github.com/Iuliia6456/paypilot-hw2

## Execution

Copy this repository's golden.jsonl into the evaluation kit's
sets directory, then run from the evaluation kit directory:

    docker compose run --rm eval --set golden --dry-run
    docker compose run --rm eval --set golden --profile clean
    docker compose run --rm eval --set golden --profile lesson-04

Use a local PayPilot stand with:
CLOCK_OVERRIDE=2026-09-15T10:00:00Z

**Dataset set_hash**: 88d07329048b

**Dataset**: 49 cases, including 10 retrieval cases.<br>
**Daily run**: 44 cases. Five release-only human cases are excluded.

Validation was performed using a copy named sets/hw3.jsonl:

    docker compose run --rm eval --set hw3 --dry-run

## Retrieval cases

| ID | Source | Corpus file and location |
| --- | --- | --- |
| RET-H01 | Complaint C-13, narrowed to Tier 3 allowance | tariffs.md - FX spreads by tier, Tier 3 row |
| RET-H02 | Complaint C-14 | disputes.md - Reason codes and chargeback windows, full table |
| RET-H03 | Corpus | limits.md - Daily and monthly limits by tier, Tier 1 row |
| RET-H04 | Corpus | limits.md - Daily and monthly limits by tier, Tier 3 row |
| RET-H05 | Corpus | disputes.md - Reason codes and chargeback windows, unauthorized_debit row |
| RET-H06 | Corpus | tariffs.md - Transfer fees by type, SEPA row |
| RET-H07 | Corpus | tariffs.md - Transfer fees by type, Card payment row and calculation-basis sentence |
| RET-H08 | Corpus | disputes.md - Reason codes and chargeback windows, service_not_rendered row |
| RET-H09 | Corpus | tariffs.md - FX spreads by tier, Tier 1 row and applicability sentence |
| RET-H10 | Corpus | limits.md - Daily and monthly limits by tier, Tier 2 row |

The contains assertions check selected text in the answer.
They do not prove that search occurred or that every statement
is correct. Retrieval failures will be diagnosed from traces.

## Runs

| Run | Profile | Result | Report |
| --- | --- | --- | --- |
| Baseline | clean | Pending / 44 | Pending |
| Lesson profile | lesson-04 | Pending / 44 | Pending |

## Prediction before lesson-04

Expected failures:
- RET-H01: the broken tariff chunk may omit the Tier 3 allowance.
  Manual reproduction returned tariffs.md#b3 ending at the start
  of that value.
- RET-H02: the duplicate-charge window may not reach the model.
  Manual reproduction returned disputes.md#b1 and #b2 without
  the required 60-day value.
- RET-H05 and RET-H08: the later dispute-table rows may be omitted
  from the top-ranked broken chunk, hiding the required windows.

Other cases remain uncertain: search wording, repeated searches
and D05 may change which evidence reaches the model or whether
it searches at all. No specific failure is predicted for them.

Prediction commit: pending.

Observed failures: pending.
Difference from prediction: pending.