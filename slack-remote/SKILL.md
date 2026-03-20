---
name: slack-remote
description: >
  Remote access to Cowork via Slack. This skill lets you control Cowork from your phone by
  combining two capabilities: (1) mirroring ongoing tasks to a Slack channel thread, and
  (2) picking up new tasks posted in that same channel. Trigger this skill when you say any of:
  "turn on cowork remote access", "enable remote access", "I'm going out", "I'm stepping away",
  "I'm heading out", "going mobile", "turn off cowork remote access", "disable remote access",
  "I'm back", "check my Slack inbox", "any new tasks in Slack?", "process my Slack queue",
  "mirror to Slack", "keep me posted on Slack", "send updates to Slack", "Slack me", or
  "notify me on Slack". Always use this skill whenever the user mentions remote access, Slack
  inbox, Slack tasks, going mobile, stepping away, mirroring to Slack, or coming back — even
  if they don't use the exact phrases above.
---

# Slack Remote Access

One skill to control Cowork from your phone. When enabled, it does two things:

1. **Mirrors ongoing tasks** — any in-progress Cowork work gets a thread in your designated
   Slack channel with progress updates, authorization requests, and a completion summary
2. **Picks up new tasks** — messages you post in the channel are automatically picked up and
   executed, with all updates happening in-thread

Everything flows through one Slack channel. Your phone becomes a remote Cowork terminal.

## Setup — REQUIRED before first use

Before this skill can work, you need to configure three things:

1. **Slack channel**: Create a dedicated Slack channel (e.g. `#my-cowork-remote`) or pick an
   existing one. Find the channel ID by right-clicking the channel name in Slack → "View
   channel details" → the ID is at the bottom of the details panel. Replace
   `YOUR_SLACK_CHANNEL_ID` everywhere in this file with that ID.

2. **Your Slack user ID**: Click your profile picture in Slack → "Profile" → the three dots
   menu → "Copy member ID". Replace `YOUR_SLACK_USER_ID` everywhere in this file with that ID.

3. **Scheduled task**: Create a scheduled task in Cowork called `slack-inbox-poll` that runs
   every minute. The task prompt should be:
   ```
   Run the Slack Remote skill's polling workflow: read channel YOUR_SLACK_CHANNEL_ID,
   filter for new messages from user YOUR_SLACK_USER_ID, and process any new task requests.
   ```
   The task should be **disabled by default** — it gets toggled on/off when you enable/disable
   remote access.

## Key information

- **Channel ID:** `YOUR_SLACK_CHANNEL_ID`
- **Your user ID:** `YOUR_SLACK_USER_ID`
- **Scheduled task ID:** `slack-inbox-poll`
- **State file:** `/tmp/slack-inbox-last-processed.txt` — stores the timestamp of the last
  processed message to avoid re-processing

---

## Toggling remote access (HANDLE THIS FIRST)

The scheduled polling task (`slack-inbox-poll`) is **off by default**. You explicitly turn it
on when stepping away and turn it off when you're back.

### Enabling remote access

Trigger: "turn on cowork remote access", "enable remote access", "I'm heading out",
"going mobile", "I'm stepping away", etc.

Do ALL of the following:

1. **Enable the polling task.** Use `update_scheduled_task` to set `enabled: true` on
   task ID `slack-inbox-poll`.

2. **Mirror any ongoing work.** If you are currently working on a task (or have recently
   completed one in this session), create a thread in the Slack channel to mirror it:
   - Send a message to channel `YOUR_SLACK_CHANNEL_ID` summarizing the current task status
   - Save the returned `ts` as `thread_ts` for subsequent updates
   - From this point on, post all progress updates, authorization requests, and the
     final summary as thread replies in this channel

3. **Do one immediate poll** of the channel (follow the Polling Workflow below) so any
   messages already waiting get picked up right away.

4. **Confirm to the user:** "Remote access is on. I'll mirror any current work and watch
   the channel for new tasks. You can head out."

### Disabling remote access

Trigger: "turn off cowork remote access", "disable remote access", "I'm back",
"stop watching Slack", etc.

1. Use `update_scheduled_task` to set `enabled: false` on task ID `slack-inbox-poll`.
2. Confirm: "Remote access is off. Welcome back."

### One-time inbox check

Trigger: "check my Slack inbox", "any new tasks?", "process my Slack queue"

Do NOT enable/disable the scheduled task — just run the Polling Workflow below once.

---

## Mirroring Workflow

When remote access is enabled and you're actively working on a task, mirror your progress
to the Slack channel.

### Starting a mirror thread

Send a message to channel `YOUR_SLACK_CHANNEL_ID` that serves as the **thread parent**:

- A brief title (e.g. "*Cowork Task: Generating Q1 Revenue Report*")
- A 1-2 sentence summary of what you're doing
- An estimate of how long it might take

Save the returned `ts` — use it as `thread_ts` for all subsequent thread replies.

### Progress updates

Post **thread replies** at meaningful milestones. Don't flood — aim for updates a person
glancing at their phone would find useful:

- When you finish a major phase
- When something unexpected happens
- When you're about to start a long sub-task
- When a sub-task completes with a notable result

Keep messages concise — 1-3 sentences. Aim for 3-6 updates per task, not 15.

### Authorization requests

When you need the user's approval for something sensitive:

1. Post in the thread:
   ```
   *Authorization needed:*
   [Clear description of what you want to do and why]
   Reply *yes* to approve, or tell me what to change.
   ```

2. Poll for response using `slack_read_thread`:
   - First check: ~30 seconds after sending
   - Subsequent checks: every ~45-60 seconds
   - After 5 minutes: send a gentle nudge
   - After 10 minutes: pause and move on to anything you can do without authorization

3. Interpret casual replies generously: "yes", "yep", "go ahead", "do it", thumbs up
   all mean approval.

### Wrapping up

Post a final thread reply summarizing what was accomplished:
```
*Done!*
[Brief summary of results and where to find any files]
```

---

## Polling Workflow

This is how new tasks from Slack get picked up. The scheduled task runs this every minute
when remote access is enabled.

### Step 1: Read the channel

Use `slack_read_channel` with channel ID `YOUR_SLACK_CHANNEL_ID`, limit 10.

### Step 2: Filter for new messages from the user

1. Read the last-processed timestamp from `/tmp/slack-inbox-last-processed.txt`.
   If the file doesn't exist, only process the most recent message (to avoid a backlog
   storm on first run).

2. Keep only messages that:
   - Are from the user (user ID `YOUR_SLACK_USER_ID`)
   - Are newer than the last-processed timestamp
   - Are top-level messages (not thread replies — ignore messages with a `thread_ts`
     that differs from their own `ts`)
   - Are not bot messages or system messages

3. If no new messages match, exit quietly. Don't send anything.

### Step 3: Process each task (oldest first)

For each new message:

1. **Acknowledge in-thread.** Reply using the message's `ts` as `thread_ts`:
   "*Got it — working on this now.* I'll post updates here as I go."

2. **Execute the task.** Treat the message text as a full Cowork prompt. Use all
   available tools and connectors.

3. **Post progress updates as thread replies.** Same guidelines as the mirroring
   workflow — concise, meaningful milestones only.

4. **Handle authorizations in-thread.** Same flow as the mirroring workflow.

5. **Wrap up in-thread** with a summary.

6. **Update the state file.** Write the message's `ts` to
   `/tmp/slack-inbox-last-processed.txt`.

---

## Important guidelines

- **Only process messages from the configured user** (`YOUR_SLACK_USER_ID`). Ignore all others.
- **One channel for everything.** All threads — both mirrored tasks and new tasks from
  Slack — go in the same channel. This keeps the phone experience simple.
- **Thread discipline.** Every update goes as a thread reply. Never send standalone messages
  to the channel mid-task.
- **Don't flood.** 3-6 updates per task is the sweet spot.
- **Be honest about limitations.** If you can't do something, say so in the thread.
- **File outputs go to Cowork's output folder.** Mention the location in the thread so
  the user knows to check when they're back.
- **Errors get reported in-thread.** No silent failures.
- **Keep the state file updated.** Critical for avoiding re-processing.
