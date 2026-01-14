# UCP Implementation Lessons

Hard-won knowledge from breaking the reference implementation. Read this before building anything.

## The Core Truth

> "If Google's own sample code has unhandled exceptions, imagine what businesses will ship."

The reference implementation is a **starting point**, not production code. Copying it wholesale will hurt you.

---

## What We Broke

### 1. Complete on Non-Existent Session → 500 Error

```
POST /checkout-sessions/fake-id/complete → 500 Internal Server Error
```

GET and PUT correctly return 404. Complete crashes the server. The endpoint doesn't check if the session exists before trying to process payment.

**Lesson:** Wrap every endpoint in try/catch. Trust nothing.

### 2. External Dependencies Block the Hot Path

```
Network error fetching profile from https://agent.example/profile
```

The server synchronously fetches agent profiles during checkout. If that URL is slow or down, your checkout hangs. We saw 5+ second delays until adding a 1-second timeout.

**Lesson:** Timeout everything. Default httpx has no timeout. Add `timeout=1.0` minimum.

### 3. Schema Validation Blocks Everything Else

Every chaos test failed on missing `payment` field before testing the actual scenario:

```json
{
  "detail": [{"type": "missing", "loc": ["body", "payment"], "msg": "Field required"}]
}
```

Want to test invalid product IDs? Can't - fails on missing payment first.
Want to test bad currency? Can't - fails on missing payment first.

**Lesson:** Pydantic validates structure exhaustively before your code runs. Agents must send complete, well-formed requests or get nothing useful back.

### 4. No Business Logic Validation

These all passed structural validation:
- `FAKE_CURRENCY` - no ISO 4217 check
- `999999999` quantity - no upper limit
- `'; DROP TABLE products;--` - no sanitization
- Zalgo text in buyer name - no normalization

**Lesson:** Pydantic validates shape, not sanity. Add your own business rules.

---

## The Schema Checkpoint Pattern

Agents are bad at error recovery. They either retry infinitely, sit idle, or hallucinate success.

**Solution:** Validate before sending.

```python
# Before submitting to UCP endpoint
def pre_flight_check(checkout_request: dict) -> tuple[bool, list[str]]:
    """Validate request will pass UCP schema before sending."""
    errors = []
    
    # Required fields
    if "currency" not in checkout_request:
        errors.append("Missing required field: currency")
    if "payment" not in checkout_request:
        errors.append("Missing required field: payment")
    if "line_items" not in checkout_request:
        errors.append("Missing required field: line_items")
    
    # Business logic
    if checkout_request.get("currency") not in VALID_CURRENCIES:
        errors.append(f"Invalid currency: {checkout_request.get('currency')}")
    
    for i, item in enumerate(checkout_request.get("line_items", [])):
        qty = item.get("quantity", 0)
        if qty < 1 or qty > 10000:
            errors.append(f"Item {i}: quantity {qty} outside valid range 1-10000")
    
    return (len(errors) == 0, errors)

# Usage
valid, errors = pre_flight_check(request)
if not valid:
    # Handle gracefully, don't send bad request
    logger.error(f"Pre-flight failed: {errors}")
    return None
    
# Now safe to send
response = await client.post("/checkout-sessions", json=request)
```

**Trade-off:** Slower checkout (two validation passes). But it works every time. People wait for payments anyway - the banks are running COBOL from 1965.

---

## Implementation Checklist

Before deploying UCP for any client:

### Error Handling
- [ ] Wrap all endpoints in try/catch
- [ ] Return proper HTTP codes (404 not 500) for missing resources
- [ ] Log full stack traces for debugging
- [ ] Never expose internal errors to clients

### Timeouts & Circuit Breakers
- [ ] Set explicit timeout on all external HTTP calls (1-5 seconds)
- [ ] Implement circuit breaker for agent profile fetches
- [ ] Graceful degradation when dependencies fail
- [ ] Health check endpoint that verifies dependencies

### Validation
- [ ] Pre-flight schema validation before sending requests (agent side)
- [ ] Currency codes against ISO 4217
- [ ] Quantity limits (min 1, max reasonable for business)
- [ ] Product ID format validation
- [ ] Sanitize all inputs before database queries

### Idempotency
- [ ] Generate and track idempotency keys
- [ ] Handle duplicate requests gracefully
- [ ] Clean up zombie sessions (started but never completed)

### Monitoring
- [ ] Track checkout success/failure rates
- [ ] Alert on 500 error spikes
- [ ] Monitor external dependency latency
- [ ] Log enough to debug distributed failures

### Payment Processor Protection
- [ ] Monitor authorization failure rates
- [ ] Rate limit per-agent to prevent retry storms
- [ ] Track and alert on chargeback patterns

---

## What Agents Will Do Wrong

Expect agents to:
1. **Forget required fields** - they'll send partial requests
2. **Retry infinitely** - no backoff, hammering your endpoint
3. **Timeout poorly** - hang for hours on slow responses
4. **Ignore errors** - proceed as if things worked
5. **Send garbage** - invalid currencies, impossible quantities

Design defensively. Assume the agent is having a bad day.

---

## Reference Implementation Bugs Found

File issues or fix locally:

1. **`complete_checkout` doesn't check session exists** - Returns 500 instead of 404
2. **No timeout on agent profile fetch** - Blocks indefinitely
3. **No currency validation** - Accepts any string
4. **No quantity limits** - Accepts 999999999
5. **No product ID sanitization** - SQL injection passes validation

These are learning opportunities, not criticisms. The samples are for understanding the protocol, not production deployment.
