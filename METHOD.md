# The Five Checks: PRISM

When auditing whether a setup's checks actually split the work—like a store FAQ bot that picks an answer from the help center—use these five principles to find where intent gets lost.

---

## P — Partition the Space

Does each check own a distinct slice of the input space, or do they overlap and fight?

For an FAQ bot handling shopper questions, this means: Is there a clear boundary between "refund questions" and "shipping questions," or does the bot see "Nova Buds" and route everything to shipping regardless of what the shopper actually asked?

---

## R — Run in Parallel

Can the checks run independently, or does one block another?

If product-name matching runs before intent detection, the bot never gets a chance to notice "refund" or "cancel" in the question. The order matters.

---

## I — Individuate the Pattern

Does each check recognize its own pattern without borrowing from neighbors?

A refund-intent check should fire on "refund for wrong size on the Trail Jacket" without needing the shipping check to fail first. Each check should know its own triggers.

---

## S — Stitch the Spectra

When multiple checks fire, how do they combine?

If both "product name" and "refund" signals are present, which wins? The stitching rule decides whether the shopper gets the right policy or the wrong one.

---

## M — Map What Each Head Sees

Can you trace exactly what each check attended to in a given input?

For "how long do i have to return the Nova Buds after they ship," you should be able to see: Did the bot latch onto "Nova Buds" (product)? Did it see "return" (refund intent)? Did it notice "ship" (shipping mention)? The map shows where attention went.

---

## The Collapse-to-Monochrome Anti-Pattern

When one check dominates and drowns out the others, you get monochrome routing. Every question about a product goes to the same FAQ, regardless of what the shopper actually wanted.

This is what happens when the FAQ bot sees "Nova Buds" and answers with shipping times—even when the shopper asked about returns. The product-name signal collapsed the space; refund intent never got a vote.

The fix: ensure no single check can override the others without explicit priority rules that account for intent signals like "refund," "cancel," or "return."

---

*These five checks appear throughout the audit. Only this file spells out the letters.*
