# Mac Messages Codex Plugin

![Mac Messages Codex Plugin](docs/assets/mac-message-codex-plugin.png)

A Codex plugin for local Apple Messages on macOS. It lets Codex list chats,
read and search iMessage/SMS history, resolve phone numbers through local
Contacts data, prepare replies, and send messages or tapback reactions only
after explicit approval gates.

## Install In Codex

From GitHub:

```bash
codex plugin marketplace add KeystoneScience/mac-messages-codex-plugin
```

From a local checkout:

```bash
codex plugin marketplace add /path/to/mac-messages-codex-plugin
```

Then restart Codex, open the plugin picker, and install **Mac Messages Codex
Plugin** from the new marketplace. Grant Full Disk Access to Codex so the plugin can read
Messages.app history and local Contacts metadata.

## What Codex Gets

- `imsg_get_state`: check plugin health, binary discovery, Messages DB access,
  Contacts indexing, and send/react gates.
- `imsg_permissions_check`: explain required macOS permissions and open the
  relevant System Settings panes when requested.
- `imsg_list_chats`: list recent local Messages chats with participants and
  resolved display names.
- `imsg_get_chat`: inspect one direct or group chat by `chat_id`.
- `imsg_read_messages`: read bounded recent history for a chat.
- `imsg_search_messages`: search recent history across one chat or recent chats,
  including contact-resolved names.
- `imsg_resolve_contacts`: resolve phone numbers, emails, or Messages handles to
  local Contacts names.
- `imsg_sent_summary`: bulk-read sent messages across all chats for a day or
  date window, grouped by conversation with compact model-friendly defaults.
- `imsg_prepare_send`: inspect a proposed send payload and return the hash
  required before sending.
- `imsg_send_message`: send only when the payload hash, confirmation flag,
  approval note, and `ALLOW_IMSG_SEND=1` gate all match.
- `imsg_prepare_reaction`: inspect a standard tapback reaction against the latest
  incoming message and return the hash required before reacting.
- `imsg_send_reaction`: send a standard tapback only when the payload hash,
  confirmation flag, approval note, and `ALLOW_IMSG_REACT=1` gate all match.

## Safety Model

Read/search tools open local databases read-only. They do not modify
`~/Library/Messages/chat.db` or Contacts data.

Sending and tapback reactions are disabled by default. A send requires:

```text
imsg_prepare_send -> reviewed payload hash -> ALLOW_IMSG_SEND=1 -> imsg_send_message
```

A reaction requires:

```text
imsg_prepare_reaction -> reviewed payload hash -> ALLOW_IMSG_REACT=1 -> imsg_send_reaction
```

Both write paths also require a matching hash, an explicit confirmation boolean,
and an approval note. This keeps Codex useful for drafting and inspection while
making accidental outbound actions hard to trigger.

## Permissions

- Full Disk Access for Codex: required for local Messages history and Contacts
  name resolution.
- Messages.app signed in to iMessage and/or SMS relay.
- Automation permission for Messages.app: required only when sending messages or
  tapback reactions.
- Accessibility permission may be needed for tapbacks because Messages tapbacks
  use UI automation.

For SMS, enable Text Message Forwarding on your iPhone for this Mac.

## Useful Prompts

- "List my recent Messages chats and tell me what needs attention."
- "Search my local messages for Sabrina and summarize the latest thread."
- "Read the latest messages in chat 1292, but do not send anything."
- "Summarize everything I sent yesterday across all Messages chats."
- "Summarize what I sent on 2026-05-05, grouped by person."
- "Prepare a reply to this thread for me to approve."
- "React with a like to the latest incoming message in this chat."
- "Check whether the plugin has the permissions it needs."

## Local Verification

Build the underlying local helper:

```bash
make build
```

Run the Codex plugin doctor:

```bash
bash scripts/run_doctor.sh
```

Run the plugin tests:

```bash
python3 -m unittest discover -s Tests -v
```

Run the full Swift test suite:

```bash
make test
```

For fast cross-chat sent-message analytics, use the report path instead of
looping through chats:

```bash
imsg report --direction sent --start 2026-05-05T00:00:00Z --end 2026-05-06T00:00:00Z --no-text --json
```

## Notes

This standalone repo is intentionally documented as a Codex plugin first. The
bundled MCP server wraps the local `imsg` helper so Codex can use Messages.app
without needing direct database writes or private send APIs.
