# L04 · RAGAS Diagnostic Report - PayPilot

HW2 dataset source: https://github.com/Iuliia6456/paypilot-hw2

## Execution

Copy this repository's golden.jsonl into the evaluation kit's
sets directory, then run from the evaluation kit directory:

    docker compose run --rm eval --set hw3 --dry-run
    docker compose run --rm eval --set hw3 --profile clean
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
| RET-H01 | Complaint C-13, narrowed to Tier 3 allowance | tariffs.md, line 16 - Tier 3 spread and allowance |
| RET-H02 | Complaint C-14 | disputes.md, “Reason codes and chargeback windows” table; line 11 - duplicate-charge window |
| RET-H03 | Corpus | limits.md, line 11 - Tier 1 monthly limit |
| RET-H04 | Corpus | limits.md, line 13 - Tier 3 daily limit |
| RET-H05 | Corpus | disputes.md, line 13 - unauthorized_debit window |
| RET-H06 | Corpus | tariffs.md, line 22 - SEPA fee |
| RET-H07 | Corpus | tariffs.md, lines 24 and 26 - card-payment fee and calculation basis |
| RET-H08 | Corpus | disputes.md, line 12 - service_not_rendered window |
| RET-H09 | Corpus | tariffs.md, line 14 - Tier 1 spread; applicability paragraph beginning at line 9 |
| RET-H10 | Corpus | limits.md, line 12 - Tier 2 monthly limit |

The contains assertions check selected text in the answer.
They do not prove that search occurred or that every statement
is correct. Retrieval failures will be diagnosed from traces.

## Runs

| Run | Profile | Result | Report |
| --- | --- | --- | --- |
| Baseline | clean | 44/44 passed; retrieval 10/10 | reports/hw3-clean-20261016T214648.json |
| Lesson profile | lesson-04 | 36/44 passed; retrieval 3/10 | reports/hw3-lesson-04-20261016T215211.json |

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

Prediction commit: 995075b.

Observed retrieval failures:
RET-H01, RET-H02, RET-H05, RET-H07, RET-H08, RET-H09, RET-H10.

The existing generation case HW-C13 also failed.

Difference from prediction:
All four predicted cases failed. However, RET-H01 failed because
the agent did not search, rather than because it retrieved a
truncated allowance row.

RET-H02, RET-H05 and RET-H08 retrieved fragments that lacked
the required dispute windows, consistent with the prediction.

The additional failures RET-H07, RET-H09 and RET-H10 also
had no tool calls. Their failure therefore cannot be attributed
to retrieved chunk contents.

RET-H03, RET-H04 and RET-H06 passed their contains checks.
These passes confirm the expected text appeared, not that
every aspect of the answer or retrieval was correct.

---
## Revision after feedback

**Revised dataset set_hash**: 7ed831290a4b.<br>
The dataset still contains 49 cases: 44 daily and five release cases.

Four cases were replaced:
- RET-H05: goods_not_received dispute window.
- RET-H06: Internal transfer flat fee.
- RET-H07: Tier 1 free monthly conversion allowance.
- RET-H09: effective date of a tier change.

### Prediction before the revised runs

On clean, I expect all ten retrieval cases to pass.

On lesson-04:
- RET-H01 and RET-H10 are likely to fail again: the previous
  run skipped search for these questions.
- RET-H02 and RET-H08 are likely to fail because the retrieved
  dispute fragments previously omitted their required windows.
- RET-H05 may pass: disputes.md#b2 contains goods_not_received
  and its 90-day window.
- RET-H06 may pass: tariffs.md#b4 contains the complete Internal
  fee row.
- RET-H07 may pass: tariffs.md#b3 contains the complete Tier 1
  allowance.
- RET-H09 is uncertain: the effective-date sentence may be split
  or absent from the single returned fragment.
- RET-H03 and RET-H04 may pass.


Revised results: pending.