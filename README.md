# Problem-Solver

Great support isn't just closing tickets—it’s understanding the business cost behind every bug. I built this platform to mirror the challenges I love tackling: cutting through incident noise, protecting customer trust during outages, and translating complex system behavior into actionable insights for both clients and core engineering teams.

## Ticket escalation analysis

This PostgreSQL query ranks ticket categories by volume over the last 90 days, calculates average resolution time for valid resolved tickets, and counts distinct affected accounts.

```sql
SELECT
    category,
    COUNT(*) AS ticket_count,
    ROUND(
        AVG(EXTRACT(EPOCH FROM (resolved_at - created_at)) / 3600.0)
            FILTER (
                WHERE resolved_at IS NOT NULL
                  AND resolved_at >= created_at
            )::numeric,
        2
    ) AS avg_resolution_hours,
    COUNT(DISTINCT account_id) AS impacted_account_count
FROM tickets
WHERE created_at >= NOW() - INTERVAL '90 days'
GROUP BY category
ORDER BY ticket_count DESC, category ASC
LIMIT 5;
```

**Metric notes**

- `ticket_count` counts all tickets created in the period, including unresolved ones.
- `avg_resolution_hours` includes only tickets with a resolution timestamp that is not earlier than creation. It is `NULL` when a category has no valid resolved tickets.
- `impacted_account_count` counts distinct non-null `account_id` values. If the table includes non-enterprise accounts, join or filter against the account data before calling this an enterprise-account count.
- This query assumes every row in `tickets` represents an escalation. If escalations are identified by a status or flag, add that condition to the `WHERE` clause.
