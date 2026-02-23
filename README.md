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
git clone https://github.com/YOUR_USERNAME/opencode-usage.git
ln -s "$(pwd)/opencode-usage/opencode-usage" ~/.local/bin/opencode-usage

# or just download
curl -o ~/.local/bin/opencode-usage https://raw.githubusercontent.com/YOUR_USERNAME/opencode-usage/main/opencode-usage
chmod +x ~/.local/bin/opencode-usage

# or run directly without installing
bash <(curl -s https://raw.githubusercontent.com/YOUR_USERNAME/opencode-usage/main/opencode-usage) \
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
```

## Options

| Option | Description |
|---|---|
| `--category NAME:PATH` | Assign a project path prefix to a named category. Repeatable. Multiple paths can share the same category name. |
| `--default-category NAME` | Bucket name for projects not matching any category (default: `other`) |
| `--month YYYY-MM` | Month to report (default: current month) |
| `--db PATH` | Path to opencode.db (default: `~/.local/share/opencode/opencode.db`) |
| `--top N` | Number of top projects to show per category (default: 5) |
| `--input-rate NUM` | Cost per 1M input tokens |
| `--output-rate NUM` | Cost per 1M output tokens |
| `--reasoning-rate NUM` | Cost per 1M reasoning tokens (defaults to input-rate) |
| `--cache-read-rate NUM` | Cost per 1M cache read tokens |
| `--cache-write-rate NUM` | Cost per 1M cache write tokens |
| `--currency SYMBOL` | Currency symbol for cost output (default: `$`) |
| `--pretty` | Pretty-print output with box drawing, bars, and colors |

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

╭────────────────────────────────────────────────────────────╮
│ Top Projects                                               │
╰────────────────────────────────────────────────────────────╯

  personal
  Path                                            Sessions   Messages             Tokens
  ~/Personal/Projects/my-app                           379        100        622,120,860
  ~/Personal/Projects/side-project                      72        737        131,435,367
  ~/.dotfiles                                          183        802         73,102,440

  work
  Path                                            Sessions   Messages             Tokens
  ~/Work/Projects/acme-frontend                        181         73        195,827,305
  ~/Work/Projects/acme-api                              35          7         49,860,194

╭────────────────────────────────────────────────────────────╮
│ Summary                                                    │
╰────────────────────────────────────────────────────────────╯

  personal        72.7% tokens  |  22.2% messages  |  1,961 msgs  |  957,287,990 tokens
  work            25.5% tokens  |  59.3% messages  |  5,236 msgs  |  336,056,995 tokens
  other           1.8% tokens  |  18.5% messages  |  1,635 msgs  |  23,133,180 tokens
```

## How it works

The script reads OpenCode's local SQLite database (`~/.local/share/opencode/opencode.db`), which stores all conversation sessions, messages, and token usage. It:

1. Filters data to the selected month
2. Joins sessions with their projects to get the project worktree path
3. Matches paths against your `--category` prefixes (first match wins)
4. Aggregates tokens and messages per category
5. Prints breakdown tables and a summary

## Notes

- Token data is only available from when OpenCode migrated to SQLite storage. Older months will show sessions/messages but zero tokens.
- Categories are matched by path prefix in the order specified. First match wins.
- The `--default-category` bucket catches anything not matching any defined category.

## License

MIT
