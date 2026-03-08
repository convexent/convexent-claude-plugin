---
description: Show Convexent CLI command reference and common workflows
argument-hint: Optional topic (edit, analyze, auth, calculate, export)
---

# Convexent CLI Help

Show command reference and example workflows for the Convexent CLI.

## Arguments

- `$ARGUMENTS` - Optional: a topic (e.g. "edit", "analyze", "auth", "calculate", "export")

## Instructions

### If no topic is given

Print the full command tree:

```
convexent
├── auth
│   ├── status             Show current auth state
│   ├── set-token <token>  Store an API key
│   ├── set-url <url>      Store default API URL
│   └── logout             Clear stored credentials
├── project
│   ├── list               List projects (--search, --limit)
│   └── get <id>           Project detail + models + data room
├── model
│   ├── list               List models (--project, --limit)
│   ├── get <id>           Model summary (--full-json for raw DSL)
│   ├── save <id>          Full-spec write (--spec <file|->)
│   ├── patch <id>         JSON Patch (--ops <json|file|->)
│   ├── calculate <id>     Run calculation (--scenario)
│   ├── export <id>        Download Excel (--out <path>)
│   ├── audit <id>         List change history
│   ├── restore <id>       Restore snapshot (--snapshot <audit_id>)
│   ├── prompts <id>       List AI prompts (--status, --mode, --limit)
│   ├── suggestions <id>   AI improvement suggestions
│   ├── edit
│   │   ├── propose <id>   Propose AI patch (--prompt | --suggestion "<label>")
│   │   ├── apply <id>     Apply proposed patch (--prompt <pid>)
│   │   ├── discard <id>   Discard proposed patch (--prompt <pid>)
│   │   └── feedback <id>  Send feedback (--prompt <pid>, --rating up|down)
│   └── analyze
│       ├── run <id>       Scenario analysis (--prompt, --thread)
│       └── feedback <id>  Send feedback (--prompt <pid>, --rating up|down)
├── job
│   ├── list               List jobs (--status, --limit)
│   ├── get <id>           Job details
│   └── cancel <id>        Cancel running job
└── Global flags: --output json|text|table, --api-url, --token, --verbose, --quiet
```

### If topic is "edit"

Show the AI edit workflow:

```
AI Edit Workflow (Propose → Apply)
==================================

1. Get suggestions (optional):
   npx convexent model suggestions <model-id>

2. Propose a patch:
   npx convexent model edit propose <model-id> --prompt "Add WACC calculation"
   # or from a suggestion:
   npx convexent model edit propose <model-id> --suggestion "sensitivity"

3. Review the response — it shows the prompt ID and summary

4. Apply or discard:
   npx convexent model edit apply <model-id> --prompt <prompt-id>
   npx convexent model edit discard <model-id> --prompt <prompt-id>

5. Verify with calculation:
   npx convexent model calculate <model-id>

6. Send feedback (optional):
   npx convexent model edit feedback <model-id> --prompt <prompt-id> --rating up
```

### If topic is "analyze"

Show the analysis workflow:

```
Scenario Analysis Workflow
==========================

1. Run an analysis:
   npx convexent model analyze run <model-id> --prompt "What if revenue drops 20%?"

2. Continue the thread:
   npx convexent model analyze run <model-id> --prompt "Now try 30% drop" --thread <thread-id>

3. Send feedback:
   npx convexent model analyze feedback <model-id> --prompt <prompt-id> --rating up
```

### If topic is "auth"

Show auth setup instructions:

```
Authentication Setup
====================

Check status:    npx convexent auth status
Set API key:     npx convexent auth set-token sm_your_api_key_here
Set API URL:     npx convexent auth set-url https://api.example.com
Clear creds:     npx convexent auth logout

You can also use:
  --token <key>  flag on any command
  CONVEXENT_API_KEY env var
  CONVEXENT_API_URL env var

Priority: --token flag > env var > stored credentials
```

### If topic is "calculate"

Show calculation examples:

```
Calculation
===========

Basic:           npx convexent model calculate <id>
With scenario:   npx convexent model calculate <id> --scenario upside
As JSON:         npx convexent model calculate <id> --output json

Extract specific metric:
  npx convexent model calculate <id> --output json | jq '.metrics.revenue.values.base'

Note: metric values are nested by scenario: {base: [...], upside: [...], downside: [...]}
```

### If topic is "export"

Show export examples:

```
Excel Export
============

Default name:    npx convexent model export <id>
Custom path:     npx convexent model export <id> --out ./my-model.xlsx
```

### For any other topic

Run `npx convexent <topic> --help` to show the CLI's built-in help for that command.
