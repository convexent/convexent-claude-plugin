---
description: Run AI scenario analysis on a Convexent model
argument-hint: <model-id> <analysis question>
---

# Model Analyze

Run an AI-powered scenario analysis on a Convexent financial model.

## Arguments

- `$ARGUMENTS` - Required: model ID and analysis question. Format: `<model-id> <question>`
  Example: `abc-123 What happens to IRR if we increase terminal growth to 3%?`

## Instructions

### 1. Parse arguments

Extract the model ID (first argument) and the analysis question (remaining text).

If `$ARGUMENTS` is empty, ask the user for the model ID and their analysis question.

### 2. Run the analysis

```bash
convexent model analyze run <model-id> --prompt "<question>"
```

This is an async operation — the CLI polls until the job completes. The output includes the AI's analysis response.

### 3. Present results

Show the user the analysis summary. If the analysis references specific metrics, optionally run a calculation to show current values:

```bash
convexent model calculate <model-id>
```

### 4. Follow-up

Ask if the user wants to:
- Ask a follow-up question (use `--thread` to continue the conversation)
- Make edits based on the analysis (use `/model-edit`)
- Send feedback on the analysis quality

For follow-up questions:
```bash
convexent model analyze run <model-id> --prompt "<follow-up>" --thread <thread-id>
```
