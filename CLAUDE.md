# Convexent CLI — Agent Instructions

You have access to the `convexent` CLI for working with Convexent financial models. All calculation, Excel generation, validation, and LLM work happen server-side. The CLI is a thin HTTP client.

## Setup

The CLI requires authentication. Check auth status first:

```bash
convexent auth status
```

If not authenticated, the user needs to set a token:
```bash
convexent auth set-token <api-key>
convexent auth set-url <api-url>   # if not using production
```

If you get exit code 3, tell the user to authenticate.

## Quick Reference

### Browse & Inspect
```bash
convexent project list --search "acme"
convexent project get <project-id>
convexent model list --project <project-id>
convexent model get <model-id>                # summary: assumptions + metric formulas
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
# Full spec save (structural changes)
convexent model save <model-id> --spec updated.json

# Surgical JSON Patch (value edits)
convexent model patch <model-id> --ops '[{"op":"replace","path":"/models/0/assumptions/3/value","value":0.15}]'
```

### AI Edit (Propose → Apply)
```bash
# Step 1: Propose a patch
convexent model edit propose <model-id> --prompt "Add a DCF section with WACC and terminal value"

# Or use a suggestion by label:
convexent model suggestions <model-id>
convexent model edit propose <model-id> --suggestion "sensitivity"

# Step 2: Apply or discard the proposed patch (prompt ID is in the propose output)
convexent model edit apply <model-id> --prompt <prompt-id>
convexent model edit discard <model-id> --prompt <prompt-id>
```

### AI Analysis
```bash
convexent model analyze run <model-id> --prompt "What happens to IRR if terminal growth increases to 3%?"
```

### Upload & Extract Documents
```bash
convexent upload <file> --project <project-id>           # upload + auto-extract
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
