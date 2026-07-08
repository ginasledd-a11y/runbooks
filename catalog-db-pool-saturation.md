Catalog database connection pool saturation
Summary
catalog-service can't acquire database connections because a long-running query is holding an exclusive lock on the products table. Catalog queries queue up behind the lock, the connection pool fills, and the failure cascades up to the gateway as customer-facing errors.

Alerts you'll see
Listed in the order they fire:

Alert	Severity	Meaning
PostgresBlockedQueries	critical	Queries on storefront_db are blocked waiting on a lock (root cause, fires first ~15s)
GatewayHighLatency	warning	Gateway p95 latency is above 2s — general slowness as catalog reads queue (~28s)
PostgresLongRunningTransaction	critical	A transaction has been open >60s, likely holding the lock (~70s)
CatalogDBPoolSaturation	critical	catalog-service DB pool is >80% in-use — the lock has starved catalog of connections (~95s)
GatewayHighErrorRate	critical	Gateway is returning >5% server errors — customer impact (~95s)
The Postgres lock alerts (PostgresBlockedQueries, then PostgresLongRunningTransaction) are the leading, root-cause signals. CatalogDBPoolSaturation and GatewayHighErrorRate are downstream symptoms that confirm the lock has starved catalog of connections and reached customers.

Customer impact
Product listings and product detail pages fail to load or time out. Anything that reads from the catalog is degraded, and checkout steps that depend on catalog reads may fail too.

Diagnosis
The catalog read path is:

gateway → catalog-service → postgres (products table)
Confirm pool saturation. In Grafana, db_connections_open{service="catalog-service", state="in_use"} is at or near the pool maximum and the idle count is ~0.

Find the blocking query. On the primary, look for the statement holding an ACCESS EXCLUSIVE lock on products that everything else is waiting on:

SELECT pid, usename, state, wait_event_type,
       now() - query_start AS running_for, query
FROM pg_stat_activity
WHERE datname = 'storefront_db'
ORDER BY query_start;
The culprit is typically a long-running analytics query, an ad-hoc statement, or a migration that took a heavier lock than expected.

Logs. catalog-service logs show DB query timeouts / pool-exhausted errors.

Root cause
A long-running query took an ACCESS EXCLUSIVE lock on the products table. While that lock is held, normal catalog reads can't proceed, so they hold their connections open waiting — exhausting the pool.

Remediation
Identify the offending backend from the pg_stat_activity query above (the long-running statement holding the lock on products).

Terminate it so the lock releases and queued queries can drain:

SELECT pg_terminate_backend(<pid>);
Cancel (pg_cancel_backend) first if you want a gentler stop; escalate to pg_terminate_backend if it doesn't let go promptly.

Confirm the pool recovers — in_use connections fall and idle connections return.

Stop it recurring this shift. If the query came from a scheduled analytics job or a person running ad-hoc SQL against the primary, pause the job / ask them to stop until you've moved it to a replica.


