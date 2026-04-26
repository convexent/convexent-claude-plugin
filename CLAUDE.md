# Convexent CLI — Agent Instructions

You have access to the `convexent` CLI for working with Convexent financial models. All calculation, Excel generation, validation, and LLM work happen server-side. The CLI is a thin HTTP client — it needs both the binary on PATH **and** network egress to `api.convexent.com` to function.

## Preflight

Before any other command, confirm the CLI is present and working:

```bash
convexent --version
```

| Result | What it means | Next step |
|---|---|---|
| Prints a version (e.g. `0.1.2`) | Installed and on PATH | Continue to Auth |
| `command not found` | Not installed, or the plugin's advertised `bin/` is empty | Go to **Install** |
| Hangs / network error | Sandbox can't reach `api.convexent.com` | Tell the user; the plugin is unusable until egress is allowed |

## Install

If `convexent` is not on PATH, install it from npm. In sandboxed environments (e.g. Cowork) where `/usr/lib/node_modules` is not writable, use a user-local prefix:

```bash
mkdir -p ~/.npm-global
npm config set prefix ~/.npm-global
npm install -g convexent
export PATH="$HOME/.npm-global/bin:$PATH"
```

To make the PATH change stick across shell invocations, also append the export to `~/.bashrc`:

```bash
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.bashrc
```

Verify:

```bash
convexent --version   # should print 0.1.2 or later
```

If `npm install -g convexent` fails with `EACCES` on `/usr/lib/node_modules`, you skipped the `npm config set prefix` step — the global install is trying to write into a system directory. Retry from the top of this block.

If the install succeeds but a subsequent shell reports `command not found`, the new shell isn't sourcing `~/.bashrc`. Either invoke the binary by full path (`~/.npm-global/bin/convexent …`) or prepend `export PATH="$HOME/.npm-global/bin:$PATH"` to each command.

## Auth

Single-command preflight — `auth status` verifies with the server:

```bash
convexent auth status
```

| Result | Action |
|---|---|
| `"authenticated": true` | Continue. |
| `"authenticated": false` (HTTP 401) | Stored token is stale or revoked. See **If not authenticated** below. |
| `"authenticated": null` / network error | API unreachable. Tell the user; the plugin can't function until the API is reachable. |

### If not authenticated

Preferred path is a browser-based login — no API key handling at all. The CLI prints a URL and short verification code; the user opens the URL on any device they're already signed into, approves, and the token is saved automatically.

For agents with per-bash-call timeouts (e.g. Cowork's ~45s ceiling), drive the flow in two steps so each call returns immediately:

```bash
# 1. Start a session — prints session_id, code, auth_url, expires_at as JSON.
convexent auth login-start

# 2. Surface the URL and code to the user; ask them to approve in their browser.

# 3. Poll on your own cadence (every 5–10s). Each call exits immediately.
convexent auth login-status --session <session-id>
# Returns {status: "pending" | "approved" | "expired"}.
# On "approved", the token is saved to ~/.convexent/credentials.json automatically.
```

For workstation use without timeout constraints, the wrapper does the polling for you:

```bash
convexent auth login   # opens a browser if available; otherwise prints URL + code; polls ~5 min
```

### Storing an API key directly

If the user already has a long-lived API key (from app.convexent.com → Settings → API keys), set it via stdin so the key doesn't appear on argv:

```bash
echo -n "$TOKEN" | convexent auth set-token --stdin
```

For non-prod hosts, pair with `convexent auth set-url <url>`.

**Don't ask the user to paste their API key into chat.** `--stdin` keeps the key off argv and shell history, but the agent transcript itself is still a leak surface. Either ask them to run `auth set-token --stdin` outside the agent session, or use `auth login` / `login-start` (which never exposes a raw key at all).

### Legacy: `CONVEXENT_API_KEY` env var

Still honored, but on its way out. The env var is inherited by every child process and easy to leak via `.bashrc` or CI logs. For new flows, prefer `auth login` (interactive) or `auth set-token --stdin` (programmatic).

## Quick Reference

### Browse & Inspect

```bash
convexent project list --search "acme"
convexent project get <project-id>
convexent model list --project <project-id>
convexent model get <model-id>                 # summary: assumptions + metric formulas
convexent model get <model-id> --full-json     # raw DSL spec JSON
```

### Calculate & Export

```bash
convexent model calculate <model-id>                    # formatted metric table
convexent model calculate <model-id> --scenario upside  # specific scenario
convexent model export <model-id> --out model.xlsx      # download Excel
```

### Edit Model (Direct)

```bash
# Full-spec save (structural changes)
convexent model save <model-id> --spec updated.json

# Surgical JSON Patch (value edits)
convexent model patch <model-id> --ops '[{"op":"replace","path":"/models/0/assumptions/3/value","value":0.15}]'
```

### AI Edit (Propose → Apply)

```bash
# Step 1: propose a patch
convexent model edit propose <model-id> --prompt "Add a DCF section with WACC and terminal value"

# Or use a suggestion by label:
convexent model suggestions <model-id>
convexent model edit propose <model-id> --suggestion "sensitivity"

# Step 2: apply or discard (prompt ID is in the propose output)
convexent model edit apply <model-id> --prompt <prompt-id>
convexent model edit discard <model-id> --prompt <prompt-id>

# Step 3: verify
convexent model calculate <model-id>
```

### AI Analysis

```bash
convexent model analyze run <model-id> --prompt "What happens to IRR if terminal growth increases to 3%?"
convexent model analyze run <model-id> --prompt "Now try 2%" --thread <thread-id>
```

### Upload & Extract Documents

```bash
convexent upload <file> --project <project-id>            # upload + auto-extract
convexent extractions --project <project-id>              # list extractions
convexent extraction <extraction-id>                      # view extraction details
```

### Create Model

```bash
# From a text prompt
convexent model create --prompt "SaaS revenue model with ARR, churn, and expansion" --name "SaaS Model"
convexent model create --prompt "DCF valuation" --project <project-id>

# One-step: upload, extract, and create from source documents
convexent model create --documents financials.xlsx --prompt "Build a 3-statement model" --name "My Model"

# Two-step: upload first, then reference the upload ID
convexent upload financials.xlsx --project <project-id>
convexent model create --from-extractions <upload-id> --prompt "Build a 3-statement model" --project <project-id>
```

Note: `--documents` and `--from-extractions` are mutually exclusive.

### History & Undo

```bash
convexent model audit <model-id>
convexent model restore <model-id> --snapshot <audit-id>
```

### Jobs

```bash
convexent job list --status started
convexent job get <job-id>
convexent job cancel <job-id>
```

## Output Formats

All commands return JSON by default when piped (non-TTY). For human-readable output, use `--output text` or `--output table`. Parse JSON output with `jq` or similar.

```bash
# Get a specific assumption value
convexent model get <id> --full-json | jq '.models[0].assumptions[] | select(.id == "revenue_growth")'

# Get calculated metric values as JSON
convexent model calculate <id> --output json | jq '.metrics.revenue.values.base'
```

## Important Patterns

- **AI edits are two-step**: `propose` creates a draft patch, `apply` commits it. Always apply (or discard) after proposing — a proposed patch that's never applied leaves the model in its pre-edit state.
- **Suggestions have no IDs**: Use `--suggestion "<partial-label>"` with fuzzy label matching (case-insensitive).
- **Metric values are scenario-nested**: `calculate` returns `{base: [...], downside: [...]}`, not flat arrays.
- **Prompts list endpoint**: `/prompts/` returns a plain array, not paginated.
- **Always run `model calculate` after any edit** to confirm the model still calculates without errors before reporting success to the user.

## Exit Codes

| Code | Meaning | Action |
|------|---------|--------|
| 0 | Success | — |
| 1 | General error | Check stderr |
| 3 | Auth required | User needs to run `convexent auth set-token <key>` (see **Auth** above) |
| 4 | Not found | Check the resource ID |
| 5 | Permission denied | User lacks access — confirm they're in the right org (`convexent auth status` shows the active org) |
| 6 | Validation error | Check input format / values against the DSL schema |
| 7 | Job failed | `convexent job get <job-id>` for error details |

## Troubleshooting Checklist

When something breaks, work through this list in order before asking the user:

1. `convexent --version` — is the CLI even installed?
2. `convexent auth status` — authenticated, and pointing at the right API URL?
3. `convexent project list` — can you read data at all? (rules out network/auth issues)
4. Re-read the exit code table above.
5. Only then: ask the user, including the failing command and the full stderr.
