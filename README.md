Storefront runbooks
Operational runbooks for the storefront platform. Each runbook covers a specific failure mode: what it looks like, the alerts it fires, how to diagnose it, and how to remediate it.

These are indexed so that incident.io's AI SRE can surface the relevant runbook into the incident channel when a matching alert opens an incident.

Incident runbooks
Triggered when a matching alert opens an incident. Each is listed against the alert that leads the incident, then the follow-on alerts in firing order.

Runbook	Symptom	Lead → follow-on alerts	Severity
Payment provider (Paylinity) degradation	Checkouts slow/failing; third-party payment provider degraded	PaylinityRequestLatency → CheckoutPaymentsLatency → GatewayHighErrorRate	critical
Catalog DB pool saturation	Product pages failing; catalog DB pool exhausted behind a table lock	PostgresBlockedQueries → PostgresLongRunningTransaction → CatalogDBPoolSaturation → GatewayHighErrorRate	critical
Operational runbooks
General how-to procedures, not tied to a specific alert.

Runbook	Use when
Disabling promo codes	A promo code is mispriced, being abused, or implicated in checkout failures and you need to turn promotions off quickly
Architecture context
The storefront is composed of four services behind a gateway, backed by postgres, with payments routed to a third-party provider (Paylinity):

gateway → catalog-service  → postgres
        → checkout-service → payments-service → Paylinity (third-party)
gateway — API reverse proxy; where customer-facing errors and latency surface.
catalog-service — product catalog, backed by postgres.
checkout-service — order processing; calls payments synchronously.
payments-service — integrates with the third-party payment provider.
Alerts are evaluated in Prometheus and routed to incident.io via Alertmanager; metrics and logs are in Grafana (Prometheus + Loki).


