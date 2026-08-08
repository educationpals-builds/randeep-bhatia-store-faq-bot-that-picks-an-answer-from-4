# Review Cycle 01 — Store FAQ bot that picks an answer from the help center

Reference audit cycle for the FAQ routing problem where shoppers ask about refunds but the bot latches onto product names and returns shipping FAQs instead.

---

## Packet under review

**Source:** Store help-desk chat logs  
**Sample size:** 3 shopper questions  
**Pass bar:** The answer matches the shopper's real ask — not a nearby FAQ about the same product

### Lines reviewed

| Line # | Shopper question |
|--------|------------------|
| 1 | how long do i have to return the Nova Buds after they ship |
| 2 | Nova Buds delivery says Friday — can i still cancel |
| 3 | refund for wrong size on the Trail Jacket, not a shipping question |

---

## Per-check measurements produced

### Check 1: Unowned (Rating: 4)

**Measurement:** Mass-across-boundary  
**Result:** 0.73 — intent tokens ("return," "cancel," "refund") route to shipping FAQ cluster instead of returns/cancellation cluster  
**Threshold:** Flag if > 0.40  
**Status:** ⚠️ FLAGGED

### Check 2: Copies (Rating: 2)

**Measurement:** Max cross-head similarity (L2-normalized flattened per-head maps, stack, matmul against transpose, read off-diagonal)  
**Result:** 0.31 — moderate duplication between product-name attention and intent-word attention  
**Threshold:** Flag if > 0.50  
**Status:** ✓ PASS

### Check 3: Room (Rating: 1)

**Measurement:** Per-head entropy versus uniform  
**Result:** 0.89 — attention spread is healthy, not over-concentrated  
**Threshold:** Flag if < 0.60  
**Status:** ✓ PASS

### Check 4: Stitch (Rating: 2)

**Measurement:** Cross-boundary coherence delta  
**Result:** 0.28 — some mismatch when intent crosses from question to FAQ selection  
**Threshold:** Flag if > 0.35  
**Status:** ✓ PASS

### Check 5: Ablation (Rating: 1)

**Measurement:** Ablation delta (zero one head before concat)  
**Result:** 0.12 — removing product-name head causes minimal routing change  
**Threshold:** Flag if > 0.25  
**Status:** ✓ PASS

---

## Caught-or-missed line

**Line 3:** "refund for wrong size on the Trail Jacket, not a shipping question"

| Outcome | Detail |
|---------|--------|
| Bot response before fix | "Trail Jacket ships in 3–5 business days" |
| Expected response | Refund policy for wrong-size items |
| Caught by | Unowned check — mass-across-boundary flagged intent tokens routing to wrong cluster |
| Root cause | Product name "Trail Jacket" dominated routing; "refund" and "wrong size" were unowned by any returns-policy head |

The shopper explicitly said "not a shipping question" yet the bot returned shipping info. The unowned check caught this because refund/cancel intent tokens had no dedicated routing path — they fell through to the product-name matcher.

---

## Severity story

When Line 3 hits the bot during sale week, the shopper sees shipping times instead of the refund window. They assume returns aren't possible, abandon the cart, and the store loses the sale. CX gets a complaint ticket that takes 4 minutes to resolve manually — multiplied across hundreds of similar questions during peak traffic.

---

## Call

Ship with conditions — CX lead Priya adds a dedicated refund/cancel word watch before any product-name matching runs, tested against "refund for wrong size on the Trail Jacket."

---

## Tripwire

Re-run Every week until launch, then monthly once it's live. If the deciding check drifts more than one step from this ruling → Marisol and Bram, plus you since the bad routing lands on your desk.

---

## Time cost

| Phase | Duration |
|-------|----------|
| Packet assembly | 3 min |
| Five-check walk | 8 min |
| Measurement computation | 4 min |
| Severity write-up | 2 min |
| Call + tripwire | 2 min |
| **Total cycle time** | **19 min** |

---

## Verdict

**Deciding check:** Unowned (rated 4)  
**Position:** Hold my rating — I'll defend it  
**Ship status:** Conditional — pending Priya's refund/cancel word watch implementation and verification against Line 3
