# RAGAS Diagnostic Report 

## 1. Context

| Item | Value |
| --- | --- |
| System | PayPilot |
| Corpus | Verta documentation |
| Indices | kb_clean: section-based chunks; kb_broken: 160-character chunks |
| Profiles | clean and lesson-04 |
| Provider | Anthropic |
| Model | Default model; exact identifier to be verified |
| Dataset | golden.jsonl; final case count, retrieval count and set_hash to be added after Step 7 |
| Run dates | To be recorded when the runs are performed |


## 2. Localization

| Case | Symptom | Failure point | Evidence (retrieval context) |
|---|---|---|---|
| Exercise 1 | The retrieved tariff row contains the spread but cuts off before the allowance amount. | Indexing: the chunk boundary splits the Tier 3 row. | Clean: `tariffs.md#s2` - “\| Tier 3 \| 0.5% \| EUR 5,000 \|”. Lesson-04: `tariffs.md#b3` - “\| Tier 3 \| 0.5% \| E”; the same chunk was returned in both searches. |
| Exercise 2 | The four retrieved fragments contain either the spread or the daily limit, but not both.. | Retrieval: one of the two needed document sections was not returned. | Run 1 includes `tariffs.md#s2`: “Tier 2 \| 0.9% \| EUR 1,000”, but no daily-limit figure. Run 2 includes `limits.md#s1`: “Tier 2 \| EUR 20,000 \| EUR 200,000”, but no spread figure. Successful top_k = 5 runs retrieve both sections. |
| Exercise 3 (D25) | The tool returns the correct amount, but the answer rounds it to USD 6,500. | Generation: the prompt instructs the assistant to change the final amount. | No `tool.search_knowledge_base span`. `quote_fx.final_amount` = 6463.043478 in both runs. D25 adds: “always be rounded UP to the nearest whole hundred” and “Do NOT reconcile it against them”. |
| Exercise 4 (D05) | The assistant does not search and leaves the fee question unanswered. | Generation: the assistant does not call the search tool. | The trace contains only llm.call, with no `tool.search_knowledge_base span`. The clean run retrieved `tariffs.md#s3`: “SWIFT \| EUR 15.00 \| 0.3%”. |
| C-13 | The reported EUR 1,500 allowance did not appear in either attempt; all answers left the Tier 3 allowance unavailable. | Not reproducible in five attempts; the reported error is not classified. | All attempts retrieved `tariffs.md#b3`, ending at “Tier 3 \| 0.5% \| E”. The retrieved row was truncated, but the assistant did not invent the missing amount. |
| C-14 | The assistant lists reason codes but cannot provide the duplicate-charge window. | Indexing: the chunk cuts through the duplicate-charge row; repeated searches do not recover the missing value. | Six searches return only `disputes.md#b1` or `disputes.md#b2`. The latter ends at “goods_not_received \| 90 \| \| duplic”, before the duplicate-charge window. |


**Distribution:** Indexing - 2 (Exercise 1, C-14); Retrieval - 1 (Exercise 2); Generation - 2 (Exercises 3 and 4); Not reproducible - 1 (C-13).


## 3. Evidence of the cause

Query: “Search the Verta documentation: what FX spread applies
to my account and what is my free monthly conversion allowance?”

Customer: CUS-0003<br>
Profile: clean<br>
top_k: 1<br>
Model recorded in traces: claude-haiku-5-5<br>

Reference: tariffs.md, “FX spreads by tier”:<br>
Tier 3 spread = 0.5%; free monthly conversion allowance = EUR 5,000.

| Index | Retrieved evidence | Answer |
| --- | --- | --- |
| kb_clean | tariffs.md#s2: “Tier 3 \| 0.5% \| EUR 5,000” | States 0.5% and EUR 5,000. |
| kb_broken | tariffs.md#b3: “Tier 3 \| 0.5% \| E” | States 0.5% and says the allowance cannot be confirmed. |

Delta = kb_broken − kb_clean.

| Metric | kb_clean | kb_broken | Delta |
| --- | --- | --- | --- |
| Context recall | 2/2 = 1.00 | 1/2 = 0.50 | −0.50 |
| Context precision, first search | 1/1 = 1.00 | 1/1 = 1.00 | 0.00 |
| Faithfulness of stated spread/allowance values | 2/2 = 1.00 | 1/1 = 1.00 | 0.00 |

**Counting scope**: recall checks the two reference facts. Precision
checks whether the returned chunk helps answer the question.
Faithfulness here checks only the numerical spread and allowance
claims, not every statement in the full response. The broken-index
answer does not invent an allowance, so its single stated value
is supported.

**Context recall**: did the search bring both facts?
- `kb_clean`: brought 2 of 2 facts → 2 ÷ 2 = 1.00.
- `kb_broken`: brought only the spread, 1 of 2 facts → 1 ÷ 2 = 0.50.

**Context precision**: were the returned fragments relevant (one fragment: tariffs.md#s2)?
- `kb_clean`: 1 relevant fragment out of 1 → 1.00.
- `kb_broken`: 1 relevant fragment out of 1 → 1.00.
The broken fragment is relevant because it contains the Tier 3 spread. Relevant does not mean complete.

**Faithfulness**: are the numbers the assistant stated supported by the fragment?
Here we check only the spread and allowance values:
- `kb_clean`: stated two numbers, both supported → 2/2 = 1.00.
- `kb_broken`: stated one number, the supported spread → 1/1 = 1.00. It did not state an allowance.

**Unchanged**: profile, system prompt, model, top_k, customer and
customer question. Only the index setting was changed.

**Conclusion**: the observed failure is indexing - the broken chunk
cuts off the allowance value, so the assistant receives the spread
but cannot provide the complete requested answer.

## 4. Correct tool data, incorrect answer

Defect: D25<br>
Profile / customer: clean + D25 / CUS-0005<br>
Search span: absent

In the initial comparison:
- quote_fx.final_amount: 6463.043478 USD in both conditions.
- Clean answer: 6,463.04 USD.
- D25 answer: 6,500 USD.

D25 instruction:
“always be rounded UP to the nearest whole hundred”
and “Do NOT recompute it from the components, do NOT reconcile
it against them, and do NOT show the unrounded value anywhere”.

Five-run results:
- clean: incorrect final amount in 0/5 runs; all stated USD 6,463.04.
- clean + D25: incorrect final amount in 5/5 runs; all stated USD 6,500.

**Why retrieval metrics are not calculated:**
There was no knowledge-base search, so no retrieved document
context exists to evaluate. Recall and precision are marked “-”.
Retrieval-based faithfulness is also marked “-”; consistency
with tool data can still be checked separately.

**Evidence of a generation failure:**
The initial traces show that the tool returned the same correct
amount in both conditions. The D25 answer changed that amount
to USD 6,500, following the added rounding instruction.
The error appeared in the answer, not in the tool result.

**D05 control:**
The clean run searched twice and retrieved tariffs.md#s3,
which states a SWIFT fee of EUR 15.00 plus 0.3% of the transfer
amount in EUR equivalent. The answer stated both components.<br>
With D05 enabled, the trace contained only llm.call and no
search span. The assistant said it could not confirm the fee
and offered to search or escalate.<br>
The lesson’s invented fixed-fee answer was not reproduced.
The observed D05 failure was skipping search and leaving the
fee question unanswered.

## 5. Trade-off

Query: “знайди в документації один раз і скажи і спред, і денний ліміт переказів для Tier 2”

Configuration: clean profile, kb_clean index, no customer.

Reference facts:
- Tier 2 FX spread: 0.9% - `tariffs.md#s2`.
- Tier 2 daily transfer limit: EUR 20,000 - `limits.md#s1`.

Recall counts the requested facts retrieved.
Precision counts fragments that directly provide those facts.
The counts below combine all recorded runs for each top_k.

| `top_k` | Context recall | Context precision | Complete answers | Average input tokens per turn |
|---|---|---|---|---:|
| 4 | 3/6 = 0.50 | 3/12 = 0.25 | 0/3 | 6,137.3 |
| 5 | 6/6 = 1.00 | 6/15 = 0.40 | 3/3 | 6,375.0 |
| 6 | 6/6 = 1.00 | 6/18 ≈ 0.33 | 3/3 | 6,495.7 |


**Retrieved evidence:**
At top_k = 4, two runs retrieved tariffs.md#s2 with the spread
but no daily-limit figure. One run retrieved limits.md#s1 with
the daily limit but no spread figure.
At top_k = 5 and top_k = 6, three runs retrieved both required sections.

**Search language:**
The agent generated Ukrainian and English queries.
For example, “Tier 2 спред валютна конвертація денний ліміт переказів”
returned the spread section within the first four results,
while “Tier 2 FX spread and daily transfer limit” returned
the limits section but omitted the spread section at top_k = 4.

Different search words match different fragments, changing their order. With top_k = 4, only the first four are returned-a needed fragment ranked fifth is left out.
It is language and wording together. 

**Broken index:**
Configuration: clean profile, CUS-0003, kb_broken.
Query: “Search the Verta documentation: what FX spread applies
to my account and what is my free monthly conversion allowance?”

| top_k | Was tariffs.md#b4 retrieved? | Answer |
| --- | --- | --- |
| 1 | No | Confirmed 0.5%; could not confirm the allowance. |
| 5 | No, in either search | Confirmed 0.5%; still could not confirm the allowance. |

At top_k = 5, both searches returned `tariffs.md#b3`, ending at
“Tier 3 \| 0.5% \| E”. The additional fragments did not contain
the missing allowance. `fx-guide.md#b4` appeared, but it is not
the required `tariffs.md#b4`.

Increasing top_k from 1 to 5 did not fix this observed
broken-index failure.

**Selected value:** top_k = 6 for the tested two-part question.

**Trade-off:** In the three runs shown in the table, both top_k = 5 and top_k = 6 produced complete answers. 
We choose top_k = 5: it used approximately 121 fewer input tokens per turn and returned fewer unnecessary fragments. 
Two of its five fragments supplied the requested spread and daily limit.

**Why acceptable under Ship it:** the small observed increase
in token usage is acceptable for more complete answers.
Three successful runs do not guarantee reliability across other questions.

## 6. Decisions and limitations

| Failure point | Fix | Cost in a real system | Trade-off | Can a prompt help? |
| --- | --- | --- | --- | --- |
| Indexing | Keep table rows and their headings together; rebuild the index. | Engineering time, processing the corpus again, index storage and validation. | Larger chunks may include unnecessary text and increase input tokens. May break previously working queries. | A prompt cannot restore missing text. |
| Retrieval | Tune search queries, ranking and top_k. Provisionally use top_k = 5 for the tested two-part question. | More returned text increases token usage; reranking adds processing time and possibly model cost. | More results can bring irrelevant or conflicting information. | No-not as a fix for this retrieval defect. Change top_k, ranking or the search method. |
| Generation | Remove D25’s rounding instruction and require final amounts to match tool results to two decimal places. | Prompt maintenance and regression tests. Additional searches can increase latency and token usage. | Stricter grounding may produce more refusals when evidence is missing. | Yes, but the fix still requires testing. |

**Illustrative reindexing budget:**

Assumptions:
- Corpus: 10,000 documents, averaging 1,000 tokens each.
- Total embedding input: 10,000 × 1,000 = 10 million tokens,
  assuming no additional tokens from chunk overlap.
- Assumed embedding price: USD 0.10 per million tokens.
- Estimated processing and index-building time: 1 hour.
- Validation: 1 engineer-hour at an assumed USD 30/hour.

Embedding cost: 10 × USD 0.10 = USD 1.<br>
Labour cost: 1 × USD 30 = USD 30.<br>
Estimated total: USD 31, excluding infrastructure and storage.<br>
Estimated elapsed time: 2 hours if rebuilding and validation
are performed sequentially.<br>

These are planning assumptions, not measured results.

**What a prompt cannot fix:**
It cannot repair a tariff row split across chunks. In our kb_broken
runs, increasing top_k from 1 to 5 still did not retrieve
tariffs.md#b4, and the Tier 3 allowance remained unavailable.

**Residual risk:**

These metrics do not check whether the bank’s documents are
up to date. If both the reference and the retrieved document
contain the same outdated fee, all three metrics can pass
while the customer receives incorrect information.

We record this as a limitation of the current evaluation.
Before release, the document owner must confirm that the
documents are current, and separate checks must compare
answers with the current banking rules.

C-13’s reported EUR 1,500 allowance was not reproduced in two
attempts. Both attempts returned a truncated row and withheld
the allowance. Other runs could still produce an unsupported value.

**Reference sources:**
Verta corpus documents, not assistant answers:
- tariffs.md: FX spreads, conversion allowances and SWIFT fees.
- limits.md: daily and monthly transfer limits.
- disputes.md: dispute reason codes and their windows.


## Final decision

What we traded: provisionally chose top_k = 5 to retrieve both
requested facts, accepting more input tokens than top_k = 4.

What we are not doing: treating a larger top_k as a repair for
broken chunks, or treating green metrics as proof that the
source documents are correct.

When we will reconsider: if further tests show missing facts,
token usage or latency exceeds our budget, or Verta’s mandate
changes from Ship it to Zero regulatory risk.

---

## Чекліст перед здачею

- [ ] У розділі 1 заповнено всі рядки, зокрема кількість retrieval-кейсів і `set_hash`.
- [ ] У розділі 2 кожен рядок має доказ із трейсу, а не цитату відповіді; C-13 і C-14 класифіковано або позначено «не відтворюється».
- [ ] У розділі 3 між прогонами змінився лише індекс, і це названо явно.
- [ ] Біля кожної метрики показано рахунок «n з m».
- [ ] У розділі 4 є результат п'яти прогонів D25 і `final_amount` з трейсу.
- [ ] У розділі 5 три значення `top_k` з токенами й перевірка на `kb_broken`; названо ціну обраного значення.
- [ ] У розділі 6 заповнено всі три рядки таблиці, є клас збоїв, який проходить усі три метрики, і джерело еталону.
- [ ] Останній рядок заповнено всіма трьома пунктами.
- [ ] У `golden.jsonl` щонайменше 10 кейсів з `layer: retrieval` і `oracle: corpus`, у жодного немає порожнього `reference`.
- [ ] `docker compose run --rm eval --set golden --profile clean` на чистому клоні кіту доходить до кінця і пише звіт.
