---
description: Create a Convexent model from a prompt or uploaded document
argument-hint: <prompt or document path> [--name "..."] [--project <id>]
---

# Model Create

Create a new Convexent financial model. Two paths:

- **From a text prompt** — describe the model you want.
- **From a document** — point at a financial statement, deck, or spreadsheet, and the CLI uploads, extracts, and builds the model in one step.

## Arguments

- `$ARGUMENTS` — Required. Either a freeform description of the model or a path to a source document. May include `--name "..."` and `--project <id>`.
  - Examples:
    - `SaaS revenue model with ARR, churn, and 20% expansion`
    - `~/Documents/q3-financials.xlsx --name "Q3 Acquisition Target"`

## Instructions

### 1. Parse arguments

Decide whether `$ARGUMENTS` references a local file path (exists on disk) or is purely a description. Pull out any explicit `--name` / `--project` hints.

If `$ARGUMENTS` is empty, ask the user:
- What do you want to model? (Or do you have a source document to base it on?)
- Which project should it go in? (optional)

### 2. Confirm the project

If no project was given, run `convexent project list` and ask the user to pick one. If only one project exists, use it without prompting.

### 3. Create the model

**Prompt-only:**

```bash
convexent model create --prompt "<description>" --name "<name>" --project <project-id>
```

**From a document (one-step — recommended for most cases):**

```bash
convexent model create --documents <path> --prompt "<what to build from it>" --name "<name>" --project <project-id>
```

A `--prompt` is still required even with `--documents` — it tells the AI what *kind* of model to build from the source. If the user didn't give one, ask before creating.

**From a document (two-step — when the user wants to inspect extractions first):**

```bash
convexent upload <path> --project <project-id>
# capture the upload-id from the output, then:
convexent model create --from-extractions <upload-id> --prompt "<guidance>" --project <project-id>
```

`--documents` and `--from-extractions` are mutually exclusive. Default to one-step unless the user has indicated they want to review extractions first.

This is an async operation — the CLI polls until the job completes.

### 4. Report

Show the user:
- The new model ID
- The model name and project
- Suggested next steps:
  - `convexent model calculate <id>` to see initial metric values
  - `/convexent:model-analyze <id> <question>` to ask scenario questions
  - `/convexent:model-edit <id> <change>` to refine the structure
