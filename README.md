# JARVIS Ultimate

A modular Discord security and moderation bot starter using Python 3.11+, `discord.py` 2.x, and SQLite via `aiosqlite`.

## Implemented in this build

- Modular startup and extension loading.
- SQLite persistence for settings, moderation cases, security events, Join-to-Create records, XP/invite tables, and structural backup snapshots.
- `/help` with a private interactive panel.
- `/jarvis` operational dashboard with live process metrics.
- `/ping` latency diagnostics for Discord gateway heartbeat, initial interaction acknowledgement, and follow-up HTTP timing in logs.
- Performance tuning: cached hot-path guild settings, SQLite WAL with NORMAL synchronous mode, in-memory temp storage, and a short busy timeout.
- `/help_config_creator`, `/help_config_introduction`, `/help_config_show` configuration commands.
- `/ban`, `/kick`, `/timeout`, `/untimeout`, `/warn`, and `/warnings`. `/timeout` duration is configured in seconds (1–50,000).
- `/security threshold` accepts a monitoring window of 1–50,000 seconds.
- Central moderation hierarchy checks, best-effort owner/member DMs, and stored moderation history.
- `/security status`, `/security threshold`, `/security log-channel`, `/security lockdown`, and `/security unlock`.
- Channel-deletion detection with an explicit unknown actor until verified.
- `/setup-logs` creates a private log category and security channel.
- Welcome DM on join with a configurable template stored in guild settings.
- Join-to-Create setup and status commands plus basic temporary voice channel creation.
- `/backup create`, `/backup list`, and `/backup inspect` for supported structural snapshots.
- Basic database tests.

## Not yet implemented (do not assume otherwise)

The project is a functional starting bot, not yet the complete specification. Destructive backup restore, complete anti-nuke/anti-raid decision engine, full event logging matrix, timeout-expiration notification worker, farewell DMs, autoroles/reaction roles, leveling, invite attribution, `/fly`, and full JTC owner controls/empty-channel cleanup are not implemented. `/build`, `/setup-logs verify/repair`, and complete moderation command coverage are also not implemented. The help text only lists commands present in this build.

A channel-deletion listener detects the event after it occurs. It does not guarantee pre-emptive prevention, and it does not infer the actor without additional evidence.

## Setup

1. Install Python 3.11.
2. Create a Discord application and bot in the [Discord Developer Portal](https://discord.com/developers/applications).
3. Enable the **Server Members Intent** under Bot settings. Message Content Intent is not needed for the current implemented features; keep it disabled unless later features require it.
4. Invite the bot with only the permissions needed for the features you use. Moderation requires Ban Members, Kick Members, Moderate Members; JTC requires Manage Channels and Move Members; log setup requires Manage Channels; lockdown requires Manage Channels; backup snapshot commands need access to guild structure. Avoid Administrator unless you consciously accept the larger blast radius.
5. Copy `.env.example` to `.env` and set `DISCORD_TOKEN`.
6. Install dependencies: `python -m pip install -r requirements.txt`
7. Run: `python main.py`
8. Invite the bot to a test server first. Slash-command global synchronization may take time to propagate.

## Local tests

```bash
python -m unittest discover -s tests -v
```

Tests exercise database persistence only; they do not prove live Discord API behavior.

## Railway

- Deploy this repository as a worker/service.
- Set `DISCORD_TOKEN` and other environment variables in Railway Variables.
- Attach a persistent volume and set `DATABASE_PATH` to a path on that volume, for example `/data/jarvis.sqlite3`.
- Without persistent storage, SQLite data may be lost on redeploy/restart.
- Use one running replica for this SQLite-based build. Multiple replicas sharing a SQLite database are not a safe default.
- Test the Southeast Asia (Singapore) region first from India, then compare alternatives only if needed. A region change with a mounted volume can require migration and downtime.
- Keep `SYNC_COMMANDS_ON_START=true` for the first deployment so slash commands register; after successful sync, you can set it to `false` to avoid doing a global sync at every restart. Set it back to `true` when you change commands.
- `/ping` is a diagnostic, not a promise of 80 ms. Discord gateway latency and REST/action latency are different; the actual result depends on Discord and network routing.

## Replit

- Store `DISCORD_TOKEN` in Replit Secrets, not in source code.
- Confirm the workspace/runtime provides persistent storage for the chosen database path. Ephemeral storage can lose SQLite data.
- Run `python main.py` as a long-running process if your plan supports it.

## Time-based command ranges

- `/timeout`: `duration_seconds` accepts **1–50,000 seconds** (maximum 13 hours, 53 minutes, 20 seconds). Discord may enforce its own validation and API behaviour.
- `/security threshold`: `window_seconds` accepts **1–50,000 seconds**.
- Internal UI view timeouts are implementation details, not user-configurable command durations.

## Important Discord API limits

- The bot cannot intercept every action before Discord processes it.
- Audit-log attribution may be delayed or unavailable.
- Deleted message content cannot generally be fetched after deletion unless previously stored in a compliant archive.
- DMs can fail due to recipient privacy settings or API restrictions.
- Rate limits and network delays mean no guaranteed sub-20 ms end-to-end response.
- Restore operations can only reproduce supported data that was saved and that the API permits the bot to recreate.
