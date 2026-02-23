# opencode-usage

A bash script that summarizes your [OpenCode](https://opencode.ai) usage by custom categories (e.g. work vs personal) from the OpenCode's built-in local SQLite database.

Useful for splitting subscription costs between work and personal use, tracking token burn rates, or estimating what the same usage would cost via direct API access.

## Requirements

- `bash` 4+
- `sqlite3`
- OpenCode >=1.2.1 (with migrated data to SQLite storage)

## Install

```bash
# clone and symlink
git clone https://github.com/Kamyil/opencode-usage.git
ln -s "$(pwd)/opencode-usage/opencode-usage" ~/.local/bin/opencode-usage

# or just download
curl -o ~/.local/bin/opencode-usage https://raw.githubusercontent.com/Kamyil/opencode-usage/main/opencode-usage
chmod +x ~/.local/bin/opencode-usage

# or run directly without installing
bash <(curl -s https://raw.githubusercontent.com/Kamyil/opencode-usage/main/opencode-usage) \
  --category work:~/Work --category personal:~/Personal
```

## Usage

```bash
# basic — split by work and personal
opencode-usage \
  --category work:~/Work \
  --category personal:~/Personal

# multiple paths per category
opencode-usage \
  --category work:~/Work \
  --category personal:~/Personal \
  --category personal:~/.dotfiles \
  --category personal:~/second-brain

# specific month
opencode-usage --month 2026-02 \
  --category work:~/Work \
  --category personal:~/Personal

# custom categories
opencode-usage \
  --category client-a:~/Clients/a \
  --category client-b:~/Clients/b \
  --category oss:~/OpenSource

# with API cost estimation ($ per 1M tokens)
opencode-usage \
  --category work:~/Work \
  --category personal:~/Personal \
  --input-rate 5 --output-rate 15 --currency '$'

# model and tool breakdown
opencode-usage \
  --category work:~/Work \
  --category personal:~/Personal \
  --by-model --tools --pretty

# time analysis with trends over last 3 months
opencode-usage \
  --category work:~/Work \
  --category personal:~/Personal \
  --time-stats --months 3 --pretty

# full report with everything enabled
opencode-usage \
  --category work:~/Work \
  --category personal:~/Personal \
  --by-model --tools --diff-stats --time-stats \
  --cost --todos --titles --forks --pretty

# export as JSON (pipe to jq, save to file, etc.)
opencode-usage \
  --category work:~/Work \
  --category personal:~/Personal \
  --by-model --tools --json | jq .

# export as CSV
opencode-usage \
  --category work:~/Work \
  --category personal:~/Personal \
  --csv > usage-report.csv
```

## Options

### Core

| Option | Description |
|---|---|
| `--category NAME:PATH` | Assign a project path prefix to a named category. Repeatable. Multiple paths can share the same category name. |
| `--default-category NAME` | Bucket name for projects not matching any category (default: `other`) |
| `--month YYYY-MM` | Month to report (default: current month) |
| `--db PATH` | Path to opencode.db (default: `~/.local/share/opencode/opencode.db`) |
| `--top N` | Number of top projects to show per category (default: 5) |

### Cost estimation

| Option | Description |
|---|---|
| `--input-rate NUM` | Cost per 1M input tokens |
| `--output-rate NUM` | Cost per 1M output tokens |
| `--reasoning-rate NUM` | Cost per 1M reasoning tokens (defaults to input-rate) |
| `--cache-read-rate NUM` | Cost per 1M cache read tokens |
| `--cache-write-rate NUM` | Cost per 1M cache write tokens |
| `--currency SYMBOL` | Currency symbol for cost output (default: `$`) |

### Analysis features

All analysis features are opt-in. Enable any combination of them — each adds its own section to the report.

| Option | Description |
|---|---|
| `--by-model` | Show token breakdown by model/provider per category |
| `--tools` | Show tool usage breakdown per category (e.g. bash, read, edit, glob) |
| `--diff-stats` | Show lines of code added/deleted per category (from session git diff data) |
| `--time-stats` | Show session duration stats and activity-by-hour histogram |
| `--months N` | Show trends for the last N months (sessions and tokens over time) |
| `--cost` | Show actual cost from provider (if the provider reports per-request costs) |
| `--todos` | Show todo/task completion statistics by status and priority |
| `--titles` | Show top session titles per category (ranked by token usage) |
| `--forks` | Show forked session statistics (sessions branched from earlier conversations) |

### Output format

| Option | Description |
|---|---|
| `--pretty` | Pretty-print output with box drawing, bars, and colors |
| `--json` | Output report as JSON (includes data from all enabled features) |
| `--csv` | Output report as CSV (includes data from all enabled features) |

## Example output

### Default

```
OpenCode usage for 2026-02-01 to 2026-03-01 (exclusive)

bucket    sessions  messages  input_tokens  output_tokens  ...
personal  710       1911      942217054     14265498       ...
work      492       5236      326131117     9086495        ...
other     35        1635      21201636      1350473        ...

Percent split
bucket    pct_tokens_no_cache  pct_messages  pct_sessions
personal  72.71                21.76         57.4
work      25.54                59.62         39.77
other     1.76                 18.62         2.83

--- Summary ---
personal: 72.7% tokens, 21.8% messages (1916 messages, 956858689 tokens)
work: 25.5% tokens, 59.6% messages (5236 messages, 336056995 tokens)
other: 1.8% tokens, 18.6% messages (1635 messages, 23133180 tokens)
```

### Pretty (`--pretty`)

```
╭────────────────────────────────────────────────────────────╮
│ OpenCode Usage Report                                      │
│ 2026-02-01 to 2026-03-01                                   │
╰────────────────────────────────────────────────────────────╯

╭────────────────────────────────────────────────────────────╮
│ Overview                                                   │
╰────────────────────────────────────────────────────────────╯

  Category         Sessions   Messages             Tokens       Tokens+Cache
  ────────────── ────────── ────────── ────────────────── ──────────────────
  other                  35      1,635         23,133,180        303,771,088
  personal              710      1,961        957,287,990      2,720,344,046
  work                  492      5,236        336,056,995      1,594,203,598

╭────────────────────────────────────────────────────────────╮
│ Usage Split                                                │
╰────────────────────────────────────────────────────────────╯

  other
  Tokens   █░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   1.76%
  Messages ██████░░░░░░░░░░░░░░░░░░░░░░░░  18.51%
  Sessions █░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   2.83%

  personal
  Tokens   ██████████████████████░░░░░░░░  72.72%
  Messages ███████░░░░░░░░░░░░░░░░░░░░░░░   22.2%
  Sessions █████████████████░░░░░░░░░░░░░   57.4%

  work
  Tokens   ████████░░░░░░░░░░░░░░░░░░░░░░  25.53%
  Messages ██████████████████░░░░░░░░░░░░  59.28%
  Sessions ████████████░░░░░░░░░░░░░░░░░░  39.77%
```

### Model breakdown (`--by-model --pretty`)

```
╭────────────────────────────────────────────────────────────╮
│ Model Breakdown                                            │
╰────────────────────────────────────────────────────────────╯

  work
  Model                                 Sessions   Messages             Tokens       %
  unknown                                    241          0        243,678,854   72.5%
  github-copilot/claude-opus-4.5              91        433         48,410,036   14.4%
  github-copilot/claude-opus-4.6              58      1,921         15,716,238    4.7%
  openai/gpt-5.3-codex                        22      2,036         14,372,406    4.3%
  ...
```

### Tool usage (`--tools --pretty`)

```
╭────────────────────────────────────────────────────────────╮
│ Tool Usage                                                 │
╰────────────────────────────────────────────────────────────╯

  work
  Tool                           Calls       %
  bash                           2,381   25.6%
  read                           1,947   20.9%
  edit                           1,612   17.3%
  ...
```

### Git diff stats (`--diff-stats --pretty`)

```
╭────────────────────────────────────────────────────────────╮
│ Code Impact (Git Diff Stats)                               │
╰────────────────────────────────────────────────────────────╯

  Category             +Added     -Deleted          Net      Files
  ────────────── ──────────── ──────────── ──────────── ──────────
  other               +30,146       -2,052       28,094        413
  personal           +208,983     -130,500       78,483        491
  work             +2,680,965     -900,085    1,780,880      2,770
```

### Time analysis (`--time-stats --pretty`)

```
╭────────────────────────────────────────────────────────────╮
│ Session Duration                                           │
╰────────────────────────────────────────────────────────────╯

  Category          Avg (min)    Max (min)  Total (hrs)
  ────────────── ──────────── ──────────── ────────────
  other                 313.2       15,797       1163.9
  personal              299.7       56,238       2637.4
  work                  554.3       82,090       4554.6

╭────────────────────────────────────────────────────────────╮
│ Activity by Hour                                           │
╰────────────────────────────────────────────────────────────╯

  work
  08:00  ████████ 28
  09:00  ██████████████ 50
  10:00  ███████████████████ 65
  11:00  █████████████████ 58
  12:00  ████████████████████ 68
  13:00  ███████████████████ 65
  14:00  ██████████████████ 63
  15:00  ████████████ 42
  ...
```

### Multi-month trends (`--months 3 --pretty`)

```
╭────────────────────────────────────────────────────────────╮
│ Multi-Month Trends (Sessions)                              │
╰────────────────────────────────────────────────────────────╯

  Month           other   personal       work
  ────────── ────────── ────────── ──────────
  2025-12            60          2         92
  2026-01            63         58         73
  2026-02            41         21        106

╭────────────────────────────────────────────────────────────╮
│ Multi-Month Trends (Tokens)                                │
╰────────────────────────────────────────────────────────────╯

  Month                   other           personal               work
  ────────── ────────────────── ────────────────── ──────────────────
  2025-12             9,756,130             83,926         34,745,856
  2026-01            10,407,035         16,160,083         13,015,564
  2026-02            19,594,736          6,429,614         37,512,483

╭────────────────────────────────────────────────────────────╮
│ Monthly Token Distribution                                 │
╰────────────────────────────────────────────────────────────╯

  2025-12    ██████████████████████████████ 44,585,912
  2026-01    ██████████████████████████████ 39,582,682
  2026-02    ██████████████████████████████ 63,536,833
```

### Todo stats (`--todos --pretty`)

```
╭────────────────────────────────────────────────────────────╮
│ Todo Status by Category                                    │
╰────────────────────────────────────────────────────────────╯

  Category          cancelled    completed  in_progress      pending        total
  ────────────── ──────────── ──────────── ──────────── ──────────── ────────────
  work                      3          231           15           52          301
  personal                  2          374           29          157          562
  other                     3          102            1           15          121

  work           completion: ███████████████████████░░░░░░░ 76%
  personal       completion: ████████████████████░░░░░░░░░░ 66%
  other          completion: █████████████████████████░░░░░ 84%

╭────────────────────────────────────────────────────────────╮
│ Todo Priority by Category                                  │
╰────────────────────────────────────────────────────────────╯

  Category             high     medium        low   critical
  ────────────── ────────── ────────── ────────── ──────────
  work                  229         64          8          0
  personal              412        116         21         13
  other                  90         27          4          0
```

### Session titles (`--titles --pretty`)

```
╭────────────────────────────────────────────────────────────╮
│ Top Sessions by Tokens                                     │
╰────────────────────────────────────────────────────────────╯

  work
  Title                                                Messages             Tokens
  Investigating package count discrepancy                   0         23,129,020
  Analyzing Docker Nginx setup                              0         12,448,451
  Adding filter persistence for deliveries page             0         10,813,508
  ...
```

### Fork tracking (`--forks --pretty`)

```
╭────────────────────────────────────────────────────────────╮
│ Forked Sessions                                            │
╰────────────────────────────────────────────────────────────╯

  Category            Forks    Parents Avg/Parent
  ────────────── ────────── ────────── ──────────
  other                  25         12        2.1
  personal               44         18        2.4
  work                  117         44        2.7

  Most-forked sessions:

  work
    Project CRM: opportunity flow, goal cafeteria...  23 forks
    Compare SvelteKit+Drizzle vs Laravel+Inertia ...   8 forks
    Improve app visuals: colors, fonts, styling only   7 forks
    ...
```

### JSON output (`--json`)

```bash
opencode-usage --category work:~/Work --category personal:~/Personal \
  --by-model --tools --diff-stats --json | jq .
```

Produces a JSON object with keys: `period`, `overview`, `percentages`, `top_projects`, `summary`, and optional keys for each enabled feature (`models`, `tools`, `diff_stats`, `time_stats`, `hour_stats`, `actual_cost`, `todos_status`, `todos_priority`, `titles`, `forks_summary`, `forks_top`).

### CSV output (`--csv`)

```bash
opencode-usage --category work:~/Work --category personal:~/Personal \
  --tools --diff-stats --csv > report.csv
```

Outputs multiple CSV sections separated by blank lines, each with its own header row. Sections correspond to the same data as JSON keys above.

## How it works

The script reads OpenCode's local SQLite database (`~/.local/share/opencode/opencode.db`), which stores all conversation sessions, messages, token usage, parts (tool calls, reasoning steps), todos, and session metadata. It:

1. Filters data to the selected month (by first message timestamp)
2. Joins sessions with their projects to get the project worktree path
3. Matches paths against your `--category` prefixes (first match wins)
4. Builds a materialized temp table of categorized sessions with token counts
5. Runs all requested analysis queries in a single `sqlite3` invocation
6. Parses results via marker-based section splitting
7. Renders output in the selected format (text, pretty, JSON, or CSV)

### Data sources by feature

| Feature | Tables/columns used |
|---|---|
| Base report | `session`, `message` — sessions, messages, token counts |
| `--by-model` | `message.data` — `$.model.providerID`, `$.model.modelID` |
| `--tools` | `part.data` — `$.tool` (tool call parts) |
| `--diff-stats` | `session.summary_additions`, `summary_deletions`, `summary_files` |
| `--time-stats` | `session.time_created`, `time_updated` |
| `--months` | Same as base, grouped by month over N months |
| `--cost` | `part.data` — `$.cost` on `step-finish` parts |
| `--todos` | `todo` table — status, priority per session |
| `--titles` | `session.title` |
| `--forks` | `session.parent_id` (non-null = forked from another session) |

## Notes

- Token data is only available from when OpenCode migrated to SQLite storage. Older months will show sessions/messages but zero tokens.
- Categories are matched by path prefix in the order specified. First match wins.
- The `--default-category` bucket catches anything not matching any defined category.
- `--cost` depends on the provider reporting per-request costs in `step-finish` parts. Many providers (including GitHub Copilot) report 0.
- `--months` assigns sessions to months by their first message timestamp, not by activity window.
- The `unknown` model shown in `--by-model` output represents sessions where model info was not recorded in message metadata.

## License

MIT
