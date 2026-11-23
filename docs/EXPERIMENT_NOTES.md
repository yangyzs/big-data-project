# Experiment notes (verify before final submission)

## Fixed settings

- Fuzzy baseline: `rapidfuzz.fuzz.ratio` on lowercased and stripped names; threshold 80.
- Embedding model: `sentence-transformers/all-MiniLM-L6-v2` with normalized vectors, cosine similarity, top-5 left candidates per right value.
- LLM: `Qwen/Qwen2.5-1.5B-Instruct`; deterministic generation (`do_sample=False`, `max_new_tokens=12`), generic entity identity prompt; output classes YES / NO / UNSURE. Accept only YES.
- Invalid model responses are recorded separately as ABSTAIN for evaluation.

## Quality checks before release

- [ ] Copy the **latest working combined notebook** into notebooks/; don't reuse a stale debug notebook.
- [ ] Remove duplicate definitions and experiment scratch cells.
- [ ] Remove `%pip uninstall` and `pip install -U transformers` cells.
- [ ] Confirm variables have dataset prefixes (`basket_`, `country_`, `device_`) and do not overwrite the ground-truth DataFrame from a different dataset.
- [ ] Verify all saved result CSVs against ground-truth ID pairs.
- [ ] Clear notebook cell outputs containing redundant stack traces/model warnings before committing; keep key metric tables or export them to CSV.
- [ ] Document actual runtime/library/GPU versions.
- [ ] Note that recorded LLM time is per-pair aggregation, not full pipeline runtime.
- [ ] If including large raw prediction files, ensure they contain no secrets or Drive authentication tokens.

## Known error-analysis findings

- Country: 477 fuzzy matches outside top-5 include 4 true matches and 473 false positives.
- Device: 917 fuzzy matches outside top-5 include 6 true matches and 911 false positives.
- Device Qwen-only: 1,799 false positives; examples include same brand but different models and similar product categories.
- Device Qwen-only: 118 false negatives, of which 113 were missed by top-5 retrieval and 5 were retrieved but rejected by Qwen.
- Country Qwen outputs: 4 invalid responses (`SURE`), recorded as ABSTAIN.
- Device Qwen outputs: 9 invalid responses, recorded as ABSTAIN.

## Scope and limitations

The threshold was selected during analysis on BasketballTeam. Country and Device provide two additional evaluations using the same fixed parameters, but the findings are limited to these benchmark datasets. Do not claim that predicted LLM call savings are end-to-end runtime savings.
