---
description: Export a Convexent model to Excel
argument-hint: <model-id> [output-path]
---

# Model Export

Download a Convexent model as an Excel workbook (.xlsx) with all scenarios and live formulas baked in.

## Arguments

- `$ARGUMENTS` — Required: model ID. Optional: output path.
  - Examples:
    - `abc-123`
    - `abc-123 ~/Downloads/q3-target.xlsx`

## Instructions

### 1. Parse arguments

Extract the model ID (first argument) and an optional output path (second).

If `$ARGUMENTS` is empty, ask for the model ID. If the user doesn't know it, run `convexent model list` (filtered by project if helpful) and let them pick.

### 2. Pick an output path

If the user didn't supply one:
- Fetch the model name: `convexent model get <model-id> --output json | jq -r .name`
- Slugify it: lowercase, replace whitespace with `-`, strip non-alphanumerics.
- Default to `./<slug>.xlsx` in the current working directory.
- If that path already exists, ask before overwriting.

### 3. Export

```bash
convexent model export <model-id> --out <path>
```

### 4. Report

Tell the user the absolute path of the saved file. Mention that the workbook includes all scenarios and live formulas (not just baked values), so they can edit assumptions in Excel and see metrics recalculate.
