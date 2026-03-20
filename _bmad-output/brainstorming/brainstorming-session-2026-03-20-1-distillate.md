---
type: bmad-distillate
sources:
  - "brainstorming-session-2026-03-20-1.md"
downstream_consumer: "bmad-create-prd"
created: "2026-03-20"
token_estimate: 950
parts: 1
---

## Core Concept
- FixFlow: autonomous quoting agent for emergency plumbing & boiler services (London); diagnoses first, then quotes — no technical knowledge required from customer
- Stack: LangGraph (Python) + Claude claude-sonnet-4-6 + FastAPI + Next.js 14; 9-node graph with conditional edges
- Tagline: "It's 11pm. Your boiler's dead. You get a quote in 47 seconds."
- Hackathon: 24hr, 3-person team, judged on Autonomy · Usefulness · Technical Depth · Creativity

## Users
- B2B buyer: small/medium London trades business owner (5–20 engineers); loses 3–5 quote requests nightly to faster competitors; needs 24/7 quoting, pricing consistency, scope protection
- End user: London homeowner/renter in emergency (boiler out, pipes burst); wants instant quote without phone call
- Demo observer: hackathon judge; London technical/product professional; relatable problem

## Agent Architecture — 9 LangGraph Nodes
1. Input Guard: prompt injection detection; gas smell keyword → hard stop → emergency redirect, skip all nodes
2. Intent Classifier: quote / enquiry / complaint / emergency → routes accordingly
3. Diagnostic Node (differentiator): structured Q&A max 4–5 questions (error code? hot water/heating/neither? pressure gauge? boiler age? sounds?) → symptom list; max 4 exchanges before classification
4. Job Classifier: symptom list → job type + confidence (high/medium/low); low conf → range quote; replacement → escalate; gas smell → hard stop regardless of position
5. Availability Check: queries store by urgency tier; applies surge flag (evening after 6pm / weekend); returns ≤3 slots
6. Pricing Node: job type + confidence + urgency + postcode zone + time of day; ULEZ surcharge outer London; range output when medium/low confidence; parts estimate included
7. Negotiation Node: handles pushback; enforces floor; new customer 10% max discount; no competitor match; holds firm with value explanation
8. Authority Check: auto-confirm <£300; flag for review £300–£800; hard block >£800 or replacement
9. Output Guard: strips internal fields (cost price, margin, engineer names, capacity %); formats customer-facing quote; appends quote reference, 24hr validity, next steps

## Business Rules (Locked)
- Coverage: inner London (E,EC,N,NW,SE,SW,W,WC) standard; outer London/within M25 +£25 ULEZ; outside M25 declined
- Urgency multipliers: same-day (before 2pm) ×1.35; evening/weekend (after 6pm) ×1.50; next-day ×1.00
- Job types/base ranges: repressurise £80–£120; minor repair £120–£200; diverter valve £150–£250; heat exchanger/limescale £150–£300; leak/PRV £100–£200; major fault/low-conf £200–£400 flagged; replacement → ESCALATE; gas smell → HARD STOP
- Negotiation: new customer max 10%; returning max 15%; floor £80 absolute; no competitor match; no free service
- Authority: auto-confirm <£300; human review £300–£800; hard block >£800 or replacement

## Diagnostic Knowledge Base
- Pressure <1 bar, no heat/water → Repressurise (High)
- Error F28/F29 → Minor repair ignition/gas valve (High)
- Banging/kettling → Limescale/heat exchanger (High)
- No hot water only, heating works → Diverter valve (High)
- Leaking from boiler → PRV or pump seal (Medium)
- 15+ years + multiple issues → Replacement escalate (High)
- Multiple unclear symptoms → Major fault range quote, flag (Low)
- Gas smell/sulphur/rotten eggs → HARD STOP; Gas Emergency 0800 111 999 (N/A)

## Demo Scenarios (5, in order)
1. Clean path: pressure low, inner London, next-day → Repressurise £95 in <60s; all 9 nodes green
2. Surge pricing: same request 9pm → Availability detects evening → ×1.5 auto-applied; yellow
3. Negotiation: "Checkatrade quoted £80, match it?" → floor held, 10% new customer offered; yellow
4. Gas smell: "rotten egg smell" → Input Guard hard stop; red; emergency services flagged
5. Scope escalation: 18yr boiler + multiple faults → replacement → authority block → human handoff; yellow

## Shared Contracts
- State schema (QuoteState TypedDict): session_id, messages, customer_name, symptoms[], job_classification, confidence, availability_slots, calculated_price {min,max,midpoint}, urgency_tier, ulez_applicable, negotiation_result, authority_status, injection_detected, safety_escalation, fields_stripped[], quote_reference, quote_sent, escalated, events[]
- Event format: {id:uuid, timestamp, level:green|yellow|red, node:one-of-9, message, detail:{}}
- API: POST /api/session/start → {session_id,customer_name}; POST /api/chat → {session_id,message} → {response,events[]}; WS /ws/{session_id} streams to GuardPanel; GET /api/quotes

## Tech Stack
- Agent: LangGraph (Python)
- LLM: Claude claude-sonnet-4-6
- Backend: FastAPI (Python) + WebSockets native
- Frontend: Next.js 14 + Tailwind CSS
- Data: JSON files (MVP)
- Auth: Civic Auth (optional, last-hour only)
- Deployment: local for demo

## Team & Build Plan
- Person A: agent core (LangGraph graph, all 9 nodes, diagnostic KB, business logic, state schema)
- Person B: backend API + data layer (FastAPI, WebSocket, JSON configs, README)
- Person C: frontend (Next.js split-panel, chat widget, GuardPanel dashboard)
- Hours 0–2: lock contracts, schema, repo, business_config.json (all)
- Hours 2–14: parallel build
- Hour 10: first integration check (scenario 1 E2E)
- Hours 14–16: full integration all 5 scenarios
- Hours 16–20: bug fix, time scenario 1 (<60s), polish
- Hours 20–22: demo scripts, UI polish, red alert validation
- Hours 22–24: pitch deck (6 slides), demo video, GitHub README, submit

## Repo Structure
- /agent (Person A), /api (Person B), /frontend (Person C), /contracts.md (locked, team agreement to change), /business_config.json (Person B owns)

## Pitch
- Open: 11pm, boiler dead, 3 forms sent, no reply; one company has FixFlow → quote in 47s: diagnosed, checked availability, priced, handled negotiation; zero humans
- Demo: scenario 1 (all green) then scenario 4 (gas smell, red alert); GuardPanel speaks
- Close: 200K UK trades businesses lose revenue nightly for speed; FixFlow is the proof of concept

## Scope
- In: 9-node LangGraph agent, diagnostic KB, business rules, GuardPanel WebSocket dashboard, FastAPI backend, Next.js frontend, 5 demo scenarios
- Out: real calendar/booking, email/WhatsApp channel, payment processing, CRM/persistence, multi-language, admin portal, real engineer dispatch
- Deferred/conditional: Civic Auth (last hour only if time allows)
