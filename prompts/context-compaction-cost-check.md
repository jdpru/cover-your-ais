# Context Compaction Cost Check

Copy the prompt below into your coding agent to check whether context compaction would make an LLM agent in your codebase cheaper without lowering task success.

```text
Check whether context compaction would make the LLM agent in this codebase cheaper without lowering task success. Use the recipe from NVIDIA's "You don't need long context" (arXiv 2611.05306). Do not change production behavior: put any prototype behind a flag that is off by default, and ask me before enabling it, merging it, or spending more than $5 on test runs.

1. Find the agent loop. Locate the model calls and how context is built across turns: history appending, truncation or sliding windows, existing summarization or compaction, context and output limits, how big tool outputs get, which models and provider are used, and whether prompt caching is on. If there is no multi-turn loop (for example, only single-shot calls), say so plainly and stop. If large inputs (documents, logs, files) are pasted into prompts, note it.

2. Estimate the savings. Use logs, traces, or test runs (or reasoning if none exist) to get typical and worst-case turns per task, peak context, uncached input, cached input, and output tokens. Price them at the provider's real rates, including cache discounts and any long-context surcharge. Compaction cuts how much history is re-sent, but each restart re-sends uncached input and adds recap output, so it can cost more when most history is already cheap cached input. Say where it should help and by roughly how much, or say it probably won't.

3. Prototype it behind the flag:
- Use two settings: a compaction threshold and a hard limit about 33% higher. Count the system prompt and tool definitions too. Start at 36K/48K tokens, then try 24K/32K. Go to 12K/16K only if the tests hold up; verbose or smaller models collapsed there.
- When the context passes the threshold, send this as the next user message in the same conversation:
  "Stop work. Write only a concise handoff recap; do not call tools. The conversation will reset to the system prompt, the task, and this recap. Record: (1) Objective, in one line. (2) Established facts: exact paths, symbols, commands, values, and errors, with observations kept separate from unverified ideas. (3) Changes made: each file and whether it was verified. (4) Verification: checks run and their results. (5) Dead ends not to repeat. (6) Remaining plan in order, the very next step, and the condition for being done. Copy verbatim any instructions or preferences the user gave after the original task. If needed output was truncated, give a narrow command to get it back."
- Then start a fresh conversation with the same system prompt, the original task verbatim, and the recap. Files and environment stay as they are. Repeat each time the threshold is crossed. If the recap doesn't fit under the hard limit, end the run and count it as a failure.
- If large inputs get pasted into prompts, also try storing them as files and letting the agent search them and read small windows instead.

4. Test before and after. Pick 10–30 of this codebase's own tasks that have automatic pass/fail checks (evals, tests, golden outputs). Run the baseline and each compaction setting with the same model and settings, and repeat runs if they're cheap. For each task, record: success, uncached input, cached input, and output tokens, number of compactions, peak context, cost at real prices, and wall time. Call it a win only if success stays within noise and real cost goes down.

5. Report back with what you found, a results table, a recommendation, and exactly what needs my approval.
```

## Source and caveats

- Source: NVIDIA, "You don't need long context", arXiv 2611.05306 (preprint, Nov 3 2026).
- The paper's 2–9x savings are NVIDIA's serving-cost estimates, not provider prices. At real list prices with prompt caching, the paper's own tables show smaller savings and sometimes higher cost.
- Results vary by model, and tight budgets (16K) hurt verbose or smaller models.
- The recipe (loop, thresholds, recap fields) comes from the paper. The $5 cap, off-by-default flag, real-price billing and test design were added.
