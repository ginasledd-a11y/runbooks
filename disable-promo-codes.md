Disabling promo codes
Type: Operational procedure — not tied to a specific alert.

Summary
How to turn off promotional discount codes at checkout. This is a fast, self-service mitigation that stops codes being redeemed without a code deploy — useful when a promotion is mispriced, being abused, or implicated in checkout problems.

Promotions are managed in the Promo management dashboard (Growth admin tooling), so disabling a code takes effect immediately across the storefront.

When to use this
A promo code is mispriced or stacking/discounting more than intended.
A code has leaked or is being abused (e.g. a single-use code redeemed en masse).
A promotion is implicated in checkout failures — for example, an incident where checkout is returning errors (CheckoutServerErrors / GatewayHighErrorRate) and the failing orders all carry a promo code. Disabling promotions removes the trigger and restores checkout while engineering works the underlying fix.
This is a mitigation, not a fix. If the failures stem from a code defect in the discount logic, disabling promos buys time — the defect still needs a proper fix and redeploy before promotions are turned back on.

Procedure
Open the Promo management dashboard (Growth admin) and go to Active campaigns.
Identify the codes in play. For a broad mitigation, note all currently active codes (e.g. SAVE5, SAVE10); for a targeted change, just the offending one.
Disable them:
To stop a single promotion, set its status to Inactive.
To stop everything at once, use Disable all active promos.
Confirm the change is live. Changes apply immediately — no deploy is required. The codes stop being accepted at checkout; carts that don't use a code are unaffected throughout.
Verify
In the dashboard, the affected campaigns show Inactive.
A test checkout that applies a now-disabled code is rejected (the discount no longer applies), while a normal checkout without a code still succeeds.
If mitigating an incident: checkout error rate falls as promo orders stop hitting the failing path — CheckoutServerErrors and GatewayHighErrorRate recover. Confirm in Grafana that checkout success returns to normal.
Escalation
Marketing — owns promotional campaigns and the promo management dashboard; pull them in before changing live campaigns where you can.
Checkout / Payments on-call — owns checkout-service; engage them if promotions are disabled as an incident mitigation so the underlying fix is tracked and promos are safely re-enabled afterwards.

