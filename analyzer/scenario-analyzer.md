# Store FAQ Bot Routing Analyzer

Machine-readable analyzer for auditing whether the store FAQ bot correctly routes shopper questions to the right help center answer—especially when product names appear alongside refund/cancel intent.

---

## Domain

**System under audit:** Store FAQ bot that picks an answer from the help center  
**Failure mode:** Shoppers ask about refunds, but the FAQ bot answers with shipping times because it latched onto the product name.  
**Stakes:** Shoppers get the wrong policy and leave the cart

---

## Input Schema

```yaml
analyzer_input:
  shopper_question:
    type: string
    description: Raw question from store help-desk chat logs
    example: "how long do i have to return the Nova Buds after they ship"
  bot_selected_faq:
    type: string
    description: The FAQ article ID or title the bot chose
  bot_response:
    type: string
    description: The answer text the bot returned to the shopper
  actual_intent:
    type: enum
    values: [refund, cancel, shipping, sizing, warranty, other]
    description: Ground-truth intent labeled by reviewer
```

---

## Check Definitions

| Check ID | Name | What It Measures | Threshold |
|----------|------|------------------|-----------|
| `unowned` | Unowned intent | Whether the shopper's actual ask (refund, cancel) has a dedicated routing path before product-name matching runs | Rating ≥ 3 to pass |
| `copies` | Duplicate routes | How many FAQ articles compete for the same intent when a product name is present | ≤ 1 competing article |
| `room` | Headroom | Gap between confidence score for correct FAQ vs. next-best match | ≥ 0.15 confidence delta |
| `stitch` | Stitch coherence | Whether the bot's answer addresses the question type, not just the product mentioned | Intent-match = true |
| `ablation` | Ablation delta | Performance change when product-name signal is zeroed before routing | Δ accuracy ≤ 0.05 |

---

## Measurement Functions

### mass_across_boundary

```python
def mass_across_boundary(routing_scores: dict, intent_boundary: str) -> float:
    """
    Compute how much routing weight crosses from refund/cancel intent 
    into shipping/product-info territory.
    
    Returns: float between 0.0 (no leakage) and 1.0 (full leakage)
    """
    refund_cancel_mass = sum(routing_scores.get(k, 0) for k in ['refund', 'cancel', 'return'])
    shipping_product_mass = sum(routing_scores.get(k, 0) for k in ['shipping', 'delivery', 'tracking'])
    
    if refund_cancel_mass + shipping_product_mass == 0:
        return 0.0
    
    return shipping_product_mass / (refund_cancel_mass + shipping_product_mass)
```

### max_cross_similarity

```python
def max_cross_similarity(embedding_matrix: np.ndarray) -> float:
    """
    L2-normalize rows, compute similarity matrix, return max off-diagonal.
    High values indicate FAQ articles are too similar to disambiguate.
    
    Threshold: flag if > 0.85
    """
    normalized = embedding_matrix / np.linalg.norm(embedding_matrix, axis=1, keepdims=True)
    similarity = normalized @ normalized.T
    np.fill_diagonal(similarity, 0)
    return similarity.max()
```

### intent_entropy

```python
def intent_entropy(routing_distribution: list) -> float:
    """
    Entropy of routing confidence across FAQ candidates.
    Low entropy = confident routing. High entropy = confused routing.
    
    Threshold: flag if > 1.5 (too uncertain)
    """
    probs = np.array(routing_distribution)
    probs = probs / probs.sum()
    return -np.sum(probs * np.log2(probs + 1e-10))
```

### ablation_delta

```python
def ablation_delta(question: str, bot_router) -> float:
    """
    Zero the product-name signal, re-route, measure accuracy change.
    
    Threshold: flag if Δ > 0.05
    """
    original_route = bot_router.route(question)
    masked_question = mask_product_names(question)  # "Nova Buds" → "[PRODUCT]"
    ablated_route = bot_router.route(masked_question)
    
    return abs(original_route.confidence - ablated_route.confidence)
```

---

## Specimen Sentences for Testing

Source: Store help-desk chat logs

```
how long do i have to return the Nova Buds after they ship
Nova Buds delivery says Friday — can i still cancel
refund for wrong size on the Trail Jacket, not a shipping question
```

---

## Pass Bar

> The answer matches the shopper's real ask — not a nearby FAQ about the same product

A routing decision passes when:
1. The selected FAQ addresses the shopper's stated intent (refund, cancel, sizing)
2. Product-name presence does not override intent signals
3. The bot does not default to shipping/delivery FAQ when refund/cancel words appear

---

## Deciding Check

**Top crack:** `unowned`

The unowned check is the deciding factor because refund/cancel intent currently has no dedicated routing path that fires before product-name matching. When a shopper writes "refund for wrong size on the Trail Jacket," the bot sees "Trail Jacket" and routes to Trail Jacket shipping FAQ instead of the refund policy.

---

## Output Schema

```yaml
analyzer_output:
  question_id: string
  shopper_question: string
  measurements:
    unowned:
      rating: integer  # 1-5
      has_dedicated_path: boolean
    copies:
      rating: integer
      competing_articles: integer
    room:
      rating: integer
      confidence_delta: float
    stitch:
      rating: integer
      intent_match: boolean
    ablation:
      rating: integer
      delta: float
  pass: boolean
  severity: string  # "blocks", "degrades", "acceptable"
  notes: string
```

---

## Ship Condition

Ship with conditions — CX lead Priya adds a dedicated refund/cancel word watch before any product-name matching runs, tested against "refund for wrong size on the Trail Jacket."

---

## Tripwire

Re-run Every week until launch, then monthly once it's live. If the deciding check drifts more than one step from this ruling → Marisol and Bram, plus you since the bad routing lands on your desk.

---

## Integration

This analyzer imports measurement definitions from `specs/measurements.md` and validates against thresholds defined in `specs/scenario-audit.spec.json`. Run cycles are logged in `runs/review-cycle-01.md`.
