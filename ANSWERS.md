# Midterm 1: answers

GitHub username:Inria-io
Session code:KETTLE-62
Fingerprint:c6e8a29d

> Fill in the three lines above from `my_data/DATA.md`. Do not rename headings or table columns: a script reads the numbers.
> Write only in the empty cells and lines. Percentages as 0.78 or 78 %, money as plain numbers.

## AI-use declaration

I used Claude (Anthropic's AI assistant) during this exam to: (1) compute recall@1/3/5 and MRR@10 from retrieval_ranks.csv by counting rows where rank_A/rank_B fell within each k threshold; (2) apply the Wilson interval formula given in the exam to each candidate's recall and each pipeline's weakest cell; (3) apply the cost formulas given in the exam (per-request cost by language, weighted by traffic share, annualised) to DATA.md's token counts and prices; (4) help structure and word the 1d decision, 2b memo, and Part 3 Why lines within the word limits. I verified the recall and MRR counts by re-checking them against the raw CSV rows myself, and I can walk through the formula for any individual number in the oral check.

## Part 1: Retrieval and cost decision

### 1a. Metrics (all 60 questions)

| config | recall@1 | recall@3 | recall@5 | MRR@10 |
|---|---|---|---|---|
| A | 0.333 | 0.483 | 0.600 | 0.429 |
| B | 0.333 | 0.517 | 0.767 | 0.476 |

### 1b. Recall and 95 % Wilson interval of each candidate (recall at its own k)

| candidate | recall@k | lower | upper |
|---|---|---|---|
| A, k=3 | 0.483 | 0.362 | 0.607 |
| A, k=5 | 0.600 | 0.474 | 0.714 |
| B, k=3 | 0.517 | 0.393 | 0.638 |
| B, k=5 | 0.767 | 0.646 | 0.856 |

### 1c. Annual model cost in USD

| candidate | annual cost |
|---|---|
| A, k=3 | 22265.47 |
| A, k=5 | 29700.25 |
| B, k=3 | 14507.61 |
| B, k=5 | 16770.49 |

### 1d. Decision (at most 150 words)

Chosen candidate: B, k=5

(A,5) is disqualified immediately — at $29,700 it exceeds the $26,000 budget. Among the remaining three, (B,5) has the highest recall (0.767) with an interval [0.646, 0.856] that does not overlap (A,3)'s [0.362, 0.607] and barely doesn't overlap (B,3)'s [0.393, 0.638], so its advantage is real, not noise. Its annual cost ($16,770) leaves over $9,000 of budget headroom. By language, (B,5) recalls EN 18/20 (0.900), RU 15/20 (0.750), KK 13/20 (0.650) — Kazakh is served worst by a wide margin, a gap hidden by the overall 0.767 figure. I would still ship (B,5): it is the cheapest candidate with a statistically defensible lead over the two low-budget alternatives, and the remaining budget could fund a Kazakh-specific fix.

### 1e. Next measurement (at most 60 words)

Re-run recall@5 for config B on Kazakh questions only, with a larger Kazakh-only sample (e.g. 200 questions instead of 20), since the current 13/20 estimate has a wide interval and can't yet distinguish "KK is slightly worse" from "KK retrieval is badly broken" — which would change whether (B,5) is still the right choice.

## Part 2: A document pipeline you can defend

### 2a. Numbers

| pipeline | overall accuracy | weakest cell (language, condition) | accuracy there | lower | upper | annual cost |
|---|---|---|---|---|---|---|
| H | 0.889 | KK, clean | 0.700 | 0.521 | 0.833 | 62152.20 |
| L | 0.839 | KK, degraded | 0.767 | 0.591 | 0.882 | 10800.00 |

Personal-data pages per year sent abroad under H: 730000

### 2b. Memo (at most 300 words)

To: Head of Operations
Re: Invoice-reading pipeline recommendation

Who is served worst: Pipeline H's weakest cell is Kazakh clean scans, 21/30 = 0.700 [95% CI 0.521, 0.833] — notably below its 0.889 overall accuracy. Pipeline L's weakest cell is Kazakh degraded scans, 23/30 = 0.767 [0.591, 0.882]. Both pipelines serve Kazakh-language customers worst; the overall accuracy figures (0.889 and 0.839) conceal this, since they average over much stronger English and Russian results.

Personal data: Three questions from Lecture 5.1. Where does the scan go — under H, every page's image is sent to a hosted API abroad; under L, scans stay on our own server in Kazakhstan. Who stores it — under H, the vendor stores it on terms we would need to verify in their data-processing agreement; under L, we control storage directly. For how long — under H this is unknown without checking the vendor's retention policy; under L we set retention ourselves. 25% of scans (730,000 pages/year) carry an individual's name and ИИН — personal data under Kazakhstan's Law No. 94-V. Sending these abroad under H is a cross-border transfer of regulated personal data; a vendor's compliance statement is not itself a compliance review and would need independent verification.

Safeguards (checkable): for L, we can audit server access logs and retention settings directly. For either pipeline, redacting or masking ИИН before storage, and a signed data-processing agreement specifying retention period, are both independently verifiable.

Choice: I recommend Pipeline L. It is far cheaper ($10,800/yr vs $62,152/yr), keeps personal data inside Kazakhstan avoiding the cross-border transfer question entirely, and its accuracy (0.839) is only modestly below H's (0.889). Residual risk: L's Kazakh-degraded accuracy (0.767) is still the weakest cell overall and would benefit from a human-review step on low-confidence Kazakh extractions.

## Part 3: Explain your own numbers

### Q1

Number: 1.15
Why (at most 40 words): Byte ratio already captures UTF-8 encoding cost. A token/byte ratio above 1 means the tokenizer adds a further penalty beyond encoding, so most of the Kazakh premium comes from the tokenizer, not the encoding.

### Q2

Number: 644
Why (at most 40 words): No — e5 only embeds its first 512 own tokens, so a sentence starting at token 644 is truncated away before embedding. This is a silent failure: no visible error, the chunk simply can't retrieve on that sentence.

### Q3

Number: 80.2%
Why (at most 40 words): Top-p removes only the low-probability tail and renormalises over what's left, barely touching the top token's share. Temperature instead reshapes every probability continuously, including the top one.

### Q4

Number: 18.6%
Why (at most 40 words): I excluded arithmetic sequences and multiplication since their answers follow a fixed rule, not memorized facts. A "never invent a fact" instruction can't fix genuinely unknown facts — it can only turn guesses into refusals.

### Q5

Number: 13.65
Why (at most 40 words): At 13.65× Chinchilla's ratio, this model is heavily over-trained on tokens relative to its size. Chinchilla found models like GPT-3 were the opposite — under-trained on tokens for their parameter count.