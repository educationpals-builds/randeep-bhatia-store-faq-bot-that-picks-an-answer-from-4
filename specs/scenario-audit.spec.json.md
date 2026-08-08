{
  "spec_name": "Store FAQ bot that picks an answer from the help center",
  "description": "Review routine for auditing whether the store FAQ bot routes shopper questions to the correct help-center answer — not a nearby FAQ about the same product.",
  "domain": "e-commerce help-desk routing",
  "standard_line": "The answer matches the shopper's real ask — not a nearby FAQ about the same product",
  "stakes": "Shoppers get the wrong policy and leave the cart",
  "usage_reality": "Short mobile questions with product names in the middle",

  "change_triggers": [
    "Any update to the FAQ bot's retrieval logic or embedding model",
    "New product lines added to the help center",
    "Changes to refund, cancel, or shipping policy content",
    "Priya's refund/cancel word watch rule is modified",
    "Help-center article restructuring or merges"
  ],

  "calendar_floor": {
    "pre_launch": "weekly",
    "post_launch": "monthly",
    "note": "Re-run every week until launch, then monthly once it's live"
  },

  "reviewer_roles": [
    {
      "role": "primary_owner",
      "name": "You",
      "responsibility": "Run the audit cycle; bad routing lands on your desk"
    },
    {
      "role": "drift_alert",
      "names": ["Marisol", "Bram"],
      "responsibility": "Notified if the deciding check drifts more than one step from this ruling"
    },
    {
      "role": "condition_owner",
      "name": "Priya",
      "title": "CX lead",
      "responsibility": "Adds and maintains the dedicated refund/cancel word watch before any product-name matching runs"
    }
  ],

  "sampling_rule": {
    "source": "Store help-desk chat logs",
    "method": "Pull questions where a product name appears mid-sentence and the shopper intent is refund, cancel, or return",
    "minimum_sample_size": 5,
    "priority_filter": "Questions that triggered shipping-related FAQ responses when the ask was about refunds or cancellations"
  },

  "specimen_sentences": [
    "how long do i have to return the Nova Buds after they ship",
    "Nova Buds delivery says Friday — can i still cancel",
    "refund for wrong size on the Trail Jacket, not a shipping question"
  ],

  "checks": {
    "unowned": {
      "description": "Does the bot leave the shopper's actual intent (refund, cancel, return) unaddressed while answering a different question about the same product?",
      "rating": 4,
      "is_deciding_check": true,
      "measurement": "mass_across_boundary",
      "threshold": "≥ 0.35 of retrieval weight crosses from refund/cancel intent to shipping/delivery FAQ",
      "flag_condition": "mass_across_boundary ≥ 0.35"
    },
    "copies": {
      "description": "Do multiple retrieval paths latch onto the same product-name token and return near-duplicate shipping FAQs?",
      "rating": 2,
      "measurement": "max_cross_head_similarity",
      "threshold": "cosine similarity > 0.85 between L2-normalized flattened per-head maps",
      "flag_condition": "max_cross_head_similarity > 0.85"
    },
    "room": {
      "description": "Is there capacity in the retrieval to surface refund/cancel content, or is it crowded out by shipping matches?",
      "rating": 1,
      "measurement": "per_head_entropy",
      "threshold": "entropy < 0.4 relative to uniform distribution indicates collapsed attention",
      "flag_condition": "per_head_entropy < 0.4"
    },
    "stitch": {
      "description": "When the bot stitches together an answer, does it blend refund policy with shipping info in a confusing way?",
      "rating": 2,
      "measurement": "stitch_coherence_score",
      "threshold": "coherence < 0.6 indicates mixed-intent answer",
      "flag_condition": "stitch_coherence < 0.6"
    },
    "ablation": {
      "description": "If we zero out the product-name matching head, does the bot correctly route to refund/cancel FAQ?",
      "rating": 1,
      "measurement": "ablation_delta",
      "threshold": "delta > 0.25 indicates product-name head is overriding intent signal",
      "flag_condition": "ablation_delta > 0.25"
    }
  },

  "deciding_check": "unowned",
  "top_crack_rationale": "The bot leaves refund/cancel intent unowned because it latches onto the product name and routes to shipping FAQ instead",

  "ship_call": {
    "decision": "Ship with conditions",
    "condition": "CX lead Priya adds a dedicated refund/cancel word watch before any product-name matching runs, tested against \"refund for wrong size on the Trail Jacket.\"",
    "test_sentence": "refund for wrong size on the Trail Jacket, not a shipping question",
    "owner": "Priya"
  },

  "watch_tripwire": {
    "cadence_pre_launch": "weekly",
    "cadence_post_launch": "monthly",
    "drift_threshold": "deciding check (unowned) drifts more than one step from this ruling",
    "alert_recipients": ["Marisol", "Bram", "You"],
    "rationale": "Bad routing lands on your desk"
  },

  "attach_or_it_doesnt_count_gate": {
    "required_attachments": [
      "Screenshot or log excerpt showing the shopper question and bot response",
      "Per-check measurement values for that question",
      "Timestamp and session ID from store help-desk chat logs"
    ],
    "rule": "A review cycle without attached evidence for each flagged line does not count as a completed audit"
  },

  "cycle_verdict": "Hold my rating — I'll defend it"
}
