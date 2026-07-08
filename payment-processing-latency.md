Payment provider (Paylinity) degradation
Type: Third-party dependency — vendor escalation playbook.

Summary
Paylinity is our third-party payment provider — an external company whose platform we send card transactions to.

We use them for all customer payments: every order placed on the storefront is charged through Paylinity. If Paylinity is unavailable or degraded, customers can't pay.

Use this runbook when payments are failing or slow and Paylinity is, or might be, the cause — when you need to know who they are, how to reach them urgently, and who owns the relationship internally.

Because they're a third party, there's nothing we can deploy to fix them: the job is to escalate to Paylinity and contain customer impact while they resolve it.

Alerts that point here
Listed in the order they fire:

Alert	Severity	Meaning
PaylinityRequestLatency	critical	payments-service → Paylinity p95 > 3s — the provider is degraded (leads, ~15s)
GatewayHighLatency	warning	Gateway slow as checkout backs up (~28s)
CheckoutPaymentsLatency	critical	The Paylinity latency has reached the checkout path (~48s)
GatewayHighErrorRate	critical	Customers seeing checkout errors (~95s)
PaylinityRequestLatency leads and names the provider directly — that's your cue this is Paylinity, not us.

Confirm it's actually Paylinity
Before escalating to the vendor, sanity-check it's really them and not an internal problem wearing a Paylinity mask:

payments-service → Paylinity latency is elevated while our own services (checkout-service, payments-service) are otherwise healthy — they're blocked waiting on Paylinity, not struggling themselves.
Paylinity's status page shows degradation (or it's quiet but the latency is unambiguous).
If our own services look unhealthy, stop — this isn't a vendor outage; work the relevant service runbook instead.

Contact & escalate Paylinity
Vendor outage means escalation speed matters. For a live customer-impacting incident, hit all of these in parallel — don't wait on one channel:

Channel	Detail
Status page	https://status.paylinity.com
24/7 emergency line	+1 (888) 555-0142 — choose option 2, "live incident"
Urgent escalation email	urgent@paylinity.com — monitored 24/7 for Sev1; cc your TAM
Support portal	https://support.paylinity.com — raise a Priority 1 ticket
Technical account manager	Dana Whitfield — dana.whitfield@paylinity.com / +1 (888) 555-0188
Have ready when you contact them:

Our merchant ID: MID-STOREFRONT-04417 (in 1Password → "Paylinity")
The symptom: charge-API p95 latency, approximate start time, timeout/error rate
The ask: acknowledgement, a provider-side incident ID, and an ETA
SLA: our contract commits Paylinity to a 30-minute response for P1 and 99.95% monthly availability. If they miss the response window, escalate through the TAM and flag the SLA breach to account management.


