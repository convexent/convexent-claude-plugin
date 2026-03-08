# Convexent CLI — Agent Instructions

You have access to the `convexent` CLI for working with Convexent financial models. All calculation, Excel generation, validation, and LLM work happen server-side. The CLI is a thin HTTP client.

## Setup

The CLI requires authentication. Check auth status first:

```bash
npx convexent auth status
```

If not authenticated, the user needs to set a token:
```bash
npx convexent auth set-token <api-key>
npx convexent auth set-url <api-url>   # if not using production
```

If you get exit code 3, tell the user to authenticate.

## Quick Reference

### Browse & Inspect
```bash
npx convexent project list --search "acme"
npx convexent project get <project-id>
npx convexent model list --project <project-id>
npx convexent model get <model-id>                # summary: assumptions + metric formulas
npx convexent model get <model-id> --full-json     # raw DSL spec JSON
```

### Calculate & Export
```bash
npx convexent model calculate <model-id>                    # formatted metric table
npx convexent model calculate <model-id> --scenario upside  # specific scenario
npx convexent model export <model-id> --out model.xlsx      # download Excel
```

### Edit Model (Direct)
```bash
# Full spec save (structural changes)
npx convexent model save <model-id> --spec updated.json

# Surgical JSON Patch (value edits)
npx convexent model patch <model-id> --ops '[{"op":"replace","path":"/models/0/assumptions/3/value","value":0.15}]'
```

### AI Edit (Propose → Apply)
```bash
# Step 1: Propose a patch
npx convexent model edit propose <model-id> --prompt "Add a DCF section with WACC and terminal value"

# Or use a suggestion by label:
npx convexent model suggestions <model-id>
npx convexent model edit propose <model-id> --suggestion "sensitivity"

# Step 2: Apply or discard the proposed patch (prompt ID is in the propose output)
npx convexent model edit apply <model-id> --prompt <prompt-id>
npx convexent model edit discard <model-id> --prompt <prompt-id>
```

### AI Analysis
```bash
npx convexent model analyze run <model-id> --prompt "What happens to IRR if terminal growth increases to 3%?"
```

### History & Undo
```bash
npx convexent model audit <model-id>
npx convexent model restore <model-id> --snapshot <audit-id>
```

### Jobs
```bash
npx convexent job list --status started
npx convexent job get <job-id>
npx convexent job cancel <job-id>
```

## Output Formats

All commands return JSON by default when piped (non-TTY). For human-readable output, use `--output text` or `--output table`. Parse JSON output with `jq` or similar.

```bash
# Get a specific assumption value
npx convexent model get <id> --full-json | jq '.models[0].assumptions[] | select(.id == "revenue_growth")'

# Get calculated metric values as JSON
npx convexent model calculate <id> --output json | jq '.metrics.revenue.values.base'
```

## Important Patterns

- **AI edits are two-step**: `propose` creates a draft patch, `apply` commits it. Always apply after proposing.
- **Suggestions have no IDs**: Use `--suggestion "<partial-label>"` with fuzzy label matching (case-insensitive).
- **Metric values are scenario-nested**: Calculate returns `{base: [...], downside: [...]}`, not flat arrays.
- **Prompts list endpoint**: `/prompts/` returns a plain array, not paginated.

## Exit Codes

| Code | Meaning | Action |
|------|---------|--------|
| 0 | Success | — |
| 1 | General error | Check stderr |
| 3 | Auth required | User needs to run `auth set-token` |
| 4 | Not found | Check the resource ID |
| 5 | Permission denied | User lacks access |
| 6 | Validation error | Check input format/values |
| 7 | Job failed | Check job error details |
