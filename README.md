# Cowork Slack Remote

A single skill that turns Slack into a remote control for [Claude Cowork](https://claude.ai).

**The problem:** When Cowork is running a long task on your machine and you need to step away, you lose visibility. You can't check progress, approve actions, or start new tasks from your phone.

**The solution:** This skill bridges Cowork and Slack so you can monitor, authorize, and even initiate tasks entirely from your phone.

## What it does

Tell Cowork "I'm heading out" to enable remote access. From that point:

- **Mirrors ongoing tasks** — any in-progress work gets a thread in your Slack channel with real-time progress updates
- **Authorization via Slack** — reply "yes" from your phone to approve sensitive actions
- **Remote tasking** — post a message in the channel and Cowork picks it up within 60 seconds
- **On/off toggle** — say "I'm back" to disable. No notification spam when you're at your desk

Tell Cowork "I'm back" to disable. Clean on, clean off.

## Installation

### As a Cowork Skill

1. Download the `.skill` file from the [Releases](../../releases) page
2. Double-click to install in Cowork

### Manual

1. Clone this repo
2. Copy the `slack-remote/` folder to your Cowork skills directory

## Setup

Before using the skill, you need to configure it with your own Slack details:

1. **Create a Slack channel** (e.g., `#my-cowork-remote`) or pick an existing one
2. **Find your channel ID**: Right-click the channel → "View channel details" → ID is at the bottom
3. **Find your Slack user ID**: Click your profile → three dots menu → "Copy member ID"
4. **Update the skill**: Open `slack-remote/SKILL.md` and replace:
   - `YOUR_SLACK_CHANNEL_ID` with your channel ID
   - `YOUR_SLACK_USER_ID` with your user ID
5. **Create a scheduled task** in Cowork called `slack-inbox-poll` (see the SKILL.md for the full prompt). It should be disabled by default — the skill toggles it on/off automatically.

## Requirements

- [Claude Cowork](https://claude.ai) with the Slack connector enabled
- A Slack workspace where Cowork can send messages

## License

MIT
