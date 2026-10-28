**Goal:**
Stop POST /orders/{id}/cancel from accepting an order already in the DELIVERED state. A delivered order must be refused with HTTP 400 and must not be cancelled.

**Context:**
The order state machine in app/modules/orders/service.py treats DELIVERED as terminal: it has no valid transition out of DELIVERED. The current cancel_order() logic only rejects orders already marked CANCELLED, so a DELIVERED order can still be mutated to CANCELLED by the cancel endpoint. The route in app/modules/orders/routes.py converts OrderOwnershipError into HTTP 400, which is the expected failure path for invalid cancellation attempts.

**In scope:**
- Enforce the terminal-state rule for DELIVERED orders in the orders cancellation flow.
- Ensure POST /orders/{id}/cancel returns HTTP 400 when the target order status is DELIVERED.
- Preserve the order record unchanged after the rejected cancellation attempt.
- Ensure any associated invoice remains untouched when the cancellation is rejected.
- Keep behavior aligned with the documented state machine and existing route error handling.
- Add or update targeted tests covering the delivered-order rejection path.

**Out of scope:**
- alembic/
- .github/
- pyproject.toml
- any new dependency
- unrelated order features, user authentication changes, or payment workflow changes
- broad refactors outside the cancellation validation bug

**Acceptance:**
- Given an order whose status is DELIVERED, a POST /orders/{id}/cancel request returns HTTP 400.
- The response body includes an error message indicating the cancellation is invalid for the current order state.
- The DELIVERED order remains DELIVERED after the failed cancellation request; no status mutation occurs.
- Any associated invoice remains untouched after the failed cancellation request; no invoice cancellation or payment-side mutation occurs.
- A non-delivered order that is valid to cancel still cancels successfully through the same endpoint.
- The behavior is covered by a focused automated test that exercises the delivered-order rejection case.

**Stop and ask a human if:**
- The business rule should allow cancellation of DELIVERED orders in some exceptional case or for a specific actor.
- There is ambiguity about whether cancellation should be blocked only at DELIVERED or also for other terminal states such as SHIPPED or CANCELLED.
- The fix would require changing the meaning of the order lifecycle or introducing a policy exception outside the current state machine.

**Reviewer:**
Maintainer or designated project reviewer for issue #2.
