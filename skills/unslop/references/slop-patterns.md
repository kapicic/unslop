# Slop patterns

What makes model-written prose tiring: it spends words on things other than information. Each pattern below wastes the reader's attention in a specific way. Use them as a lens, not a word list — a phrase from this page can be fine in context, and slop shows up in phrasings not listed here. The test is always: remove the sentence or phrase; did the reader lose a fact from the ledger? If not, it goes.

## 1. Saying it three times
The document opens with a thesis callout, repeats it in a summary table, again in each section's "takeaway", and once more in "What changed". Keep the one placement where the reader needs it (usually the top, or the table if the table is the deliverable) and cut the echoes.

> Before: **Bottom line:** the cache fix cut p95 latency by 40%. … *(three sections later)* … In summary, the cache fix delivered a 40% p95 improvement, which is a significant win.
> After: The cache fix cut p95 latency by 40%. *(said once)*

## 2. Throat-clearing and signposting
Sentences that announce content instead of delivering it: "It's worth noting that", "Importantly,", "Here's the thing:", "Let's dive in", "This section covers", "As mentioned above", "To put it simply".

> Before: It's important to note that, as discussed earlier, the migration requires a maintenance window.
> After: The migration needs a maintenance window.

## 3. Empty hedges vs real uncertainty
"May potentially", "arguably", "it seems that", "to some extent" — hedges that name no specific doubt. Cut them. But a specific caveat ("not re-verified since 6 Oct", "vendor-reported") is information: keep it, once, next to the claim. If the same caveat repeats on every line, state it once at the top ("All competitor figures are vendor-reported").

## 4. Intensifiers and self-praise
"Genuinely", "truly", "real", "concrete", "crucial", "robust", "seamless", "significant", "notably", "a quiet positive", "great progress". They assert importance instead of showing it. The number shows it.

> Before: This is a genuinely significant improvement with real impact on reliability.
> After: Failed runs dropped from 4.1% to 0.6%.

## 5. Negation pivots and framing contrasts
"Not X, but Y", "X isn't just A — it's B", "This is a compliance choice, not a capability gap", "The question isn't whether…; it's…". Sometimes the contrast is the point (a common misreading being corrected). Usually it's rhythm. State Y.

## 6. Padding in threes
"Fast, reliable, and scalable." "Clear, concise, and actionable." Triplets where one word does the work or none carries a fact.

## 7. Process narration and meta
The document talking about itself or how it was made: "This edition ships as a refresh rather than a standard cycle", "Scores below are a reasoned synthesis…", "This run lands only 14 days after…". Keep a method caveat if it changes how much the reader should trust the numbers — in one short line.

## 8. Em-dash and parenthesis stacking
Clauses bolted on with — dashes — and (parentheses) until the main claim is buried. Split into sentences or drop the asides that carry no ledger item.

> Before: Uptime held — despite the region failover (which, notably, was unplanned) — at 99.95%, a strong result.
> After: Uptime held at 99.95% through an unplanned region failover.

## 9. Abstract nouns and jargon
"Leverage", "utilize", "landscape", "ecosystem", "surface" (as a noun), "posture", "calculus", "load-bearing", "stakeholders", "cohort", "alignment", "learnings". Name the actual thing: the people, the customers, the API, the decision.

## 10. Nominalizations and passive voice without an actor
"Perform an investigation of" → "investigate". "A decision was made to roll back" → "We rolled back" (or "Matt rolled back" — the actor is often a ledger item).

## 11. Decorative structure
Emoji section markers, a callout box per paragraph, bold on every bullet lead, H2s over two-line sections, a "TL;DR" that repeats the first paragraph, tables with one column of prose. Structure should help the reader find data. Keep tables of numbers, timelines, code, link lists.

## 12. Closers
"Let me know if you have questions", "Hope this helps", "In summary,", "Overall,", "Happy to discuss", "Next steps will be shared soon". Cut. If there are real next steps, they're ledger items — keep them as such.

## 13. Over-explaining the obvious
Defining terms the audience uses daily, explaining why a fix matters when the number already says it, restating the ticket description in the update. Write for the actual reader.

## Worked example

> **Before (112 words)**
> 🚀 **Exciting progress this week!** We're thrilled to share that the team has made significant strides on the export pipeline. After a thorough investigation, it was determined that the root cause of the timeouts was, in fact, an unindexed join on `orders.customer_id` — not, as initially suspected, network latency. A fix has been implemented (PR #482) and is currently being rolled out to staging. Importantly, early results look genuinely promising: p95 export time has dropped from 48 s to 6 s. Going forward, we'll continue to monitor closely and will share further updates as they become available. Overall, this represents a major step toward our Q4 reliability goals. Let us know if you have any questions!

> **After (35 words)**
> Export timeouts came from an unindexed join on `orders.customer_id`, not network latency. The fix (PR #482) is rolling out to staging; p95 export time dropped from 48 s to 6 s. Still monitoring.

Ledger check: cause, ruled-out hypothesis, PR #482, staging rollout, 48 s → 6 s, ongoing monitoring — all kept. "Q4 reliability goals" named no goal, so it went. Nothing was added: a rewrite never invents a date, owner or next step the source didn't state.
