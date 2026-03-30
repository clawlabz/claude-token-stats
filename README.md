# token-stats

A Claude Code slash command (`/token-stats`) that shows your AI token usage, cache hit rates, and cost across **Claude Code, OpenClaw, and Codex CLI** — all from local session files, no API calls required.

## Data sources

| Source | Location | Cost display |
|--------|----------|-------------|
| Claude Code | `~/.claude/projects/` | `~$` estimated |
| OpenClaw | `~/.openclaw/agents/*/sessions/` | `$` stored (Anthropic) / `$0` (other models) |
| QClaw | `~/.qclaw/agents/*/sessions/` | same as OpenClaw |
| Codex CLI | `~/.codex/sessions/` | `~$` estimated |

`$` = cost stored in log file. `~$` = calculated from pricing table.

## What it shows

**Daily summary** (default):

```
Date         Source             Sess  TotalInput    Output  CacheHit%     Total          Cost  Models
──────────────────────────────────────────────────────────────────────────────────────────────────────
2026-03-30   claude               12      108.9M    297.7K      97.7%    109.2M  ~$  45.78  Sonnet4.6
2026-03-30   openclaw/main        37       20.1M    815.8K      56.5%     20.9M   $  48.45  Sonnet4.6
2026-03-30   openclaw/ifig       111        5.7M     18.6K      93.0%      5.7M  ~$   3.35  Sonnet4.6
2026-03-30   codex                 3        7.5M     55.5K      40.8%      7.5M  ~$   0.00  GPT5.4
2026-03-30     ALL               163      142.2M      1.2M      88.7%    143.4M  ~$  97.58  Sonnet4.6+GPT5.4
──────────────────────────────────────────────────────────────────────────────────────────────────────
TOTAL                             163      142.2M      1.2M      88.7%    143.4M  ~$  97.58

  TotalInput = fresh(12.1M) + cache_write(20.0M) + cache_read(110.1M)
  Cost legend:  $ = stored in log   ~$ = estimated from pricing table
```

**Session breakdown** (per day):

```
Time    Source                 Project           Msgs  TotalInput    Output  CacheHit%          Cost  Topic
──────────────────────────────────────────────────────────────────────────────────────────────────────────────
02:17   claude                 work/claw          441       44.8M     99.2K      98.0%   ~$  17.96  整体迁移完毕了吗
08:24   openclaw/main          clawd              231       20.1M    815.8K      56.5%    $  48.45  ClawHealth审查
```

**Model breakdown**:

```
Short           Model ID                                  TotalInput    Output     Total          Cost
──────────────────────────────────────────────────────────────────────────────────────────────────────────────
Opus4.6         claude-opus-4-6                               435.2M    464.5K    435.7M  ~$ 160.56
Sonnet4.6       claude-sonnet-4-6                             176.8M      1.3M    178.0M  ~$ 117.07
Haiku4.5        claude-haiku-4-5-20251001                       3.1M     40.2K      3.2M   $   0.89
GPT5.4          gpt-5.4                                       853.1M      1.6M    854.8M  ~$   0.00
```

Column reference:
- **TotalInput** = `input + cache_write + cache_read` — all tokens the model processed on input
- **CacheHit%** = `cache_read / TotalInput` — higher = more cache reuse (much cheaper per token)
- **Total** = TotalInput + Output
- **Cost** — `$` stored natively in log; `~$` estimated at API list prices (not actual subscription billing)

## Installation

```bash
curl -fsSL https://raw.githubusercontent.com/clawlabz/token-stats/main/install.sh | bash
```

Restart Claude Code — `/token-stats` is ready.

> The command also **self-heals**: if the script is ever missing, it re-downloads automatically on the next `/token-stats` run.

## Usage

Inside Claude Code:

```
/token-stats                               # 30-day daily summary + model breakdown (all sources)
/token-stats 2026-03-30                    # session breakdown for a specific date
/token-stats --days 7                      # last 7 days
/token-stats --project claw               # filter by project/cwd keyword
/token-stats --source claude              # only Claude Code
/token-stats --source openclaw            # only OpenClaw
/token-stats --source codex               # only Codex CLI
/token-stats --source claude --source openclaw  # multiple sources
/token-stats --models                     # model breakdown only
```

Or directly from the terminal:

```bash
python3 ~/.claude/scripts/token-stats.py
python3 ~/.claude/scripts/token-stats.py --days 7
python3 ~/.claude/scripts/token-stats.py --source openclaw --days 3
python3 ~/.claude/scripts/token-stats.py 2026-03-30
```

## Requirements

- Python 3.8+ (stdlib only, no extra packages)
- One or more of: Claude Code, OpenClaw, Codex CLI

## Pricing used for cost estimates

Claude (Anthropic API list prices):

| Token type   | Price per million |
|--------------|-------------------|
| Input        | $3.00             |
| Output       | $15.00            |
| Cache write  | $3.75             |
| Cache read   | $0.30             |

OpenClaw Anthropic sessions: uses the **stored cost** from the log file (exact, not estimated).

Codex CLI / OpenAI models: pricing varies by model and service. Currently shown as `~$0.00` — update `CODEX_PRICE` in the script when pricing is known.

## What's NOT included

- **Claude.ai web / mobile**: not logged locally.
- **Gemini CLI**: stores sessions as protobuf (binary), not parseable without a schema.
- **Actual subscription billing**: Claude Max and similar plans bill differently. These estimates reflect equivalent API list prices.

## How it works

Each tool logs its sessions locally in different formats:

- **Claude Code** writes JSONL to `~/.claude/projects/<path>/`. Each assistant message includes a `usage` field with token counts and model name.
- **OpenClaw** writes JSONL to `~/.openclaw/agents/<agent>/sessions/`. Each assistant message includes `usage` with token counts and a `cost.total` field (pre-calculated by OpenClaw for Anthropic models).
- **Codex CLI** writes JSONL to `~/.codex/sessions/YYYY/MM/DD/`. Token usage is in `event_msg / token_count` entries as per-turn deltas (`last_token_usage`).

The script parses all three formats into a unified schema and displays them together.

## License

MIT
