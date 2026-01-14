# UCP Learning Notes

## 2026-01-13: Initial Research

### Key Insight
UCP isn't just a commerce protocol - it's the infrastructure layer for agent-native business. Every business will need this to be discoverable by AI agents.

> "I'm not seeing a product, but a way to improve every product we make from now on."

### The Protocol Stack
```
MCP (Tools) + A2A (Agent-to-Agent) + AP2 (Payments) + UCP (Commerce)
```

These work together:
- **MCP**: How agents connect to tools/capabilities
- **A2A**: How agents talk to other agents  
- **AP2**: How agents handle payment authorization
- **UCP**: How agents conduct commerce with merchants

### Core UCP Concepts

**Discovery**: Merchants publish `/.well-known/ucp` manifest declaring:
- Services they support
- Capabilities they offer (checkout, fulfillment, discounts, etc.)
- Transport bindings (REST, MCP, A2A)
- Payment handlers they accept

**Capability Negotiation**: 
- Agent sends profile URL in request header
- Merchant fetches agent profile
- Compute intersection of capabilities
- Respond with only mutually supported features

**Payment Handlers**: Abstraction that separates:
- What's accepted (instruments)
- How it's processed (handlers)
- Agents never touch raw payment data - only tokens

**AP2 Mandates**: Cryptographic proof of authorization
- Intent Mandate: Pre-authorized spend limits (for autonomous agents)
- Cart Mandate: Specific authorization for exact cart (human-present)
- Payment Mandate: Signals to payment network about agent involvement

### UCP vs AP2 - The Key Distinction

**UCP can work without AP2** - but only for "human-present" transactions.

**AP2 is required for autonomy** - agents acting without human confirmation.

| Scenario | UCP | AP2 | Human Involvement |
|----------|-----|-----|-------------------|
| You browse, agent checks out | ✓ | Optional | You approve each purchase |
| Agent shops, you approve cart | ✓ | Cart Mandate | You review before payment |
| Agent shops and buys autonomously | ✓ | Intent Mandate | Set limits once, walk away |

**Bottom line:** UCP = how agents and merchants talk. AP2 = how agents prove they're authorized to spend.

When consulting: Add AP2 so businesses can handle fully autonomous agents, not just assisted shopping.

### What "UCP-Ready" Means for a Business

Minimum requirements:
1. `/.well-known/ucp` - JSON manifest with capabilities
2. `POST /checkout-sessions` - Create cart
3. `GET /checkout-sessions/{id}` - Retrieve cart
4. `PATCH /checkout-sessions/{id}` - Update cart
5. `POST /checkout-sessions/{id}/complete` - Finalize purchase
6. Payment handler configuration
7. HTTPS everywhere
8. Signing keys for webhooks

### Google's Product Restrictions (Important!)

Google's implementation ≠ the protocol. Their Merchant Center has strict eligibility:

**NOT eligible for Google AI checkout:**
- Subscriptions (recurring billing)
- Services (lessons, classes, travel)
- Digital goods (software, virtual items)
- Rentals
- Pre-orders
- Customized/personalized goods
- Age-restricted items

**This means:** Compel English and VFX Buddy won't be in Google's agent shopping. But other platforms implementing UCP may have different rules. B2B purchasing agents don't follow Google Shopping policy.

### Business Opportunities Identified

1. **Consulting**: Get businesses UCP-ready ($15-25K per engagement)
2. **Integration patterns**: Reusable templates/middleware
3. **Testing tools**: UCP compliance validation
4. **Agent infrastructure**: Purchasing agents for specific verticals
5. **Upsell pathway**: UCP integration → SOPs Nobody Reads training
6. **Compliance Agent**: Monitor risk signals, flag fraud (see chaos testing notes)

### Application Ideas

- **Clara**: Personal purchasing agent with Intent Mandates
- **Compel English**: UCP-enabled for non-Google agent platforms
- **VFX Buddy**: Agent-discoverable subscription management
- **Procurement agent**: For manufacturing (ConexSmart-style operations)

### Key Dates

- **Jan 11, 2026**: Google releases UCP
- **First-mover window**: ~6-12 months before commoditization

---

## 2026-01-14: Hands-On Testing

### Morning: Happy Path

Ran the flower shop sample. Nine seconds, seven steps:

```
GET  /.well-known/ucp              → Discovery
POST /checkout-sessions            → Create cart (3500 cents)
PUT  /checkout-sessions/{id}       → Add item (6500 cents)
PUT  /checkout-sessions/{id}       → Apply discount (5850 cents)
PUT  /checkout-sessions/{id}       → Set fulfillment
POST /checkout-sessions/{id}/complete → Payment
```

Four endpoints. Standard REST. The protocol isn't the hard part.

### Afternoon: Chaos Testing

Wrote a chaos script to break things. Found:

**1. Reference implementation has bugs**
- `POST /complete` on non-existent session → 500 (should be 404)
- Server crashes, doesn't handle gracefully

**2. External dependencies in hot path**
- Agent profile fetch blocks checkout
- No timeout = 5+ second hangs
- Single point of failure

**3. Schema strictness is brutal**
- Every test failed on missing `payment` field first
- Can't even test invalid products - fails on schema before business logic
- Agents must send perfectly formed requests or get nothing useful

**4. No business logic validation**
- `FAKE_CURRENCY` accepted
- `999999999` quantity accepted  
- SQL injection string accepted as product ID
- Pydantic validates shape, not sanity

### Key Insight

> "If Google's own sample code has unhandled exceptions, imagine what businesses will ship."

The reference implementation is a starting point, not production code.

### The Schema Checkpoint Pattern

Agents are bad at error recovery. Solution: **validate before sending**.

```python
def pre_flight_check(request: dict) -> tuple[bool, list[str]]:
    """Validate request will pass UCP schema before sending."""
    errors = []
    if "currency" not in request:
        errors.append("Missing: currency")
    if "payment" not in request:
        errors.append("Missing: payment")
    # ... business logic checks ...
    return (len(errors) == 0, errors)
```

Trade-off: Slower checkout. But it works every time. People wait for payments anyway - banks run on COBOL from 1965.

### Payment Processor Note: Paddle

Researched Paddle for Compel English. **Not recommended:**
- Not on UCP partner list
- $5M FTC settlement (June 2025) for enabling tech support scams
- Their MoR model conflicts with UCP's "you remain merchant of record"

**Better choice:** Stripe - already UCP endorsed, handles tax, you control the relationship.

### Questions Answered

- [x] What's the minimum viable UCP implementation? → 4 endpoints + manifest
- [x] How does error recovery work mid-checkout? → Poorly. Design defensively.

### Questions Remaining

- [ ] How does capability extension inheritance work in practice?
- [ ] What's the actual UX of the Intent Mandate signing?
- [ ] How do webhooks work for order updates?

### Next Steps

1. Build mock merchant from scratch (not copying sample)
2. Implement the pre-flight validation pattern
3. Add proper timeouts and circuit breakers
4. Test against Clara when server infrastructure ready

See also:
- [CHAOS_TESTING.md](./CHAOS_TESTING.md) - Full test results
- [LESSONS_LEARNED.md](./LESSONS_LEARNED.md) - Implementation checklist
