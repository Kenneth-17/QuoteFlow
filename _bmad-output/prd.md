---
stepsCompleted: ['step-01-init', 'step-02-discovery', 'step-02b-vision', 'step-02c-executive-summary', 'step-03-success', 'step-04-journeys', 'step-05-domain-skipped', 'step-06-innovation', 'step-07-project-type', 'step-08-scoping', 'step-09-functional', 'step-10-nonfunctional', 'step-11-polish', 'step-12-complete']
workflowStatus: 'complete'
completedAt: '2026-03-20'
inputDocuments:
  - '_bmad-output/brainstorming/brainstorming-session-2026-03-20-1.md'
  - '_bmad-output/brainstorming/brainstorming-session-2026-03-20-1-distillate.md'
briefCount: 0
researchCount: 0
brainstormingCount: 2
projectDocsCount: 0
classification:
  projectType: 'saas_b2b'
  projectScope: 'demo/reference implementation — no auth, no multi-tenancy, no production SaaS surfaces'
  domain: 'general/ai-automation'
  complexity: 'high'
  projectContext: 'greenfield'
  coreValueProp: 'autonomous + auditable + bounded — the agent acts within owner-defined rules, LangSmith proves it'
  keyDifferentiators:
    - '9-node LangGraph graph with stateful multi-turn conversation and conditional edges'
    - 'LangSmith tracing for full agent observability — node-by-node trace, inputs/outputs, latency'
    - 'Input Guard with prompt injection detection and gas-smell hard stop from any node position'
    - 'Shared QuoteState contract as the coordination backbone'
    - 'Owner-bounded scope enforcement — agent refuses to answer outside defined brands, job types, and coverage zones'
workflowType: 'prd'
---

# Product Requirements Document - FixFlow

**Author:** ken
**Date:** 2026-03-20

## Executive Summary

FixFlow is an autonomous AI quoting agent for emergency plumbing and boiler services, demonstrated through a London-based reference implementation. It handles the full quoting workflow — symptom diagnosis, job classification, availability checking, dynamic pricing, negotiation, and authority enforcement — without human involvement, 24/7. A quote is returned in under 60 seconds from first message.

The product targets small-to-medium trades businesses (5–20 engineers) that lose quote requests daily to faster competitors due to out-of-hours coverage gaps. The end customer is a London homeowner or renter in an emergency who needs an immediate, credible response at 11pm — not a callback the next morning.

FixFlow is a hackathon reference implementation, not a production deployment. It proves a category claim: that the most labour-intensive, repetitive parts of service business operations — lead qualification, scoping, pricing, negotiation — are fully automatable with tools available today.

### What Makes This Special

Four properties in combination make FixFlow distinct:

1. **Diagnoses before it quotes.** A structured diagnostic node extracts symptoms through a max 5-question exchange and produces a job classification with confidence level. Quotes are scoped to the actual job — not a generic range.

2. **Autonomous and traceable.** Every node execution, routing decision, and tool call is captured in LangSmith, giving judges a full step-by-step trace of the agent's reasoning — input, output, latency, and branching logic — per conversation. No custom observability infrastructure required.

3. **Safety-first architecture.** The Input Guard detects prompt injection and gas-smell keywords before any processing begins. A gas-smell trigger produces a hard stop and emergency redirect from any node position. Authority limits are enforced at the architecture level, not via prompt instruction.

4. **Bounded by owner rules — not a general AI.** FixFlow only operates within the business owner's defined knowledge base: supported brands, service types, coverage zones, and job types. If a customer asks about anything outside that scope, the agent does not guess — it escalates. The agent's knowledge is the business config, nothing more.

The core insight: service businesses don't lack pricing knowledge or availability data — they lack a system that acts on that knowledge without requiring a human to be present. Models are capable enough, frameworks are mature enough, and the compounding cost of manual quoting is now higher than the cost of building the alternative.

**Stack:** FastAPI (Python), LangGraph, Claude claude-sonnet-4-6, LangSmith tracing; UI deferred post-MVP.
**Hackathon:** 24-hour build, 3-person team, judged on Autonomy · Usefulness · Technical Depth · Creativity.

## Success Criteria

### User Success

**B2B Buyer (Trades Business Owner):**
- FixFlow handles the entire quoting loop autonomously — no staff time required after initial setup
- Cost savings from eliminating out-of-hours admin coverage without sacrificing quote quality or customer experience
- Customers receive a diagnosed, reasoned quote — not a generic price list
- Deployable to any service business with a configuration swap

**End Customer (Homeowner in Emergency):**
- Receives a credible, specific quote within 60 seconds at any hour
- No technical knowledge required — the agent extracts symptoms through conversation
- Interaction is responsive and trustworthy, not a chatbot runaround

### Business Success (Hackathon)

| Criterion | Target | How |
|---|---|---|
| **Autonomy** | High | Full quoting loop with zero human input; 9-node graph with conditional routing |
| **Usefulness** | High | Real problem, real numbers (revenue lost to slow quoting), deployable architecture |
| **Technical Depth** | High | LangGraph stateful graph, LangSmith tracing, typed state contract, cross-cutting safety enforcement |
| **Creativity** | Medium-High | Diagnostic-first approach (vs. generic chatbot); safety-first architecture as a product feature |

### Technical Success

- All 6 demo scenarios execute end-to-end without failure
- Scenario 1 (happy path) completes in under 60 seconds
- Scenario 4 (gas smell) triggers hard stop before any pricing logic executes
- Scenario 6 (out-of-scope request) returns graceful escalation with no hallucinated response
- LangSmith trace available and readable for every demo run
- `QuoteState` schema consistent across all nodes — no key errors or missing fields mid-graph

### Measurable Outcomes

- Quote response time: **< 60 seconds** from first message to priced quote (Scenario 1)
- Diagnostic accuracy: correct job classification on all 6 demo scenarios
- Safety enforcement: gas-smell hard stop fires from **any node position**
- Scope enforcement: **0** hallucinated responses to out-of-knowledge-base requests
- Authority limits: no quote > £800 auto-confirmed; replacement jobs always escalated

## User Journeys

### Journey 1: End Customer — Happy Path (Demo Scenario 1)

**Persona:** Marcus, 34, renting a flat in Islington. It's 11:47pm on a Tuesday. His boiler pressure gauge is at 0.5 bar and there's no heat.

**Opening Scene:** Marcus has already called his letting agent's emergency line — voicemail. He's searched Checkatrade and submitted two forms. He finds FixFlow and opens the chat.

**Rising Action:** He types "my boiler isn't working." FixFlow's Diagnostic Node asks four targeted questions: error code? hot water affected? pressure reading? boiler age? Marcus answers each in plain language. Within two exchanges, FixFlow has classified the job: repressurise, high confidence, inner London, next-day slot.

**Climax:** FixFlow returns a quote — £95, next-day availability between 8am–10am, valid for 24 hours. Quote reference included. The entire exchange has taken 44 seconds.

**Resolution:** Marcus books the slot. He goes to sleep. No phone call required, no waiting until morning.

*Requirements: diagnostic Q&A flow, job classification with confidence, availability slot display, quote generation with reference number, 24hr validity window.*

---

### Journey 2: End Customer — Surge Pricing (Demo Scenario 2)

**Persona:** Same Marcus, same problem, but it's 9:15pm on a Saturday.

**Rising Action:** Identical diagnostic flow. Job classified the same way.

**Climax:** Quote comes back at £142.50 — the Availability Node detected an evening/weekend slot and the Pricing Node applied the ×1.5 urgency multiplier. FixFlow explains the surcharge transparently: *"Evening and weekend callouts carry a 50% urgency surcharge. Standard rate resumes for next-week appointments."*

**Resolution:** Marcus chooses the surge-priced slot. No hidden fees, no negotiation required.

*Requirements: urgency tier detection, automatic multiplier application, transparent pricing explanation.*

---

### Journey 3: End Customer — Negotiation (Demo Scenario 3)

**Persona:** Sarah, returning FixFlow customer. Quote comes back at £120 for a minor repair.

**Climax:** Sarah pushes back: *"Checkatrade quoted me £80, can you match it?"* The Negotiation Node holds the floor, offers a 15% returning-customer discount (£102), and explains the value: Gas Safe registered engineer, same-day availability, 12-month guarantee. Competitor price-matching is declined.

**Resolution:** Sarah accepts £102. Authority check auto-confirms (under £300). Quote sent.

*Requirements: competitor match refusal, returning customer discount tier, floor price enforcement, value explanation, authority auto-confirm under £300.*

---

### Journey 4: End Customer — Safety Hard Stop (Demo Scenario 4)

**Persona:** Tom, smells something like rotten eggs near his boiler.

**Opening Scene:** Tom types: *"There's a weird rotten egg smell coming from near my boiler."*

**Climax:** The Input Guard fires before any other node executes. FixFlow returns a hard stop: *"This sounds like it could be a gas leak. Do not use any electrical switches. Leave the property and call the Gas Emergency line: 0800 111 999."* No quote generated. No pricing logic runs.

**Resolution:** Tom calls the emergency line. FixFlow did its job — not by quoting, but by refusing to quote.

*Requirements: gas-smell keyword detection in Input Guard, hard stop before any downstream node, emergency redirect, no quote generation on safety trigger.*

---

### Journey 5: End Customer — Scope Escalation (Demo Scenario 5)

**Persona:** Derek, 18-year-old boiler, multiple symptoms — no heat, strange banging, pressure dropping.

**Rising Action:** Diagnostic Node collects symptoms. Job Classifier returns: replacement recommended, high confidence.

**Climax:** Authority Check blocks auto-confirmation. FixFlow responds: *"Based on your boiler's age and symptoms, our engineer recommends a full replacement assessment. This falls outside our instant quote system — a specialist will contact you within 2 hours to arrange a survey."*

**Resolution:** Derek is escalated to a human. The agent correctly identified the boundary of its authority.

*Requirements: replacement job detection, authority hard block > £800, human handoff messaging, escalation flag in QuoteState.*

---

### Journey 6: End Customer — Out-of-Scope Request (Demo Scenario 6)

**Persona:** Priya, asking about her Worcester Bosch boiler. The business only services Vaillant, Baxi, and Ideal.

**Opening Scene:** Priya types: *"My Worcester Bosch boiler is showing error code EA229."*

**Climax:** Job Classifier checks the brand against `business_config.json` — Worcester Bosch is not in the supported list. FixFlow responds: *"We currently service Vaillant, Baxi, and Ideal boilers only. For Worcester Bosch repairs, I'd recommend contacting a Worcester Bosch-approved engineer. If you have one of our supported brands, I'm happy to help."*

**Resolution:** Priya is directed to the right resource. No hallucinated diagnosis. No quote for a job the business can't fulfil.

*Requirements: knowledge-base scope validation in Job Classifier, graceful out-of-scope response, no hallucination beyond defined knowledge base, external redirect without quoting.*

---

### Journey 7: B2B Buyer — Seeing the Demo

**Persona:** James, owns a 12-engineer plumbing company in South London. Attends the hackathon demo.

**Opening Scene:** James watches Scenario 1 live via the API demo. LangSmith is on a second screen showing every node that fired, every input and output, the full reasoning trace.

**Climax:** Scenario 6 plays. Priya asks about a Worcester Bosch. FixFlow doesn't guess — it recognises the brand is out of scope and declines cleanly. James realises: *this isn't ChatGPT. It only knows what I've told it. It can't go rogue.*

**Resolution:** James asks: *"How do I get this for my business?"* The answer is `business_config.json`. The architecture is the same. The business rules are his to define.

*Requirements: LangSmith trace legibility, API-based demo (no UI dependency), config-driven scope enforcement, clean out-of-scope behaviour visible to observers.*

## Innovation & Novel Patterns

### Detected Innovation Areas

**1. Diagnostic-First Quoting**
Existing quoting tools (Checkatrade, Rated People) require customers to self-describe their problem and pick a job category. FixFlow inverts this — the agent extracts symptoms through structured dialogue and classifies the job itself. This shifts the intelligence burden from the customer to the system, improving both quote accuracy and conversion.

**2. Config-Bounded Agent Scope**
General-purpose LLMs will attempt to answer any question. FixFlow's agent is architecturally constrained to the owner's `business_config.json`. Anything outside — unsupported brands, out-of-area jobs, undefined job types — is escalated, not answered. The config *is* the agent's knowledge, and nothing outside it is accessible.

**3. Safety Enforcement as Architecture, Not Prompt**
Most AI safety implementations rely on system prompt instructions. FixFlow enforces safety at the graph level — the Input Guard fires before any other node, and a gas-smell detection produces a hard stop no downstream node can override. Authority limits are enforced by the Authority Check node, not by LLM judgement. Safety guarantees are structural, not probabilistic.

### Validation Approach

- **Demo Scenario 4** validates the gas-smell hard stop fires from the Input Guard position
- **Demo Scenario 6** validates config-bounded scope enforcement with no hallucination
- **LangSmith traces** provide node-level proof that safety nodes fire before pricing nodes — visible to judges in real time

### Risk Mitigation

| Risk | Mitigation |
|---|---|
| LLM answers out-of-scope questions despite config | Scope check implemented in Job Classifier node logic, not LLM prompt |
| Gas-smell detection misses paraphrased inputs | Keyword list covers: sulphur, rotten eggs, gas smell, gas leak variants |
| Authority limits bypassed via social engineering | Authority Check runs after Negotiation — it cannot be negotiated away |

## Technical Architecture

### Overview

FixFlow is a single-tenant, API-first reference implementation. No multi-tenancy, user authentication, or subscription model in MVP. The product is demonstrated entirely through direct API calls with LangSmith open on a second screen.

### API Surface (Complete for MVP)

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/session/start` | POST | Create new quote session, return `session_id` and `customer_name` |
| `/api/chat` | POST | Send message to agent, return agent response |
| `/api/quotes` | GET | Return quotes generated in the current session (in-memory only) |

**Authentication:** None. All endpoints are open for the demo.

**Data Storage:** In-memory only. No persistence between sessions. `/api/quotes` returns quotes from the active session; data clears on session end or server restart.

**Availability Data:** Fully stubbed via `availability.json` — no real calendar or external API.

### Integrations

| Integration | Purpose | Required |
|---|---|---|
| LangSmith | Agent trace observability — node-by-node execution, inputs/outputs, latency | Yes (`LANGCHAIN_TRACING_V2=true`) |

### Implementation Notes

- `business_config.json` is the sole configuration surface — business rules, supported brands, job types, pricing, coverage zones, and negotiation limits; schema defined in technical spec
- `QuoteState` (TypedDict) held in-memory per session via LangGraph's built-in state management
- All business logic runs server-side in the FastAPI + LangGraph layer — the API is the product for MVP

## Project Scoping & Phased Development

### MVP Strategy

**Approach:** Proof-of-concept demo — optimised for a 24-hour hackathon build. Success is defined by 6 demo scenarios executing reliably with LangSmith traces visible.

**Team:** 3 people. Person A: LangGraph agent, all 9 nodes, diagnostic KB, business logic. Person B: FastAPI backend, data fixtures, `business_config.json`. No frontend engineer needed for MVP — demo runs via API.

### MVP Feature Set

**Must-Have:**
- 9-node LangGraph agent (all nodes required for conditional routing)
- `business_config.json` with supported brands, job types, pricing, coverage zones
- `availability.json` stub
- FastAPI: `/api/session/start`, `/api/chat`, `/api/quotes`
- LangSmith tracing (`LANGCHAIN_TRACING_V2=true`)
- All 6 demo scenarios scripted, tested, and runnable via Postman

### Contingency Plan (If Time Runs Short)

Cut in this order — stop when back on track:

| Priority | Cut | Impact |
|---|---|---|
| 1 | Scenario 6 (out-of-scope request) | Loses the ChatGPT differentiator story — impressive but not core |
| 2 | Negotiation Node (simplify to fixed pricing) | Reduces Scenario 3 to a fixed-price response |
| 3 | Scenarios 1 + 4 | **Do not cut** — these are the demo |

### Post-MVP (Phase 2)

- Chat UI (web frontend)
- Real calendar/booking integration
- Multi-trades configuration (electrician, locksmith, etc.)
- Email/WhatsApp channel support
- Admin portal for business rules configuration
- CRM persistence

### Vision (Phase 3)

- Any service business deploys FixFlow by swapping `business_config.json` — plumbing today, legal or logistics tomorrow
- Human oversight layer for flagged quotes (£300–£800) via mobile notification
- Continuous learning from quote outcomes to refine pricing confidence

### Build Risk Mitigation

| Risk | Mitigation |
|---|---|
| LangGraph graph wiring takes longer than expected | Lock `QuoteState` schema and node signatures in first 2 hours — all integration depends on this |
| Integration failures at hour 10 check | Scenario 1 E2E is the integration test — if it passes, the graph is wired |
| Demo instability on the day | Script all 6 scenarios as pre-written message sequences; run from Postman collections |

## Functional Requirements

### Conversation & Session Management

- FR1: A customer can start a new quote session and receive a unique session identifier
- FR2: A customer can send messages to the agent within an active session and receive responses
- FR3: The system can maintain conversation state across multiple exchanges within a session
- FR4: The system can store and retrieve all quotes generated within the current session

### Intent Detection & Safety Screening

- FR5: The system can detect prompt injection attempts in incoming customer messages and block them before any processing occurs
- FR6: The system can detect gas-smell or gas-leak indicators in any customer message and trigger an immediate hard stop regardless of conversation stage
- FR7: The system can classify customer intent as quote request, general enquiry, complaint, or emergency and route accordingly
- FR8: The system can redirect safety-triggered conversations to emergency services with the appropriate contact information

### Symptom Diagnosis

- FR9: The system can conduct a structured diagnostic exchange with a customer using a maximum of 5 targeted questions
- FR10: The system can extract a structured symptom list from a customer's natural language responses
- FR11: The system can terminate the diagnostic phase after 4 exchanges and proceed with available information

### Job Classification & Scope Validation

- FR12: The system can classify a symptom set into a specific job type with an associated confidence level (high/medium/low)
- FR13: The system can validate any customer request against the owner's complete knowledge base (supported brands, job types, and coverage zones) and decline to respond for anything outside that defined scope
- FR14: The system can detect boiler replacement indicators and escalate automatically regardless of authority level
- FR15: The system can produce a range quote when job classification confidence is medium or low

### Pricing & Availability

- FR16: The system can check available appointment slots based on job urgency tier and return up to 3 options
- FR17: The system can detect the time and day of a request and apply the appropriate urgency multiplier (same-day, evening/weekend, next-day)
- FR18: The system can apply ULEZ surcharges based on the customer's postcode zone
- FR19: The system can calculate a job price using job type, confidence level, urgency tier, and postcode zone
- FR20: The system can include a parts estimate in the calculated price where applicable

### Negotiation

- FR21: The system can handle customer pushback on price without reducing below the configured floor price
- FR22: The system can offer a discount to new customers up to the configured maximum percentage
- FR23: The system can offer a discount to returning customers up to the configured maximum percentage
- FR24: The system can decline competitor price-matching requests and provide a value-based justification
- FR25: The system can hold firm on pricing with a business value explanation when no further discount is applicable

### Authority & Output Control

- FR26: The system can auto-confirm quotes below the configured authority threshold without human review
- FR27: The system can flag quotes within the review range for human follow-up
- FR28: The system can hard-block quotes above the authority ceiling from being issued
- FR29: The system can strip internal fields (cost price, margin, capacity data, engineer names) from all customer-facing responses
- FR30: The system can generate a quote with a unique reference number, a validity window, and next steps

### Business Configuration

- FR31: The system can load all business rules, supported brands, job types, pricing ranges, coverage zones, and negotiation limits from a single configuration file
- FR32: The system can load availability data from a stub fixture file
- FR33: An operator can modify business rules, pricing, and supported scope by editing the configuration file without changing agent code

### Observability

- FR34: The system can emit a trace record for every node execution, including inputs, outputs, routing decisions, and latency
- FR35: A developer or judge can inspect the full agent reasoning trace for any session via LangSmith

## Non-Functional Requirements

### Performance

- NFR1: The agent must return a complete quote response within **60 seconds** of the customer's first message for Scenario 1 (happy path, high confidence, next-day slot)
- NFR2: Individual node execution must complete within **5 seconds** under normal conditions
- NFR3: The FastAPI server must respond to `/api/chat` requests within **500ms** excluding LLM inference time
- NFR4: LangSmith traces must be visible in the LangSmith dashboard within **10 seconds** of a session completing

### Reliability

- NFR5: All 6 demo scenarios must execute without error on the demo machine before the presentation begins
- NFR6: The `QuoteState` schema must not produce key errors or missing field exceptions at any node transition
- NFR7: The gas-smell hard stop (FR6) must fire on **100%** of test inputs containing gas-leak indicators — no false negatives permitted
- NFR8: The system must return a graceful out-of-scope escalation on **100%** of requests for any service, brand, job type, or location not defined in `business_config.json` — no hallucinated responses outside the configured knowledge base are permitted
