# Store FAQ bot that picks an answer from the help center

## Full Audit

This audit examines whether the store FAQ bot correctly splits the work between recognizing shopper intent and matching product names—before the busy sale week.

---

## The Standard

**Pass bar committed:**

> The answer matches the shopper's real ask — not a nearby FAQ about the same product

**What breaks if the parts aren't really splitting the work:**

> Shoppers get the wrong policy and leave the cart

---

## The Specimen

**Setup under audit:** Store FAQ bot that picks an answer from the help center

**Usage reality:** Short mobile questions with product names in the middle

**Source:** Store help-desk chat logs

### Practice lines tested

```
how long do i have to return the Nova Buds after they ship
Nova Buds delivery says Friday — can i still cancel
refund for wrong size on the Trail Jacket, not a shipping question
```

---

## Check Findings

| Check | Rating |
|-------|--------|
| Unowned | 4 |
| Copies | 2 |
| Room | 1 |
| Stitch | 2 |
| Ablation | 1 |

**Deciding check:** unowned

The "unowned" check scores highest (4) because shopper intent—refund, return, cancel—has no dedicated handler before product-name matching fires.

---

## Severity Story

When a shopper types "refund for wrong size on the Trail Jacket, not a shipping question," the bot latches onto "Trail Jacket" and returns shipping times. The refund intent is unowned—no part of the setup claims it before the product-name matcher runs. The shopper sees irrelevant shipping info, assumes the store can't help, and abandons the cart. CX gets the escalation.

---

## The Call

**Verdict:** Ship with conditions — CX lead Priya adds a dedicated refund/cancel word watch before any product-name matching runs, tested against "refund for wrong size on the Trail Jacket."

**Pressure response:** Hold my rating — I'll defend it

---

## The Tripwire

**Cadence:** Re-run Every week until launch, then monthly once it's live.

**Alarm condition:** If the deciding check drifts more than one step from this ruling → Marisol and Bram, plus you since the bad routing lands on your desk.

---

## Summary

This audit found that the store FAQ bot's highest-severity gap is unowned intent: refund and cancel requests have no handler before product-name matching. The condition for shipping is that Priya's refund/cancel word watch must be in place and tested. The tripwire ensures weekly monitoring until launch, then monthly, with Marisol and Bram alerted if the unowned check drifts.
