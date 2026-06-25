---
title: Notification System Design
slug: notification-system-design
destination: my-saas-app
status: complete
created: 2025-01-15
updated: 2025-01-17
---

## Intent
Design a notification system for a multi-tenant SaaS application. Must support
push notifications, in-app notifications, and email digests. Success criteria:
notifications are reliable, preferences are user-controlled, and the system
doesn't couple to business logic.

## Decisions Locked

1. **Delivery architecture** — **Principle:** "All notifications flow through an
   async event bus. Business logic emits events; it never calls notification
   endpoints directly. This decouples delivery from domain logic and allows
   retries without re-executing business operations."

2. **Channel model** — **Principle:** "Three first-class channels: push, in-app,
   and email-digest. Each channel has an independent delivery pipeline. A
   notification event may fan out to multiple channels based on user preferences."

3. **User preference ownership** — **Principle:** "Users own their notification
   preferences at the category level (e.g., 'billing', 'team activity'), not the
   event level. Each category maps to a default channel set that users can
   override. The system never sends to a channel the user has disabled for that
   category."

4. **Tenant isolation** — **Principle:** "Notification routing is tenant-scoped.
   A tenant admin can disable categories org-wide, and user preferences layer on
   top. Tenant-level overrides win on disable; user preferences win on channel
   selection within enabled categories."

5. **Delivery guarantees** — **Principle:** "In-app notifications are
   at-least-once with client-side deduplication via idempotency keys. Push and
   email are best-effort with retry (3 attempts, exponential backoff). Failed
   deliveries are logged but do not block or retry indefinitely."

### Audit (after Decision 5)
- 2025-01-16: No contradictions found. Noted: need to decide on the event
  schema before defining notification categories. Added as next decision.
  Checked for authority leaks — tenant admin vs. user boundary is clean.

6. **Event schema** — **Principle:** "Notification events use a flat envelope:
   `{event_type, tenant_id, user_id, category, payload, idempotency_key,
   timestamp}`. The payload is channel-agnostic; each channel's renderer
   transforms it. No nested objects in the envelope — renderers pull what they
   need from payload."

7. **Rendering strategy** — **Principle:** "Each channel owns its renderer.
   Renderers are pure functions: `(event_payload, user_locale) → channel_output`.
   No shared rendering logic across channels — duplication is acceptable to keep
   channels independently deployable."

8. **Rate limiting** — **Principle:** "Rate limits are per-user-per-channel, not
   global. Defaults: push max 10/hour, email-digest max 1/day, in-app unlimited
   (UI handles volume). Tenant admins can adjust push and email limits within
   platform-defined bounds."

9. **Observability** — **Principle:** "Every notification lifecycle event
   (queued, sent, delivered, failed, opened) is logged with the idempotency key
   as correlation ID. A notification health dashboard shows delivery rates,
   failure rates, and p95 delivery latency per channel per tenant."

10. **Migration strategy** — **Principle:** "Existing direct-send calls migrate
    to event emission behind a feature flag. Both paths run in parallel during
    migration (shadow mode). The flag flips per-tenant. Migration is complete
    when all tenants are on the event path and the direct-send code is deleted —
    no permanent abstraction layer."

### Audit (after Decision 10)
- 2025-01-17: Full design audit. All 10 decisions are consistent. Checked:
  tenant isolation holds through the full pipeline. Rate limiting doesn't
  conflict with digest batching (digests are a separate scheduled job, not
  subject to per-message rate limits). Migration shadow mode needs a comparison
  log — added to spec. No authority leaks. Design is complete.

## ▶ Next Decision
None — design complete.

## Current Spec (living)

### Architecture
- Event bus (async) receives notification events from business logic
- Fan-out service reads user preferences and routes to channel pipelines
- Each channel pipeline: renderer → delivery → logging
- Idempotency key ensures at-least-once without duplicates

### Data Model
- `NotificationEvent`: event_type, tenant_id, user_id, category, payload,
  idempotency_key, timestamp
- `UserPreferences`: user_id, category → enabled channels
- `TenantOverrides`: tenant_id, category → enabled/disabled
- `DeliveryLog`: idempotency_key, channel, status, attempts, timestamps

### Channels
- **Push**: FCM/APNs, 3 retries, 10/hr rate limit per user
- **In-app**: WebSocket + REST fallback, at-least-once, client deduplicates
- **Email digest**: Scheduled batch job (default daily), 1/day max

### Migration
- Feature flag per tenant, shadow mode during parallel run
- Comparison log validates event path matches direct-send output
- Delete direct-send code after full rollout

## Audit Log
- 2025-01-16: 5-decision audit. Clean. Added event schema to decision queue.
- 2025-01-17: 10-decision audit. Full design consistent. Added comparison log
  requirement for shadow mode. Design approved.

## Handoff
- **Destination:** my-saas-app
- **Status:** Design complete, ready for implementation planning
- **Next step:** Generate implementation plan from this spec
- **Dependencies:** FCM/APNs credentials, email service provider selection
