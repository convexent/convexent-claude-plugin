---
description: Edit a Convexent model using AI — propose, review, apply, verify
argument-hint: <model-id> <edit instruction>
---

# Model Edit

Run the full AI edit workflow: propose a patch, apply it, and verify with calculation.

## Arguments

- `$ARGUMENTS` - Required: model ID and edit instruction. Format: `<model-id> <instruction>`
  Example: `abc-123 Add a DCF section with WACC and terminal value`

## Instructions

### 1. Parse arguments

Extract the model ID (first argument) and the edit instruction (remaining text).

If `$ARGUMENTS` is empty, ask the user for the model ID and what they want to change.

### 2. Get current state

Fetch the model to understand its current structure:

```bash
convexent model get <model-id>
```

### 3. Propose the edit

```bash
convexent model edit propose <model-id> --prompt "<instruction>"
```

This is an async operation — the CLI polls until the job completes. The output includes:
- A prompt ID (needed for apply/discard)
- A summary of the proposed changes

### 4. Report and confirm

Show the user:
- The proposed changes summary
- The prompt ID
- Ask if they want to apply, discard, or modify the edit

### 5. Apply or discard

Based on user response:

```bash
# Apply
convexent model edit apply <model-id> --prompt <prompt-id>

# Discard
convexent model edit discard <model-id> --prompt <prompt-id>
```

### 6. Verify

If applied, run a calculation to verify the changes:

```bash
convexent model calculate <model-id>
```

Show the user the updated metrics to confirm the edit looks correct.
