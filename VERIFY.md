# Verify: Store FAQ bot that picks an answer from the help center

Use this checklist to confirm the audit tool works for a stranger's own FAQ bot scenario.

---

## What you need

A stranger brings their own failing FAQ bot setup:
- The tool they're auditing (their store FAQ bot or similar)
- What breaks when it fails (e.g., wrong policy → abandoned carts)
- A few real failing inputs from their logs

---

## Run through /play

1. Open the tool and start a new audit session
2. Paste your failing setup description and sample inputs
3. Walk through all five checks when prompted

---

## Confirm the deciding-check finding surfaces

The tool must surface the **unowned** check as a potential finding.

For the seeded specimen (Store FAQ bot that picks an answer from the help center), the audit identified "unowned" as the deciding check — the bot latches onto product names like "Nova Buds" or "Trail Jacket" without recognizing the shopper's actual intent (return, cancel, refund).

When you run your own scenario, the tool should:
- Walk you through each of the five checks
- Surface findings specific to your setup
- Identify which check is the deciding factor for your case

---

## Confirm numeric measurement is demanded

The tool must ask for a **numeric measurement** to confirm the deciding-check finding.

For the seeded specimen, the measurement would be: how many times does the bot return a shipping/delivery FAQ when the shopper asked about refunds or cancellations?

When the tool surfaces a finding, it should prompt you to provide:
- A count, percentage, or ratio that confirms the finding
- Evidence from your own logs or test runs

If the tool accepts vague descriptions like "it happens sometimes" without pushing for a number, verification fails.

---

## Verification checklist

| Step | Pass criteria |
|------|---------------|
| Stranger pastes their own FAQ bot scenario | Tool accepts and begins the five-check walk |
| All five checks are walked | Each check gets a rating or finding |
| Deciding check is surfaced | Tool identifies which check is the top crack for this scenario |
| Numeric measurement demanded | Tool asks for a specific count or ratio to confirm the finding |
| Audit ends with call + tripwire | Tool returns a ruling and an alarm with owner and cadence |

---

## What a passing run looks like

The stranger's audit ends with:
- A scored finding for each check
- The deciding check named explicitly
- A ship/hold/conditional call
- A tripwire with cadence and owner (like: "Re-run Every week until launch, then monthly once it's live. If the deciding check drifts more than one step from this ruling → Marisol and Bram, plus you since the bad routing lands on your desk.")

If any of these are missing or generic, the tool needs revision.
