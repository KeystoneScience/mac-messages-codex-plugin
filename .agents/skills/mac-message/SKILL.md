---
name: mac-message
description: Use for local Apple Messages archive reads, chat history, search, contact-resolved lookup, and explicitly requested sends.
---

# Mac Messages

Use this for Messages.app history, chat lookup, streaming, cross-chat sent-message reports, and sends. Reading is local DB access; sending uses Messages automation and must be explicitly requested. When the Codex plugin is installed, prefer the MCP tools first (`imsg_list_chats`, `imsg_read_messages`, `imsg_search_messages`, `imsg_sent_summary`, `imsg_prepare_send`, `imsg_send_message`, `imsg_prepare_reaction`, `imsg_send_reaction`) because they add bounded reads, bulk reporting, and write approval gates. For cross-chat questions like "what did I send yesterday", use `imsg_sent_summary` first; do not loop through chats unless the user asks to inspect specific conversations.

## Sources

- DB: `~/Library/Messages/chat.db`
- Repo: `~/Projects/mac-messages-codex-plugin` or the installed plugin checkout
- CLI: `imsg`
- JSON output is NDJSON; pipe to `jq -s` for arrays.
- Codex plugin: `.codex-plugin/plugin.json` plus `.mcp.json` runs `scripts/run_mcp.sh`

## Read Workflow

Check DB access:

```bash
sqlite3 ~/Library/Messages/chat.db 'pragma quick_check;'
```

List chats:

```bash
imsg chats --json | jq -s
```

Read a chat:

```bash
imsg history --chat-id ID --json | jq -s
```

Summarize a sent-message date window across chats:

```bash
imsg report --direction sent --start 2026-05-05T00:00:00Z --end 2026-05-06T00:00:00Z --json | jq -s
```

For counts/metadata only, add `--no-text` so the report skips attributed-body
and audio-transcription decoding:

```bash
imsg report --direction sent --start 2026-05-05T00:00:00Z --end 2026-05-06T00:00:00Z --no-text --json | jq -s
```

In MCP, prefer `imsg_sent_summary` with `preset: "yesterday"` or `day: "YYYY-MM-DD"`, `timezone`, and default `output_mode: "brief"`. Use `output_mode: "counts"` for analytics without message bodies, or `output_mode: "full"` only when exact message objects are needed.

Use `--attachments` when attachment metadata matters. Use `--start`/`--end` with absolute timestamps for date-scoped questions.

## Sends

Only send, react, mark read, or show typing when the user explicitly asks. Prefer dry wording in the final confirmation: recipient, service, and what was sent. In the Codex plugin, use `imsg_prepare_send` first, then send only with `ALLOW_IMSG_SEND=1`, `confirm_send=true`, an approval note, and the matching `send_sha256`.

Common send command:

```bash
imsg send --to "+15551234567" --text "message" --service auto
```

For tapbacks, use `imsg_prepare_reaction` first, then react only with `ALLOW_IMSG_REACT=1`, `confirm_react=true`, an approval note, and the matching `reaction_sha256`.

## Verification

For repo edits:

```bash
make test
make build
```

For live read proof:

```bash
imsg chats --limit 3 --json | jq -s
```
