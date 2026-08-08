# Store FAQ Bot That Picks an Answer from the Help Center

## One-Paste Spec for Conversational Audit

**Domain:** E-commerce help-desk routing  
**Problem:** Shoppers ask about refunds, but the FAQ bot answers with shipping times because it latched onto the product name. Fix that before the busy sale week.  
**Stakes:** Shoppers get the wrong policy and leave the cart

---

## Pass Bar

The answer matches the shopper's real ask — not a nearby FAQ about the same product

---

## Usage Reality

Short mobile questions with product names in the middle

---

## Specimen Sentences (from Store help-desk chat logs)

```
how long do i have to return the Nova Buds after they ship
Nova Buds delivery says Friday — can i still cancel
refund for wrong size on the Trail Jacket, not a shipping question
```

---

## Five-Check Audit Framework

### Check 1: Unowned (Deciding Check)
**Rating:** 4/5  
Does the bot have a clear owner for refund/cancel intent before product-name matching fires?

**Worked Example:**  
Input: `refund for wrong size on the Trail Jacket, not a shipping question`  
Failure mode: Bot sees "Trail Jacket" and routes to shipping FAQ. No component owns the refund intent first.

### Check 2: Copies
**Rating:** 2/5  
Are there duplicate pathways that could intercept the same query?

**Worked Example:**  
Input: `Nova Buds delivery says Friday — can i still cancel`  
Failure mode: Both "delivery" and "cancel" trigger separate FAQ branches; product-name match wins arbitrarily.

### Check 3: Room
**Rating:** 1/5  
Is there headroom for the correct intent to surface before product matching?

**Worked Example:**  
Input: `how long do i have to return the Nova Buds after they ship`  
Failure mode: "Nova Buds" triggers product FAQ before "return" intent is evaluated.

### Check 4: Stitch
**Rating:** 2/5  
Do the routing components hand off cleanly, or do they collide?

**Worked Example:**  
Input: `refund for wrong size on the Trail Jacket, not a shipping question`  
Failure mode: Shopper explicitly says "not a shipping question" but product-name match overrides.

### Check 5: Ablation
**Rating:** 1/5  
If you remove product-name matching, does refund/cancel routing improve?

**Worked Example:**  
Input: `Nova Buds delivery says Friday — can i still cancel`  
Test: Disable product-name matcher → does "cancel" route correctly to cancellation FAQ?

---

## Audit Findings Summary

| Check | Rating | Measurement Required |
|-------|--------|---------------------|
| Unowned | 4 | Mass-across-boundary: % of refund/cancel queries routed to shipping FAQ |
| Copies | 2 | Max cross-head similarity between intent matchers |
| Room | 1 | Per-head entropy: does refund intent get evaluated before product match? |
| Stitch | 2 | Handoff collision rate on explicit negations ("not a shipping question") |
| Ablation | 1 | Delta when product-name matcher is zeroed out |

---

## Deciding Check: Unowned

The "unowned" check scored highest (4) because no component owns refund/cancel intent before product-name matching runs. This is the root cause of misrouting.

---

## Ship Call

Ship with conditions — CX lead Priya adds a dedicated refund/cancel word watch before any product-name matching runs, tested against "refund for wrong size on the Trail Jacket."

---

## Tripwire Alarm

Re-run Every week until launch, then monthly once it's live. If the deciding check drifts more than one step from this ruling → Marisol and Bram, plus you since the bad routing lands on your desk.

---

## Stranger Use

A stranger describes their own FAQ bot that misroutes queries. They provide:
1. The bot setup and what it's supposed to do
2. Who gets hurt when it fails
3. 3–5 real failing inputs from their logs

The auditor walks all five checks, proposes findings with the measurement that would confirm each, and returns:
- Scored audit (1–5 per check)
- Severity story grounded in their failing inputs
- Ship/hold call with conditions
- Tripwire with cadence and owner

---

## Verdict

Hold my rating — I'll defend it
