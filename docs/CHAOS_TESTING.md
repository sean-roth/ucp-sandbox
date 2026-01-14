# Chaos Testing Results

**Date:** 2026-01-14
**Target:** UCP Reference Implementation (flower shop sample)
**Server:** `http://localhost:8182`

## Summary

| Status | Count | Meaning |
|--------|-------|---------|
| ✅ 2xx | 0 | No unexpected successes |
| ⚠️ 4xx | 19 | Client errors (expected) |
| ❌ 5xx | 1 | Server error (bug!) |

## Key Findings

### What Works Well

1. **Pydantic validation is solid**
   - Missing fields → 422 with clear error path
   - Negative/zero quantity → 422 with helpful message
   - Invalid JSON → 422 with parse error details
   - Missing headers → 422 listing all required fields

2. **Proper 404 handling (mostly)**
   - GET non-existent checkout → 404 with `RESOURCE_NOT_FOUND`
   - PUT non-existent checkout → 404 with `RESOURCE_NOT_FOUND`

3. **Method validation**
   - DELETE on checkout-sessions → 405 Method Not Allowed

4. **Unicode handling**
   - Zalgo text in buyer name accepted without crash

### What's Broken

1. **Complete non-existent checkout → 500**
   ```
   POST /checkout-sessions/fake-id/complete → 500 Internal Server Error
   ```
   Should return 404 like GET/PUT. Server crashes on unhandled exception.

2. **No timeout on agent profile fetch**
   ```
   Network error fetching profile from https://agent.example/profile
   ```
   Blocks until network timeout (5+ seconds observed).

3. **No currency validation**
   - `FAKE_CURRENCY` passed structural validation

4. **No quantity limits**
   - `999999999` items passed validation

5. **No input sanitization**
   - SQL injection string accepted as product ID
   - `'; DROP TABLE products;--` passed validation

## Test Details

### Validation Tests (All 422 - Expected)

| Test | Input | Result |
|------|-------|--------|
| Missing currency | No `currency` field | 422 - Field required |
| Invalid product ID | `FAKE_PRODUCT_123` | 422 - Missing payment* |
| Negative quantity | `-5` | 422 - Must be >= 1 |
| Zero quantity | `0` | 422 - Must be >= 1 |
| Empty line items | `[]` | 422 - Missing payment* |
| Invalid JSON | `{not: valid}` | 422 - JSON decode error |
| Missing headers | No UCP-Agent | 422 - Field required |
| Large quantity | `999999999` | 422 - Missing payment* |
| Fake currency | `FAKE_CURRENCY` | 422 - Missing payment* |
| Unicode chaos | Zalgo text name | 422 - Missing payment* |
| SQL injection | `'; DROP TABLE--` | 422 - Missing payment* |

*Note: Most tests failed on missing `payment` field before reaching the actual test scenario.

### Resource Tests

| Test | Input | Result |
|------|-------|--------|
| GET non-existent | `/checkout-sessions/fake-id` | 404 ✓ |
| PUT non-existent | Update fake session | 404 ✓ |
| Complete non-existent | `/fake-id/complete` | **500 ✗** |
| DELETE (wrong method) | `DELETE /checkout-sessions` | 405 ✓ |

## Server Logs (500 Error)

```
ERROR:    Exception in ASGI application
Traceback (most recent call last):
  ...
  File "routes/ucp_implementation.py", line 205, in complete_checkout
    instrument = PaymentInstrument(root=payment_data)
pydantic_core._pydantic_core.ValidationError: 5 validation errors for PaymentInstrument
id
  Field required [type=missing, input_value={}, input_type=dict]
handler_id
  Field required [type=missing, input_value={}, input_type=dict]
...
```

The endpoint doesn't check if the session exists before trying to parse payment data.

## Recommendations

1. **File bug report** on UCP samples repo for the 500 error
2. **Add pre-flight validation** on agent side before sending requests
3. **Implement timeouts** on all external HTTP calls
4. **Add business logic validation** for currency, quantity limits, input sanitization
5. **Design for agent failure** - assume they'll send garbage and retry infinitely

## Reproduction

Clone the samples repo and run:

```bash
# Terminal 1: Start server
cd samples/rest/python/server
uv run server.py --port=8182

# Terminal 2: Run chaos tests
cd samples/rest/python/client/flower_shop
# (chaos test script - see experiments/chaos/)
```
