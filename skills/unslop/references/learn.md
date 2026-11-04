# Build the writing profile

Goal: a `profile.md` precise enough that someone else could write a message the user would send without edits. `$U` = `~/.claude/unslop/`.

The profile captures voice, not mistakes: describe the user's habits, but don't make rewrites reproduce typos, missing apostrophes or second-language slips.

## 1. Collect

### Slack (primary)
1. `slack_read_user_profile` with no arguments → the user's ID and name. Never hardcode the ID.
2. `slack_search_public_and_private` with `filters: "from:<@ID> after:<start> before:<end>"`, `natural_language_query: ""`, `sort: "timestamp"`, `limit: 20`, `include_context: false`. Search stops paging after 20 pages (400 results), which a busy user fills in ~10 days. Results come newest-first, so even inside a window you only see its last few days. Cover the 90 days in 13 weekly windows and take up to 2 pages (≈40 messages) per week — otherwise the sample is just the last incident week.
3. Read each message and keep the ones the user wrote to colleagues about work — channels and work DMs alike. Drop:
   - personal and social chat (friends, family, plans, small talk), whatever language
   - link-only, emoji-only, one-word acks ("ok", "thx")
   - pasted logs, stack traces, code, SQL
   - anything that reads AI-drafted: headers, bold-lead bullet lists, em-dash chains, "Here's a summary". The user sometimes posts Claude drafts; learning from them teaches the profile slop.

   Judge by reading. Don't filter with keyword lists — they miss in every language they weren't written for.

### Claude Code prompts (always blended in)
```bash
python3 -I - <<'PY'
import json, os
rows = []
for line in open(os.path.expanduser("~/.claude/history.jsonl")):
    try: d = json.loads(line)
    except ValueError: continue
    t = (d.get("display") or "").strip()
    if not t or t.startswith("/"): continue          # slash commands
    if d.get("pastedContents") and len(t.split()) < 15: continue  # mostly a paste
    if len(t.split()) < 8: continue
    rows.append(t)
for t in rows[-300:]: print(t.replace("\n", " ")[:600]); print("---")
PY
```
These are written to an agent, not to people. Use them for vocabulary, sentence shape, punctuation, directness and tone. Don't learn greetings or how the user addresses colleagues from them.

## 2. Decide the tier
`usable` = kept Slack messages + kept prompts.
- `usable ≥ 50` → tier `learned`, continue.
- `usable < 50` → copy `references/default-profile.md` to `$U/profile.md` with tier `default`, carry over Corrections (step 5), tell the user why, stop.

## 3. Analyze
Read the samples and note, with rough counts where cheap:
- sentence length (typical, longest), fragments
- casing (lowercase starts? in which register?), punctuation (em dashes? semicolons? `->` arrows? `?` density), emoji
- person: I / we / impersonal; how they refer to the team and to customers
- openers and sign-offs, if any
- how asks are phrased ("can we…", imperative, "pls")
- how uncertainty is phrased ("not sure", "I think", "?")
- words and phrases they use often; words common in AI text that never appear
- structure: one-liners vs paragraphs, bullets, order (answer first?)
- languages present; if a second language has a real sample, note how the voice differs there
- register differences: channel posts vs DMs vs instructions to the agent

## 4. Write `$U/profile.md`

```markdown
---
built: YYYY-MM-DD
tier: learned
sources: slack <kept> of <seen>, claude-code prompts <kept>
---
# Voice
- <trait, with a short evidence note>
# Vocabulary
## Uses
## Never uses
# Structure
# Registers
- Slack/chat:
- Docs/reports:
- Status updates:
# Exemplars
<!-- 12–20 short verbatim lines (≤30 words), work-only, each showing a trait. No customer secrets, credentials, personal data, or anything you'd mind seeing in a screenshot. -->
- "<quote>"
# Corrections
<!-- Newest last. Each line: "- YYYY-MM-DD: rule". These override everything above. -->
```

For docs and reports — registers the samples barely cover — extrapolate from the chat voice: same directness and vocabulary, proper casing and full sentences.

## 5. Preserve Corrections
If `$U/profile.md` already existed, copy its `# Corrections` section into the new file verbatim. Those rules came from the user directly and outrank anything inferred.

## 6. Show the user
Print the Voice section and 5 exemplars, then ask whether it sounds like them. Every correction they give becomes a dated line under Corrections.
