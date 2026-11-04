# unslop

A Claude Code skill that rewrites AI-padded text so it is short, direct and sounds like you, without dropping a number, name, link or caveat.

## Install

```
/plugin marketplace add kapicic/unslop
/plugin install unslop@kapicic
```

## Use

```
/unslop <text | file | link>       # makes a copy next to the source
/unslop <text | file | link> -r    # backs up the original, then replaces it
/unslop learn                      # builds your writing profile
/unslop                            # rewrites the previous assistant message
```

Supported links: claude.ai artifacts, Notion pages, Linear issues, documents and project updates, Slack messages, GitHub PRs and issues, and any web page. Each one needs its connector installed. Slack and web pages can't be edited, so `-r` falls back to a copy for them. GitHub edits wait for your yes.

Anything else you add is treated as a steer, for example `/unslop <link> keep the tables`.

## Your profile

The first run builds `~/.claude/unslop/profile.md` from your Slack messages (if the Slack connector is installed) and your Claude Code prompts. With fewer than 50 usable samples it falls back to a generic laconic profile.

When you critique a rewrite ("I'd never say that", "keep the table"), unslop saves the rule under **Corrections** in your profile, and later runs follow it. Running `learn` again keeps your corrections.

Backups (`-r`) and copies of read-only sources go to `~/.claude/unslop/backups/` and `~/.claude/unslop/out/`. Nothing personal lives in the plugin itself.
