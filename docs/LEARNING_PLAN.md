# UCP Learning Plan

## Phase 1: Foundations (Week 1-2)

### Week 1: Understand the Protocol Stack

**Day 1-2: Read the Specs**
- [ ] UCP Overview: https://ucp.dev/specification/overview/
- [ ] UCP Checkout: https://ucp.dev/specification/checkout-rest/
- [ ] AP2 Core Concepts: https://ap2-protocol.org/topics/core-concepts/
- [ ] A2A Key Concepts: https://a2a-protocol.org/latest/topics/key-concepts/

**Day 3-4: Run the Samples**
- [ ] Clone https://github.com/Universal-Commerce-Protocol/samples
- [ ] Set up Python environment
- [ ] Run flower shop demo (business server)
- [ ] Run agent client against flower shop
- [ ] Document what each component does

**Day 5-7: Trace a Transaction**
- [ ] Follow a complete checkout flow
- [ ] Understand the discovery → negotiation → checkout → complete cycle
- [ ] Map the JSON payloads at each step
- [ ] Note questions and gaps in understanding

### Week 2: Build the Business Side

**Day 1-3: Mock Merchant**
- [ ] Create minimal UCP-compliant server
- [ ] Implement `/.well-known/ucp` manifest
- [ ] Implement `POST /checkout-sessions`
- [ ] Implement `GET /checkout-sessions/{id}`
- [ ] Implement `POST /checkout-sessions/{id}/complete`

**Day 4-5: Payment Handler**
- [ ] Understand payment handler abstraction
- [ ] Implement mock payment handler (no real money)
- [ ] Understand AP2 Intent Mandate vs Cart Mandate

**Day 6-7: Testing**
- [ ] Test with UCP playground if available
- [ ] Test with custom agent client
- [ ] Document the implementation pattern

---

## Phase 2: Agent Side (Week 3-4)

### Week 3: Build a Shopping Agent

**Day 1-2: Discovery**
- [ ] Build agent that fetches `/.well-known/ucp`
- [ ] Parse capabilities and negotiate intersection
- [ ] Handle versioning logic

**Day 3-4: Checkout Flow**
- [ ] Create checkout sessions
- [ ] Update cart (add/remove items)
- [ ] Apply discounts if supported
- [ ] Handle fulfillment options

**Day 5-7: Payment**
- [ ] Understand tokenization flow
- [ ] Implement mock payment credential handling
- [ ] Complete checkout and receive order confirmation

### Week 4: Clara Integration

**Day 1-3: Server Setup**
- [ ] Linux server configured
- [ ] Clara base system running
- [ ] MCP tools connected

**Day 4-5: Shopping Capability**
- [ ] Give Clara UCP discovery tools
- [ ] Implement Intent Mandate pattern (pre-authorized spend limits)
- [ ] Test with mock merchant

**Day 6-7: End-to-End**
- [ ] Clara discovers merchant
- [ ] Clara creates cart
- [ ] Clara completes purchase (mock)
- [ ] Clara reports order status

---

## Phase 3: Real World (Week 5-6)

### Week 5: Production Patterns

- [ ] Error handling and edge cases
- [ ] Webhook implementation for order updates
- [ ] Identity linking (OAuth 2.0)
- [ ] Logging and observability

### Week 6: Documentation & Demo

- [ ] Write implementation guide from experience
- [ ] Create demo video showing Clara shopping
- [ ] Package mock merchant as template
- [ ] Identify consulting pitch points

---

## Success Criteria

By end of Week 6:
1. Can explain UCP to a business owner in 5 minutes
2. Can implement basic UCP compliance for a merchant
3. Can build an agent that shops across UCP merchants
4. Have Clara making (mock) purchases
5. Have documented, reusable code patterns
