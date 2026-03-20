---
type: bmad-distillate
sources:
  - "prd.md"
downstream_consumer: "general"
created: "2026-03-20"
token_estimate: 1650
parts: 1
---

## Core Concept
- FixFlow: autonomous AI quoting agent for emergency plumbing/boiler services; London-based reference implementation; handles full quoting loop (symptom diagnosis → job classification → availability → dynamic pricing → negotiation → authority enforcement) without human involvement, 24/7
- Quote returned in <60 seconds from first message
- Target: small-to-medium trades businesses (5–20 engineers) losing out-of-hours quote requests to faster competitors
- End customer: London homeowner/renter in emergency needing immediate credible response at 11pm
- Project type: hackathon demo/reference implementation (NOT production SaaS); no auth, no multi-tenancy
- Category claim: lead qualification, scoping, pricing, negotiation are fully automatable with current tooling
- Stack: FastAPI (Python), LangGraph, Claude claude-sonnet-4-6, LangSmith tracing; UI deferred post-MVP
- Hackathon: 24-hour build, 3-person team; judged on Autonomy · Usefulness · Technical Depth · Creativity

## Architecture
- 9-node LangGraph stateful graph with conditional edges and multi-turn conversation
- QuoteState (TypedDict): shared state contract held in-memory per session via LangGraph built-in state management; schema locked in first 2 hours — all integration depends on it
- Nodes: Input Guard, Diagnostic Node, Job Classifier, Availability Node, Pricing Node, Negotiation Node, Authority Check, Output Guard, Intent Classifier
- business_config.json: sole config surface — supported brands, job types, pricing ranges, coverage zones, negotiation limits; operator modifies rules without changing agent code
- availability.json: fully stubbed; no real calendar or external API
- LangSmith: mandatory tracing integration (LANGCHAIN_TRACING_V2=true); node-by-node execution, inputs/outputs, latency; visible to judges in real time
- API-first; demo runs entirely via direct API calls + Postman; no frontend for MVP
- Data storage: in-memory only; no persistence between sessions; clears on session end or server restart

## API Surface
- POST /api/session/start — create session, return session_id and customer_name
- POST /api/chat — send message to agent, return agent response
- GET /api/quotes — return quotes from current session (in-memory only)
- Authentication: none; all endpoints open for demo

## Safety Architecture (structural, not prompt-based)
- Input Guard fires before any other node; detects prompt injection and gas-smell keywords
- Gas-smell trigger: hard stop from any node position; returns emergency redirect to Gas Emergency line 0800 111 999; no quote generated, no pricing logic runs
- Gas-smell keyword set covers: sulphur, rotten eggs, gas smell, gas leak variants
- Authority Check runs after Negotiation — cannot be negotiated away; enforced at architecture level
- Scope check in Job Classifier node logic, not LLM prompt — prevents out-of-scope hallucination
- NFR: gas-smell hard stop must fire on 100% of inputs with gas-leak indicators; 0 false negatives permitted
- NFR: 100% graceful out-of-scope escalation for any service/brand/job/location not in business_config.json; 0 hallucinated responses permitted

## Key Differentiators
- Diagnostic-first: structured diagnostic node extracts symptoms via max 5-question exchange; produces job classification with confidence level; quotes scoped to actual job not generic range — inverts burden from customer to system vs. Checkatrade/Rated People
- Config-bounded scope: agent only knows what business_config.json defines; anything outside (unsupported brands, out-of-area, undefined job types) is escalated not answered
- Full LangSmith observability: judges see node-by-node trace, inputs/outputs, latency, branching logic per conversation; no custom observability infrastructure required
- Authority limits enforced structurally: no quote >£800 auto-confirmed; replacement jobs always escalated

## Pricing & Authority Rules
- Urgency tiers: same-day, evening/weekend (×1.5 multiplier), next-day
- ULEZ surcharge applied by postcode zone
- Parts estimate included where applicable
- Medium/low confidence classification → range quote
- Auto-confirm threshold: <£300
- Human review range: £300–£800
- Hard block ceiling: >£800
- Replacement jobs: always escalated regardless of price
- Floor price enforced; competitor price-matching declined
- New customer discount: up to configured max %; returning customer discount: up to configured max % (15% shown in Scenario 3)

## Demo Scenarios (all 6 required for MVP)
- Scenario 1 (happy path): boiler repressurise, high confidence, next-day, inner London; completes in <60s; quote £95; MUST NOT CUT
- Scenario 2 (surge pricing): same job, evening/weekend; quote £142.50 (×1.5 multiplier); transparent surcharge explanation
- Scenario 3 (negotiation): minor repair £120; returning customer pushback; 15% discount offered (£102); competitor match refused; auto-confirm under £300
- Scenario 4 (gas smell hard stop): Input Guard fires before any other node; hard stop + emergency redirect; MUST NOT CUT
- Scenario 5 (scope escalation): 18-year-old boiler, replacement recommended; authority hard block >£800; human handoff messaging; escalation flag in QuoteState
- Scenario 6 (out-of-scope): Worcester Bosch brand not in config (supported: Vaillant, Baxi, Ideal); graceful decline, external redirect, no hallucinated diagnosis

## Success Criteria
- All 6 demo scenarios execute end-to-end without failure
- Scenario 1 completes <60 seconds
- Scenario 4 gas-smell hard stop fires before any pricing logic
- Scenario 6 returns graceful escalation with no hallucinated response
- LangSmith trace available and readable for every demo run
- QuoteState schema consistent across all nodes — no key errors or missing fields
- Autonomy (hackathon): full quoting loop, zero human input, 9-node conditional graph
- Technical Depth: LangGraph stateful graph, LangSmith tracing, typed state contract, cross-cutting safety
- Creativity: diagnostic-first approach; safety-first as product feature

## Functional Requirements (condensed)
- FR1–4: session start/unique ID; multi-turn state; in-session quote retrieval
- FR5: prompt injection detection and block before any processing
- FR6: gas-smell/gas-leak detection → immediate hard stop from any stage
- FR7: intent classification (quote request/general enquiry/complaint/emergency) with routing
- FR8: safety-triggered conversations redirect to emergency services with contact info
- FR9–11: max 5-question diagnostic exchange; structured symptom extraction; terminate after 4 exchanges if needed
- FR12: symptom → job type + confidence (high/medium/low)
- FR13: validate all requests against business_config.json; decline anything outside
- FR14: boiler replacement detection → auto-escalate regardless of authority level
- FR15: range quote when confidence is medium/low
- FR16: check availability by urgency tier, return up to 3 slots
- FR17: time/day detection → urgency multiplier application
- FR18: ULEZ surcharge by postcode zone
- FR19: price = f(job type, confidence, urgency tier, postcode zone)
- FR20: parts estimate in price where applicable
- FR21–25: negotiation — floor price enforcement; new/returning customer discount tiers; competitor match refusal; value-based hold
- FR26–28: authority tiers — auto-confirm <threshold; flag for review in range; hard-block above ceiling
- FR29: strip internal fields (cost price, margin, capacity data, engineer names) from customer-facing responses
- FR30: quote with unique reference number, validity window, next steps
- FR31–33: load all rules from single config file; availability from stub; operator modifies config without code change
- FR34–35: LangSmith trace per node execution; full reasoning trace inspectable per session

## Non-Functional Requirements
- NFR1: complete quote response <60s for Scenario 1 (happy path)
- NFR2: individual node execution <5s under normal conditions
- NFR3: FastAPI /api/chat response <500ms excluding LLM inference
- NFR4: LangSmith traces visible in dashboard within 10s of session completing
- NFR5: all 6 demo scenarios error-free before presentation
- NFR6: QuoteState no key errors or missing field exceptions at any node transition
- NFR7: gas-smell hard stop fires on 100% of gas-leak indicator inputs (0 false negatives)
- NFR8: 100% graceful out-of-scope escalation; 0 hallucinated responses outside configured knowledge base

## Scope Boundaries
- In: 9-node LangGraph agent; business_config.json; availability.json stub; FastAPI 3-endpoint API; LangSmith tracing; 6 demo scenarios via Postman
- Out (MVP): UI/frontend, user auth, multi-tenancy, real calendar integration, persistence/database, production SaaS surfaces
- Deferred (Phase 2): chat UI, real calendar/booking, multi-trades config (electrician, locksmith), email/WhatsApp channels, admin portal, CRM persistence
- Deferred (Phase 3): any-vertical deployment via config swap; human oversight mobile notifications for £300–£800 flagged quotes; continuous learning from quote outcomes

## Team & Build Plan
- Person A: LangGraph agent, all 9 nodes, diagnostic knowledge base, business logic
- Person B: FastAPI backend, data fixtures, business_config.json
- No frontend engineer needed for MVP
- Contingency cuts (in order): (1) Scenario 6 — loses ChatGPT differentiator but not core; (2) Negotiation Node simplified to fixed pricing; (3) Scenarios 1+4 — DO NOT CUT
- Build risk: lock QuoteState schema and node signatures in first 2 hours; Scenario 1 E2E is integration test; script all 6 scenarios as pre-written Postman message sequences

## Risks
- LLM answers out-of-scope questions despite config — mitigated: scope check in Job Classifier node logic, not LLM prompt
- Gas-smell detection misses paraphrased inputs — mitigated: keyword list covers sulphur/rotten eggs/gas smell/gas leak variants
- Authority limits bypassed via social engineering — mitigated: Authority Check runs after Negotiation, cannot be negotiated away
- LangGraph graph wiring time overrun — mitigated: lock schema first 2 hours
- Integration failures at hour 10 — mitigated: Scenario 1 E2E as integration test
- Demo instability on the day — mitigated: pre-scripted Postman collections for all 6 scenarios
