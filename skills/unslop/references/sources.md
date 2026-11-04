# Sources: detect → fetch → copy → replace

`$U` = `~/.claude/unslop/`. `<slug>` = lowercase title, hyphens, max 6 words. `<stamp>` = `YYYY-MM-DD-HHMM`.

Every `-r` starts with a backup: `$U/backups/<stamp>-<slug>.<ext>`, line 1 = `<!-- source: <url or path> -->` (or `# source:` for non-HTML), then the original content exactly as fetched.

Read-only sources write their copy to `$U/out/<YYYY-MM-DD>-<slug>.md` and also print it in chat. The copy is paste-ready: no source line or other added text — the filename and the report already say where it came from.

## Quick table

| Input | Fetch | Copy (default) | `-r` |
|---|---|---|---|
| raw text | as given | chat | not applicable — just the rewrite |
| previous message (no input) | conversation | chat | not applicable |
| local path | `Read` | `<dir>/<stem>.unslop<ext>` | backup → overwrite |
| `claude.ai/artifact/…`, `claude.ai/code/artifact/…` | `Artifact` read | new private artifact | backup → publish with `url` |
| `notion.so`, `notion.site`, `app.notion.com` | `notion-fetch` | sibling page "<title> (unslop)" | backup → `notion-update-page` replace_content (guarded) |
| Linear document `linear.app/<ws>/document/…` | `get_document` | `save_document` new, same parent | backup → `save_document` with `id` |
| Linear issue `linear.app/<ws>/issue/KEY-123/…` | `get_issue` | `$U/out/` + chat | backup → `save_issue` `description` |
| Linear project update `…/project/<slug>/…#project-update-<id8>` | `get_status_updates` | `$U/out/` + chat | backup → `save_status_update` `body` |
| Slack `…slack.com/archives/<C>/p<ts>` | `slack_read_thread` | `$U/out/` + chat | unsupported |
| GitHub PR/issue | `gh pr view` / `gh issue view` | `$U/out/` + chat | backup → show → wait for yes → `gh pr edit` |
| Google Doc | needs the Google Docs connector | `$U/out/` + chat | unsupported |
| any other URL | `WebFetch` (or Chrome `get_page_text`) | `$U/out/` + chat | unsupported |

## Details and traps

### Local file
Copy goes next to the original: `notes.md` → `notes.unslop.md`, `page.html` → `page.unslop.html`. Never touch other files in the directory.

### claude.ai artifact
- `Artifact` action `read` with the `url`. The header says whether the user is a `writer` and may name a saved file holding the full page — build from that file when it exists, otherwise from the returned HTML.
- If the read shows a Claude Docs document rather than an HTML page, use the Claude Docs connector (`read`, then `batch` create for a copy or `update` for `-r`) and keep its block structure.
- Write the rewritten HTML to `$U/out/<YYYY-MM-DD>-<slug>.html`.
- Copy: `Artifact` publish with that `file_path`, no `url`, `icon` = a generic word matching the original (e.g. `document`, `incident`, `chart`), `<title>` = original title + " · unslop", `description` = "Unslopped copy of <original url>". The new artifact is private.
- `-r`: requires `writer`. Backup the original HTML, then publish with `url` = the original and the rewritten file. If not a writer, say so and make the copy instead.

### Notion
- `notion-fetch` the page. Note `<ancestor-path>` / parent, `truncated`, `unknown_block_ids`.
- Before writing Notion Markdown, read `notion://docs/notion-flavored-markdown-spec` once (via `notion-fetch` with that URI) — table, toggle and callout syntax is strict.
- Copy: `notion-create-pages` with the same parent (`page_id`, or `data_source_id` when the page sits in a database — then fill required properties from the source). If the source is a top-level page with no parent, use `creation_mode: "draft"` (private) and say so. Title: "<title> (unslop)".
- `-r`: refuse and make the copy instead if the page contains child pages (`<page url=…>`), databases (`<database …>`), synced blocks, is `truncated`, or has `unknown_block_ids` — `replace_content` would delete or mangle what you can't see. Otherwise backup the fetched Markdown, then `notion-update-page` with `command: "replace_content"` and `new_str`. Never set `allow_deleting_content: true`.

### Linear
- Issue: `get_issue` with the identifier from the URL. Rewrite the description only — comments belong to their authors.
- Document: `get_document` with the slug/ID from the URL. Copy: `save_document` with `title` "<title> (unslop)" and the same parent (`project`, `issue`, `initiative` or `team`) as the source.
- Project update: the URL fragment `#project-update-<id8>` holds the first 8 hex chars of the update id. `get_status_updates` with `type: "project"` and `project` = the slug from the URL, find the update whose `id` starts with `<id8>`. `-r`: `save_status_update` with `id`, `type: "project"`, `body`; leave `health` unchanged.
- Linear Markdown: use `@displayName` for mentions, literal newlines.

### Slack
- Message link `https://<ws>.slack.com/archives/<channel>/p<16 digits>`: ts = first 10 digits + "." + last 6. If the link has `?thread_ts=…`, that is the parent.
- `slack_read_thread` with `channel_id` and `message_ts`. Rewrite only the linked message unless the user asked for the thread.
- No tool edits a Slack message, and posting needs the user's explicit go-ahead. `-r` → "Slack messages can't be edited from here" + the copy.

### GitHub
- `gh pr view <url> --json title,body` or `gh issue view <url> --json title,body`.
- `-r`: backup, show the rewrite, wait for an explicit yes. Then write the body to `$U/out/` and `gh pr edit <url> --body-file <file>` (or `gh issue edit`). A PR someone else opened still needs the yes — it lands under the user's name.

### Other URLs
- `WebFetch` with prompt: "Return the page's full main text verbatim as Markdown — every heading, paragraph, list, table, number and link. Do not summarize." WebFetch runs a small model over the page and can still drop detail; for long pages or pages behind a login, use the Chrome extension's `get_page_text` instead.
- Google Docs: the connector needs authentication. Ask the user to authenticate it or paste the text; a published-to-web doc works through `WebFetch`.
