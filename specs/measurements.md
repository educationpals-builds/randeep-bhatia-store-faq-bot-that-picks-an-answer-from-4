# Measurements for Store FAQ Bot Routing Audit

This document defines each per-check measurement used to verify that the store FAQ bot picks an answer from the help center that matches the shopper's real ask — not a nearby FAQ about the same product.

---

## 1. Mass-Across-Boundary (Unowned Check)

**What it measures:** How much attention weight crosses from the shopper's intent words (refund, return, cancel) to the product-name tokens (Nova Buds, Trail Jacket) versus staying on the intent.

**How to compute:**
1. Extract the attention matrix from the routing layer.
2. Identify intent-word token positions and product-name token positions.
3. Sum attention weights flowing from intent tokens to product-name tokens.
4. Divide by total attention weight originating from intent tokens.

**Threshold:** Flag if mass-across-boundary > 0.35 (more than 35% of intent attention leaks to product names).

**Example from store help-desk chat logs:**
- Input: "refund for wrong size on the Trail Jacket, not a shipping question"
- If "refund" and "wrong size" send > 35% of their attention to "Trail Jacket," the bot may latch onto product-name FAQs instead of refund policy.

---

## 2. Max Cross-Head Similarity (Copies Check)

**What it measures:** Whether multiple attention heads are doing redundant work — copying the same pattern instead of splitting duties.

**How to compute:**
1. For each attention head, flatten the per-head attention map into a 1D vector.
2. L2-normalize each flattened vector.
3. Stack all normalized vectors into a matrix.
4. Compute similarity matrix: matmul(stack, stack.transpose()).
5. Read the off-diagonal entries; take the maximum.

**Threshold:** Flag if max off-diagonal similarity > 0.80 (two heads are 80%+ redundant).

**Example from store help-desk chat logs:**
- Input: "Nova Buds delivery says Friday — can i still cancel"
- If head 2 and head 5 both attend identically to "Nova Buds" and "Friday," they're copying rather than splitting cancel-intent from delivery-status.

---

## 3. Per-Head Entropy vs. Uniform (Room Check)

**What it measures:** Whether each head's attention distribution is too peaked (no room for nuance) or appropriately spread.

**How to compute:**
1. For each attention head, compute the entropy of its attention distribution over tokens.
2. Compute the entropy of a uniform distribution over the same token count.
3. Calculate ratio: head_entropy / uniform_entropy.

**Threshold:** Flag if ratio < 0.25 (head attends to fewer than 25% of tokens' worth of spread — too narrow).

**Example from store help-desk chat logs:**
- Input: "how long do i have to return the Nova Buds after they ship"
- If a head puts 95% attention on "Nova Buds" alone, entropy ratio drops below 0.25, leaving no room to also weigh "return" and "how long."

---

## 4. Stitch Consistency (Stitch Check)

**What it measures:** Whether the routing decision stitches together intent + product correctly, or drops one when combining.

**How to compute:**
1. Run the input through the routing layer.
2. Record which FAQ category is selected.
3. Check if the selected category matches the intent keyword (refund → refund policy, cancel → cancellation policy, return → return policy).
4. Score: 1 if match, 0 if mismatch.

**Threshold:** Flag if stitch score = 0 (the bot stitched to the wrong category).

**Example from store help-desk chat logs:**
- Input: "refund for wrong size on the Trail Jacket, not a shipping question"
- If the bot routes to "Trail Jacket shipping times" instead of "refund policy," stitch score = 0.

---

## 5. Ablation Delta (Ablation Check)

**What it measures:** How much the routing decision changes when you zero out one attention head before the final concat — reveals which head is actually doing the work.

**How to compute:**
1. Run the input normally; record the output logits for each FAQ category.
2. Zero out one attention head's contribution before the concat layer.
3. Re-run; record new output logits.
4. Compute delta: L2 norm of (original_logits - ablated_logits).
5. Repeat for each head.

**Threshold:** Flag if max ablation delta < 0.10 (no single head matters enough — work isn't split, it's diffuse) OR if one head's delta > 0.70 (one head does everything — no split at all).

**Example from store help-desk chat logs:**
- Input: "Nova Buds delivery says Friday — can i still cancel"
- If zeroing head 3 changes the routing from "shipping" to "cancellation" with delta = 0.85, that head alone controls the decision — ablation delta flags it.

---

## Summary Table

| Check    | Measurement                  | Threshold                     |
|----------|------------------------------|-------------------------------|
| Unowned  | Mass-across-boundary         | > 0.35 flags                  |
| Copies   | Max cross-head similarity    | > 0.80 flags                  |
| Room     | Per-head entropy vs. uniform | < 0.25 flags                  |
| Stitch   | Stitch consistency score     | = 0 flags                     |
| Ablation | Ablation delta               | < 0.10 or > 0.70 flags        |

---

## Pass Bar

The answer matches the shopper's real ask — not a nearby FAQ about the same product.

All measurements feed into this standard. A line passes only when no measurement crosses its flag threshold.
