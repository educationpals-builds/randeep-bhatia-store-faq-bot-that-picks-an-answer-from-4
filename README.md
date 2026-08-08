# Store FAQ bot that picks an answer from the help center

Shoppers ask about refunds, but the FAQ bot answers with shipping times because it latched onto the product name. This audit fixes that before the busy sale week.

## Verdict

**Ship with conditions** — CX lead Priya adds a dedicated refund/cancel word watch before any product-name matching runs, tested against "refund for wrong size on the Trail Jacket."

## The standard

The answer matches the shopper's real ask — not a nearby FAQ about the same product

## What breaks if this fails

Shoppers get the wrong policy and leave the cart

## Tripwire

Re-run Every week until launch, then monthly once it's live. If the deciding check drifts more than one step from this ruling → Marisol and Bram, plus you since the bad routing lands on your desk.

## Deciding check

**Unowned** — scored 4 out of 5. The bot latches onto product names (Nova Buds, Trail Jacket) and ignores the shopper's actual intent (return, cancel, refund). No part of the current setup owns the job of detecting intent words before product-name matching fires.

## Worked example: the audit that produced this ruling

These lines came from Store help-desk chat logs:

```
how long do i have to return the Nova Buds after they ship
Nova Buds delivery says Friday — can i still cancel
refund for wrong size on the Trail Jacket, not a shipping question
```

Usage reality: Short mobile questions with product names in the middle

The bot sees "Nova Buds" or "Trail Jacket" and routes to shipping/delivery FAQs, missing the actual ask (return window, cancellation, refund). The unowned check scores highest because nothing in the current setup claims responsibility for intent detection before product matching.

## One-paste rebuild block

To run this audit on your own FAQ bot:

1. State what your bot is supposed to do and who gets hurt when it fails
2. Paste 3–5 real failing inputs from your logs
3. Walk the five checks — score each 1–5
4. Identify which check scores highest (the deciding check)
5. Make a call: ship, hold, or ship with conditions
6. Set a tripwire: cadence, trigger threshold, and who wakes up

The five checks are documented in [METHOD.md](METHOD.md). Full audit details are in [charter.md](charter.md).

## Verification

See [VERIFY.md](VERIFY.md) to confirm the tool surfaces the deciding-check finding and demands a numeric measurement for it.

<!-- educationpals-build-verified -->
