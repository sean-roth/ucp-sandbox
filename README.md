# UCP Sandbox

Learning sandbox for the emerging agentic commerce stack:

- **UCP** (Universal Commerce Protocol) - Commerce layer
- **AP2** (Agent Payments Protocol) - Payment authorization layer  
- **A2A** (Agent2Agent Protocol) - Inter-agent communication
- **MCP** (Model Context Protocol) - Tool/capability layer

## Why This Matters

Google released UCP on January 11, 2026. This is the infrastructure layer for AI agents to conduct commerce - discovering merchants, managing carts, handling payments, tracking orders.

Endorsed by: Shopify, Target, Walmart, Stripe, PayPal, Visa, Mastercard, and 20+ other major players.

This is table stakes for the AI economy. Learn it now.

## Key Insight

> "If Google's own sample code has unhandled exceptions, imagine what businesses will ship."

The reference implementation is a starting point, not production code. See [LESSONS_LEARNED.md](docs/LESSONS_LEARNED.md) for what we found breaking it.

## Goals

1. **Understand both sides** - Build agents that shop AND merchants that sell
2. **Integrate with Clara** - Make this real, not theoretical
3. **Document patterns** - Create reusable templates for future projects
4. **Identify opportunities** - Consulting, tooling, products

## Repository Structure

```
ucp-sandbox/
├── docs/
│   ├── LEARNING_PLAN.md      # Week-by-week learning roadmap
│   ├── NOTES.md              # Running notes and observations
│   ├── RESOURCES.md          # Links to specs, repos, docs
│   ├── LESSONS_LEARNED.md    # Hard-won implementation knowledge
│   └── CHAOS_TESTING.md      # Breaking the reference implementation
├── experiments/
│   ├── 01-discovery/         # UCP manifest discovery
│   ├── 02-checkout/          # Checkout session flow
│   ├── 03-payments/          # AP2 payment handling
│   └── 04-agent/             # Full agent implementation
├── mock-merchant/            # Test storefront with UCP endpoints
└── clara-integration/        # Clara shopping capabilities
```

## Quick Links

### Specifications
- [UCP Spec](https://ucp.dev/specification/overview/)
- [AP2 Spec](https://ap2-protocol.org/specification/)
- [A2A Spec](https://a2a-protocol.org/latest/specification/)

### Official Repos
- [UCP Protocol](https://github.com/Universal-Commerce-Protocol/ucp)
- [UCP Samples](https://github.com/Universal-Commerce-Protocol/samples)
- [AP2 Protocol](https://github.com/google-agentic-commerce/AP2)
- [A2A Protocol](https://github.com/a2aproject/A2A)

### Google Implementation
- [Merchant Integration Guide](https://developers.google.com/merchant/ucp)
- [Developer Blog Post](https://developers.googleblog.com/under-the-hood-universal-commerce-protocol-ucp/)

## Status

- [x] Repository created
- [x] Learning plan drafted
- [x] UCP samples running locally
- [x] Happy path tested (9 seconds, 7 steps)
- [x] Chaos testing complete (found bugs!)
- [ ] Mock merchant built (from scratch, not copy)
- [ ] Pre-flight validation pattern implemented
- [ ] Agent discovery working
- [ ] Clara integration complete

## What We've Learned So Far

**The protocol is simple:** 4 REST endpoints + a discovery manifest.

**The implementation is hard:**
- Schema strictness breaks agents that send incomplete requests
- External dependencies (agent profile fetch) block the hot path
- No business logic validation in reference code
- Error handling has gaps (500s where 404s should be)

**The opportunity is real:**
- First-mover window: 6-12 months
- Consulting: $15-25K per implementation
- Most businesses will copy sample code and ship bugs

See [docs/NOTES.md](docs/NOTES.md) for the full learning journal.
