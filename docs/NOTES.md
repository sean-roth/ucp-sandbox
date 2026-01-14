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

### Questions to Answer

- [ ] How does capability extension inheritance work in practice?
- [ ] What's the actual UX of the Intent Mandate signing?
- [ ] How do webhooks work for order updates?
- [ ] What's the minimum viable UCP implementation?
- [ ] How does error recovery work mid-checkout?

### Business Opportunities Identified

1. **Consulting**: Get businesses UCP-ready ($15-25K per engagement)
2. **Integration patterns**: Reusable templates/middleware
3. **Testing tools**: UCP compliance validation
4. **Agent infrastructure**: Purchasing agents for specific verticals
5. **Upsell pathway**: UCP integration → SOPs Nobody Reads training

### Application Ideas

- **Clara**: Personal purchasing agent with Intent Mandates
- **Compel English**: UCP-enabled course purchases
- **VFX Buddy**: Agent-discoverable subscription management
- **Grocery price watcher**: Monitor sales, auto-buy deals
- **Hardware price tracker**: Watch for GPU/RAM deals
- **Procurement agent**: For manufacturing (ConexSmart-style operations)

### Key Dates

- **Jan 11, 2026**: Google releases UCP
- **First-mover window**: ~6-12 months before commoditization

---

## Running Notes

*Add dated entries as learning progresses*

### 2026-01-14

Started ucp-sandbox repository. Next step: clone samples repo and get flower shop running.
