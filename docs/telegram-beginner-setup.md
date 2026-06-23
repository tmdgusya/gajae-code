---
title: Telegram beginner setup guide
description: Step-by-step GJC Telegram notification setup for first-time users.
---
# Telegram beginner setup guide

This guide is the slow, copy-paste-friendly path for setting up GJC Telegram notifications from nothing. It assumes you are comfortable using Telegram but have not configured a Telegram bot before.

## Read this first

GJC currently uses Telegram **forum topics** for real session delivery:

- `gjc notify setup` can discover and store a private Telegram DM chat id.
- Real session notifications and remote replies require a **Telegram supergroup with Topics enabled**.
- The bot must be an administrator in that group and must have **Manage Topics** permission.
- Do not expect private DM delivery to work for full GJC session traffic today.

If setup says it succeeded but no Telegram messages arrive, the most common cause is that `notifications.telegram.chatId` still points to your private DM instead of the Topics-enabled supergroup.

## What you will create

By the end, you will have:

1. GJC installed and working locally.
2. A Telegram bot created through BotFather.
3. A Telegram supergroup named however you prefer.
4. Topics enabled in that supergroup.
5. The bot added to the supergroup as an admin with Manage Topics.
6. GJC configured with the supergroup chat id, usually a value like `-1001234567890`.
7. A running `gjc daemon` process that can create one Telegram topic per GJC session.

## Screenshot plan for maintainers

The guide works without screenshots, but screenshots make it much easier for new users. Add screenshots under:

```text
docs/assets/telegram-beginner/
```

Recommended files:

| Slot | Suggested file | What the screenshot should show | Redaction |
| --- | --- | --- | --- |
| 1 | `01-botfather-newbot.png` | BotFather `/newbot` flow after the bot username is accepted. | Hide the full token. Leave only the first 4 characters visible if needed. |
| 2 | `02-group-created.png` | Telegram group info screen after creating the group. | Group name can remain visible. |
| 3 | `03-topics-toggle.png` | Group edit/settings screen with Topics enabled. | None required. |
| 4 | `04-add-bot-to-group.png` | Add Members screen showing the bot selected. | Hide unrelated contacts. |
| 5 | `05-bot-admin-permissions.png` | Bot administrator permissions with Manage Topics enabled. | None required. |
| 6 | `06-getupdates-chat-id.png` | Terminal output showing the `-100...` supergroup chat id. | Hide bot token and unrelated chat ids. |
| 7 | `07-notify-status.png` | `gjc notify status` showing `enabled: true` and the `-100...` chat id. | Token is masked by GJC already. |
| 8 | `08-session-topic-created.png` | Telegram group with a GJC-created session topic. | Hide private repo names if needed. |

When screenshots are added, place each one immediately after the matching step as standard Markdown image links, for example:

```md
![BotFather returns a new bot token](./assets/telegram-beginner/01-botfather-newbot.png)
```

Do not commit a screenshot containing a full BotFather token.

## Prerequisites

You need:

- A Telegram account.
- A terminal.
- Internet access.
- Permission to install packages on your machine.
- A machine supported by GJC:
  - macOS Apple Silicon,
  - Linux x64/arm64,
  - Windows x64,
  - or another platform where you install through Bun/source.

You do not need to know Telegram Bot API internals. The commands below print exactly what to look for.

## Step 1: Install Bun

GJC is distributed through the Bun package installer path. If you already have Bun, skip to Step 2.

### macOS or Linux

Run:

```sh
curl -fsSL https://bun.sh/install | bash
```

Close and reopen your terminal, then verify:

```sh
bun --version
```

Expected output is a version number, for example:

```text
1.2.20
```

### Windows PowerShell

Run:

```powershell
powershell -c "irm bun.sh/install.ps1|iex"
```

Close and reopen PowerShell, then verify:

```powershell
bun --version
```

If `bun` is not recognized, restart the terminal again. If it still fails, confirm that Bun's install directory is on `PATH`.

## Step 2: Install GJC

Run:

```sh
bun install -g gajae-code
```

Verify:

```sh
gjc --version
gjc --smoke-test
```

Expected result:

- `gjc --version` prints a version.
- `gjc --smoke-test` exits successfully.

If `gjc` is not recognized, restart your terminal. On Windows, Bun usually installs launchers under:

```text
%USERPROFILE%\.bun\bin
```

That directory must be on `PATH`.

## Step 3: Create a Telegram bot with BotFather

1. Open Telegram.
2. Search for `@BotFather`.
3. Open the verified BotFather chat.
4. Send:

```text
/newbot
```

5. BotFather asks for a display name. Example:

```text
My GJC Bot
```

6. BotFather asks for a username. It must end in `bot`. Example:

```text
my_gjc_helper_bot
```

7. BotFather replies with an HTTP API token that looks like:

```text
1234567890:AAExampleExampleExampleExampleExample
```

Copy that token and keep it private.

Important:

- Treat the token like a password.
- Do not paste it into screenshots, GitHub issues, Discord, Slack, or shared terminals.
- If you accidentally expose it, use BotFather to revoke/regenerate it.

Suggested screenshot slot: `01-botfather-newbot.png` after the token is redacted.

## Step 4: Confirm the bot can join groups

Most new bots can join groups by default. Confirm with BotFather:

1. In `@BotFather`, send:

```text
/mybots
```

2. Pick your bot.
3. Open **Bot Settings**.
4. Open **Allow Groups?**.
5. Make sure groups are allowed.

For receiving normal messages in groups, BotFather privacy may matter. GJC primarily needs commands, callbacks, and topic-routed messages, but disabling privacy reduces confusion during setup.

Optional but recommended:

1. In BotFather, send:

```text
/setprivacy
```

2. Pick your bot.
3. Choose **Disable**.

## Step 5: Create the Telegram group

GJC needs a group because it creates one topic per GJC session.

In Telegram:

1. Create a new group.
2. Add yourself.
3. Give the group a clear name, for example:

```text
GJC Remote Control
```

Suggested screenshot slot: `02-group-created.png`.

## Step 6: Enable Topics in the group

Topics are required.

In Telegram desktop or mobile:

1. Open the group.
2. Open group info/settings.
3. Choose **Edit** or the pencil icon.
4. Find **Topics**.
5. Turn **Topics** on.
6. Save.

After this, the group is a forum-enabled supergroup. The Telegram Bot API should report:

```text
is_forum: true
```

Suggested screenshot slot: `03-topics-toggle.png`.

If you cannot find the Topics switch:

- Make sure you are editing a group where you are owner/admin.
- Update Telegram Desktop or Telegram mobile.
- On some clients, the setting appears only after the group has been converted to a supergroup. Creating a new group and enabling Topics from Telegram Desktop is often the fastest fix.

## Step 7: Add the bot to the group

In the group:

1. Open group info/settings.
2. Choose **Add Members**.
3. Search for your bot username, for example:

```text
@my_gjc_helper_bot
```

4. Add the bot.

Suggested screenshot slot: `04-add-bot-to-group.png`.

If the bot does not appear in search:

- Open the bot DM once and press **Start**.
- Use a direct add link in your browser, replacing the username:

```text
https://t.me/my_gjc_helper_bot?startgroup=true
```

## Step 8: Make the bot an admin with Manage Topics

The bot must be able to create topics.

In the group:

1. Open group info/settings.
2. Open **Administrators**.
3. Add or edit your bot as an administrator.
4. Enable these permissions:
   - Send Messages
   - Manage Topics
5. Save.

Suggested screenshot slot: `05-bot-admin-permissions.png`.

The exact Telegram UI name may be:

- **Manage Topics**
- **Manage Forum Topics**
- **토픽 관리**

If the permission is missing, confirm Topics are enabled first. Telegram often hides Manage Topics until the group is a forum group.

## Step 9: Send a command in the group so Telegram exposes the chat id

In the group, send a command addressed to your bot:

```text
/start@my_gjc_helper_bot
```

Replace `my_gjc_helper_bot` with your bot username.

Use the full `@botusername` form. A plain message may be hidden from the bot when privacy mode is enabled.

## Step 10: Get the supergroup chat id

The chat id usually starts with `-100`. You need that exact value.

### macOS/Linux/Git Bash

Run this command in a terminal. It asks for the bot token without printing it:

```sh
read -s -p "Telegram bot token: " BOT_TOKEN; echo
node - <<'NODE'
const token = process.env.BOT_TOKEN;
if (!token) throw new Error('BOT_TOKEN is missing');
const res = await fetch(`https://api.telegram.org/bot${token}/getUpdates`, {
  method: 'POST',
  headers: {'content-type': 'application/json'},
  body: JSON.stringify({allowed_updates: ['message', 'channel_post', 'my_chat_member']})
});
const body = await res.json();
if (!body.ok) throw new Error(body.description || 'getUpdates failed');
const chats = new Map();
for (const update of body.result ?? []) {
  const chat = update.message?.chat ?? update.channel_post?.chat ?? update.my_chat_member?.chat;
  if (!chat) continue;
  chats.set(String(chat.id), {
    id: chat.id,
    type: chat.type,
    title: chat.title ?? chat.first_name ?? chat.username ?? '',
    isForum: chat.is_forum === true,
  });
}
if (chats.size === 0) {
  console.log('No chats found. Send /start@YourBotUsername in the Telegram group, then run this command again.');
} else {
  for (const chat of chats.values()) {
    console.log(`${chat.id}\t${chat.type}\tforum=${chat.isForum}\t${chat.title}`);
  }
}
NODE
unset BOT_TOKEN
```

Expected output contains a line like:

```text
-1001234567890    supergroup    forum=true    GJC Remote Control
```

Use the `-100...` value.

### Windows PowerShell

Run:

```powershell
$BOT_TOKEN_SECURE = Read-Host -AsSecureString "Telegram bot token"
$BSTR = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($BOT_TOKEN_SECURE)
$env:BOT_TOKEN = [Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)

@'
const token = process.env.BOT_TOKEN;
if (!token) throw new Error('BOT_TOKEN is missing');

fetch(`https://api.telegram.org/bot${token}/getUpdates`, {
  method: 'POST',
  headers: {'content-type': 'application/json'},
  body: JSON.stringify({allowed_updates: ['message', 'channel_post', 'my_chat_member']}),
})
  .then(r => r.json())
  .then(body => {
    if (!body.ok) throw new Error(body.description || 'getUpdates failed');
    const chats = new Map();
    for (const update of body.result ?? []) {
      const chat = update.message?.chat ?? update.channel_post?.chat ?? update.my_chat_member?.chat;
      if (chat) chats.set(String(chat.id), chat);
    }
    if (chats.size === 0) {
      console.log('No chats found. Send /start@YourBotUsername in the Telegram group, then run this command again.');
      return;
    }
    for (const chat of chats.values()) {
      console.log(`${chat.id}\t${chat.type}\tforum=${chat.is_forum === true}\t${chat.title ?? chat.first_name ?? chat.username ?? ''}`);
    }
  });
'@ | node

Remove-Item Env:BOT_TOKEN
```

Expected output contains a line like:

```text
-1001234567890    supergroup    forum=true    GJC Remote Control
```

Suggested screenshot slot: `06-getupdates-chat-id.png`. Redact unrelated chat ids.

## Step 11: Configure GJC with the supergroup chat id

Use `gjc notify setup` with both the token and the `-100...` chat id.

Important: because the chat id starts with `-`, pass it as `--chat-id=-100...` with an equals sign. Do not write `--chat-id -100...`.

```sh
gjc notify setup --token '<BOTFATHER_TOKEN>' --chat-id=-1001234567890
```

Expected output:

```text
Token validated. Message your bot now from the private Telegram chat to pair notifications.
Using provided chat id -1001234567890 (non-interactive).
Notifications enabled. botToken=1234…(len 46) chatId=-1001234567890
```

The first sentence still mentions private chat because the same setup command also supports private pairing. When `--chat-id` is provided, it uses the provided supergroup id and does not wait for a private chat.

## Step 12: Reload the Telegram daemon

Run:

```sh
gjc daemon reload telegram --force
```

Expected output:

```text
telegram reload: ok — spawned fresh telegram daemon (owner_spawned)
```

Then check status:

```sh
gjc daemon status telegram
gjc notify status
```

Expected output should show:

```text
telegram: running ...
Notifications
  enabled: true
  botToken: 1234…(len 46)
  chatId: -1001234567890
  redact: false
```

Suggested screenshot slot: `07-notify-status.png`.

## Step 13: Verify Telegram permissions from the terminal

This command reads the token and chat id from your GJC config and checks the actual Telegram state.

```sh
node - <<'NODE'
const fs = require('node:fs');
const cfg = fs.readFileSync(`${process.env.HOME}/.gjc/agent/config.yml`, 'utf8');
const token = cfg.match(/botToken:\s*([^\n]+)/)?.[1]?.trim().replace(/^["']|["']$/g, '');
const chatId = cfg.match(/chatId:\s*"?([^"\n]+)"?/)?.[1]?.trim();
if (!token || !chatId) throw new Error('Missing notifications.telegram.botToken or chatId in ~/.gjc/agent/config.yml');
async function call(method, body = {}) {
  const res = await fetch(`https://api.telegram.org/bot${token}/${method}`, {
    method: 'POST',
    headers: {'content-type': 'application/json'},
    body: JSON.stringify(body),
  });
  const json = await res.json();
  if (!json.ok) throw new Error(`${method}: ${json.description || res.statusText}`);
  return json.result;
}
(async () => {
  const me = await call('getMe');
  const chat = await call('getChat', {chat_id: chatId});
  const member = await call('getChatMember', {chat_id: chatId, user_id: me.id});
  console.log(`bot=@${me.username}`);
  console.log(`chat=${chat.id} ${chat.type} ${chat.title ?? ''}`);
  console.log(`is_forum=${chat.is_forum === true}`);
  console.log(`bot_status=${member.status}`);
  console.log(`can_manage_topics=${member.can_manage_topics === true}`);
})();
NODE
```

You want:

```text
is_forum=true
bot_status=administrator
can_manage_topics=true
```

If any of those are false, fix the Telegram group settings before continuing.

## Step 14: Optional hard smoke test: create and delete a test topic

This proves Telegram will let the bot create session topics.

```sh
node - <<'NODE'
const fs = require('node:fs');
const cfg = fs.readFileSync(`${process.env.HOME}/.gjc/agent/config.yml`, 'utf8');
const token = cfg.match(/botToken:\s*([^\n]+)/)?.[1]?.trim().replace(/^["']|["']$/g, '');
const chatId = cfg.match(/chatId:\s*"?([^"\n]+)"?/)?.[1]?.trim();
if (!token || !chatId) throw new Error('Missing notifications.telegram.botToken or chatId in ~/.gjc/agent/config.yml');
async function call(method, body = {}) {
  const res = await fetch(`https://api.telegram.org/bot${token}/${method}`, {
    method: 'POST',
    headers: {'content-type': 'application/json'},
    body: JSON.stringify(body),
  });
  const json = await res.json();
  if (!json.ok) throw new Error(`${method}: ${json.description || res.statusText}`);
  return json.result;
}
(async () => {
  const topic = await call('createForumTopic', {chat_id: chatId, name: 'GJC setup test'});
  console.log(`created_thread=${topic.message_thread_id}`);
  if (topic.message_thread_id !== undefined) {
    await call('deleteForumTopic', {chat_id: chatId, message_thread_id: topic.message_thread_id});
    console.log('deleted_test_topic=true');
  }
})();
NODE
```

Expected output:

```text
created_thread=7
deleted_test_topic=true
```

The number may differ.

## Step 15: Start or reuse GJC

From the repository where you want to use GJC, start normally:

```sh
gjc
```

or:

```sh
gjc --tmux
```

When notifications are active, a session publishes a local endpoint under:

```text
.gjc/state/notifications/<sessionId>.json
```

The Telegram daemon scans those endpoints and creates a topic in your configured group.

Suggested screenshot slot: `08-session-topic-created.png` once the first GJC session topic appears.

## How to talk back to GJC from Telegram

Use the topic that GJC created for the session.

Do:

- Open the configured Telegram group.
- Open the GJC-created session topic.
- Tap inline buttons on ask prompts.
- Or type a message in that same session topic.

Do not:

- Send messages to the bot DM and expect the current GJC session to receive them.
- Send messages in the group's General topic and expect routing to a session.
- Use a different group than the configured `chatId`.

When inbound messages are accepted, the daemon may set Telegram reactions:

```text
eyes reaction = queued by the daemon
check-mark reaction = consumed by the GJC session
```

## Common failure cases

### `gjc notify setup` succeeds, but Telegram stays silent

Check:

```sh
gjc notify status
```

If `chatId` is a positive number like `6352039836`, it is probably your private DM. Replace it with the supergroup id:

```sh
gjc notify setup --token '<BOTFATHER_TOKEN>' --chat-id=-1001234567890
gjc daemon reload telegram --force
```

### `Option '--chat-id' argument is ambiguous`

This happens when the chat id starts with `-` and is passed as a separate argument.

Wrong:

```sh
gjc notify setup --token '<TOKEN>' --chat-id -1001234567890
```

Right:

```sh
gjc notify setup --token '<TOKEN>' --chat-id=-1001234567890
```

### `is_forum=false`

Topics are not enabled. Open the Telegram group settings and enable Topics.

### `can_manage_topics=false`

The bot is an administrator but lacks Manage Topics. Edit the bot administrator permissions and enable Manage Topics.

### The bot does not see the group in `getUpdates`

Do all of these:

1. Add the bot to the group.
2. Send a command addressed to the bot:

```text
/start@YourBotUsername
```

3. Run the `getUpdates` command again.

If it still does not appear, disable bot privacy in BotFather with `/setprivacy`, then send the command again.

### Telegram 409 conflict

Only one `getUpdates` poller can own a bot token. Stop old manual bridges or stale bot scripts, then run:

```sh
gjc daemon reload telegram --force
```

### The daemon is stale

Check:

```sh
gjc daemon status telegram
```

If it says `stale`, reload:

```sh
gjc daemon reload telegram --force
```

### You exposed the token by accident

Regenerate the token in BotFather:

1. Send `/mybots` to BotFather.
2. Pick your bot.
3. Open **API Token**.
4. Revoke/regenerate the token.
5. Reconfigure GJC:

```sh
gjc notify setup --token '<NEW_BOTFATHER_TOKEN>' --chat-id=-1001234567890
gjc daemon reload telegram --force
```

## Final checklist

Before declaring setup complete, all of these should be true:

```text
[ ] gjc --smoke-test succeeds
[ ] Telegram bot exists
[ ] Bot can join groups
[ ] A Telegram supergroup exists
[ ] Topics are enabled in that group
[ ] Bot is administrator in that group
[ ] Bot has Manage Topics permission
[ ] getUpdates showed a -100... supergroup chat id
[ ] gjc notify status shows chatId: -100...
[ ] gjc daemon status telegram shows running
[ ] Verification command shows is_forum=true
[ ] Verification command shows can_manage_topics=true
[ ] Optional create/delete topic smoke test succeeds
[ ] A new GJC session creates a topic in the group
```

After this, use the GJC-created Telegram topic for that session. That topic is the remote conversation surface.
