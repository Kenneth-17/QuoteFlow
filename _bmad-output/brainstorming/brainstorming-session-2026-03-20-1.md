---
stepsCompleted: [1, 2, 3]
inputDocuments: ['SafeQuote_PRD.md']
session_topic: 'What specific AI agent project to build for a 24-hour hackathon'
session_goals: 'Narrow down to a concrete buildable idea using Claude + LangGraph that scores high on Autonomy, Usefulness, Technical depth, and Creativity'
selected_approach: 'ai-recommended'
techniques_used: ['Reversal Inversion', 'Cross-Pollination', 'What If Scenarios']
context_file: 'SafeQuote_PRD.md'
status: 'COMPLETE — all decisions locked'
---

## Session Overview

**Topic:** What specific AI agent project to build for a 24-hour hackathon
**Goals:** Identify a concrete, scoped project using Claude + LangGraph that demonstrates true autonomy, real tool use, error recovery, and multi-step reasoning — impressive enough to win

### Hackathon Context

- **Theme:** AI Agents — push autonomy as far as possible
- **Stack:** Claude + LangGraph (Civic Auth secondary / optional last-hour add)
- **Judging:** Autonomy · Usefulness · Technical Depth · Creativity
- **Deliverables:** 3-min demo video, pitch deck, public GitHub repo
- **Duration:** 24 hours
- **Team:** 3 people, all using AI to build

---

## LOCKED DECISION: The Product

### FixFlow — Autonomous Quoting Agent for Emergency Plumbing & Boiler Services (London)

**Tagline:** *"It's 11pm. Your boiler's dead. You get a quote in 47 seconds."*

**Core differentiator:** Agent doesn't just quote — it **diagnoses first, then quotes**. Structured troubleshooting Q&A maps customer-observable symptoms to job classifications before pricing. No technical knowledge required from the customer.

**Why this wins on all 4 criteria:**

| Criterion | Proof point |
|---|---|
| Autonomy | Diagnoses, prices, negotiates, escalates — zero human input across all 5 scenarios |
| Usefulness | Every Londoner has had this problem at the worst time. Instant emotional connection |
| Technical Depth | 9 LangGraph nodes, conditional edges, diagnostic knowledge base, confidence-based pricing, error recovery |
| Creativity | Diagnostic layer before quoting is novel. Gas smell hard-stop is a memorable demo moment |

---

## Customer Personas

### Customer 1 — Who Pays (B2B)
**The plumbing business owner** (e.g., "FixFlow Plumbing London" owner)
- Small/medium London trades business, 5–20 engineers
- Loses 3–5 quote requests every night to competitors who respond faster
- Has pricing inconsistency across engineers
- Scared of committing to out-of-scope jobs (liability)
- **Why they buy:** "It quotes for me 24/7, never undersells, never overpromises, I see everything it does"

### Customer 2 — End User
**London homeowner or renter in an emergency**
- Boiler out on a cold night, pipes burst, no hot water
- Goes to Google at 10pm, finds FixFlow, gets a quote in 47 seconds
- **Why they love it:** No phone call, no waiting, professional response at midnight

### Demo Observer — Hackathon Judge
- London-based technical/product professional
- Has personally experienced the "can't get a plumber" problem
- Key moment: watching the gas smell hard-stop fire red on the GuardPanel

---

## Agent Architecture — 9 LangGraph Nodes

```
Input Guard
    ↓
Intent Classifier
    ↓
Diagnostic Node        ← THE DIFFERENTIATOR
    ↓
Job Classifier
    ↓
Availability Check
    ↓
Pricing Node
    ↓
Negotiation Node
    ↓
Authority Check
    ↓
Output Guard
```

### Node Definitions

**1. Input Guard**
- Detects prompt injection, off-topic requests, abuse
- Gas smell / safety keyword detection → hard stop, skip all nodes, emergency redirect
- Passes clean input forward

**2. Intent Classifier**
- Classifies: quote request / general enquiry / complaint / emergency
- Routes: quote → diagnostic flow, emergency → immediate escalation, other → polite redirect

**3. Diagnostic Node** *(the differentiator)*
- Runs structured troubleshooting Q&A (max 4–5 questions customer can answer without technical knowledge)
- Questions: error code on display? hot water / heating / neither? pressure gauge reading? boiler age? unusual sounds?
- Maps answers → symptom list for Job Classifier
- Max 4 exchanges before classification

**4. Job Classifier**
- Takes diagnostic output → assigns job type + confidence tier (high / medium / low)
- Low confidence → quotes a range, flags uncertainty in output
- Replacement detected → skip pricing, route to escalation
- Gas smell detected → hard stop regardless of node position

**5. Availability Check**
- Queries availability store based on urgency tier (same-day / next-day / standard)
- Applies surge flag if evening (after 6pm) or weekend
- Returns up to 3 slot options

**6. Pricing Node**
- Calculates based on: job type + confidence tier + urgency tier + postcode zone + time of day
- ULEZ surcharge for outer London postcodes
- Outputs price range (not fixed price) when confidence is medium/low
- Parts estimate included in range where relevant

**7. Negotiation Node**
- Handles pushback: "Checkatrade quoted me £80", "can you do cheaper?"
- Enforces floor — never below minimum margin
- Can offer first-time customer discount (10% max)
- Holds firm beyond that with value explanation
- Competitor match: never offered

**8. Authority Check**
- Auto-confirm: jobs under £300
- Quote + flag for review: £300–£800
- Hard block + human handoff: over £800 or boiler replacement

**9. Output Guard**
- Strips internal fields: cost price, margin, engineer names, capacity percentage
- Formats professional customer-facing quote
- Appends: quote reference, 24hr validity, next steps

---

## Business Rules (Locked)

### Coverage Zones
```
Inner London (E, EC, N, NW, SE, SW, W, WC): standard rate
Outer London / within M25: +£25 ULEZ surcharge
Outside M25: declined
```

### Urgency Tiers
```
Same-day (booked before 2pm):   base × 1.35
Evening / weekend (after 6pm):  base × 1.50
Next-day standard:              base × 1.00
```

### Job Types + Base Price Ranges
```
Repressurise (low pressure):          £80–£120
Minor repair (ignition/valve fault):  £120–£200
Diverter valve:                       £150–£250
Heat exchanger / limescale:           £150–£300
Leak / pressure relief valve:         £100–£200
Major fault (unknown/low confidence): £200–£400 (flagged for engineer review)
Boiler replacement:                   ESCALATE — no quote issued
Gas smell detected:                   HARD STOP — emergency redirect only
```

### Negotiation Rules
```
New customer max discount:    10%
Returning customer max:       15%
Absolute floor:               never below £80
Competitor price match:       never offered
Free service:                 blocked
```

### Authority Limits
```
Agent auto-confirms:          under £300
Human review required:        £300–£800
Hard block (no quote):        above £800 or replacement
```

---

## Diagnostic Knowledge Base (Symptom → Job Mapping)

| Symptoms | Job Classification | Confidence |
|---|---|---|
| Pressure low (<1 bar), no heat or water | Repressurise | High |
| Error code F28 / F29 (ignition fault) | Minor repair — ignition or gas valve | High |
| Banging / kettling sounds | Limescale / heat exchanger | High |
| No hot water only, heating works | Diverter valve fault | High |
| Leaking water from boiler | Pressure relief valve or pump seal | Medium |
| Boiler 15+ years old + multiple issues | Likely replacement — escalate | High |
| Multiple symptoms, unclear | Major fault — range quote, flag for review | Low |
| Gas smell / sulphur / rotten eggs | HARD STOP — Gas Emergency 0800 111 999 | N/A |

---

## Demo Scenarios (5, in demo order)

| # | Scenario | Trigger | Key Node | GuardPanel |
|---|---|---|---|---|
| 1 | **Clean path** — Boiler not working, pressure low, inner London, next-day booking | "My boiler stopped working" | Diagnostic → Repressurise → £95 quoted in <60s | All green, all 9 nodes visible |
| 2 | **Surge pricing** — Same request at 9pm | Evening timestamp | Availability node detects evening → ×1.5 applied autonomously | Yellow: surge pricing event |
| 3 | **Negotiation** — "Checkatrade quoted me £80, match it?" | Customer pushback after quote | Negotiation node holds floor, offers 10% new customer discount | Yellow: negotiation floor enforced |
| 4 | **Gas smell** — Customer mentions sulphur smell | "There's a rotten egg smell near the boiler" | Input Guard hard stop | 🔴 Red: safety escalation, quote blocked, emergency services flagged |
| 5 | **Scope escalation** — Boiler 18 years old, multiple faults | Age + symptoms → replacement | Job Classifier → replacement → authority block → human handoff | Yellow: escalation, agent knows its limits |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Agent Framework | LangGraph (Python) |
| LLM | Claude claude-sonnet-4-6 |
| Backend API | FastAPI (Python) |
| Real-time events | WebSockets (native FastAPI) |
| Frontend | Next.js 14 + Tailwind CSS |
| Business data | JSON files (MVP) |
| Auth (optional) | Civic Auth — add last if time allows |
| Deployment | Local for demo |

---

## Team Structure & Workstreams

| Person | Workstream | Owns |
|---|---|---|
| A | Agent Core | LangGraph graph, all 9 nodes, diagnostic KB, business logic, state schema |
| B | Backend API + Data Layer | FastAPI, WebSocket, JSON config files, repo setup |
| C | Frontend | Next.js split-panel UI, chat widget, GuardPanel dashboard |

---

## Shared Contracts (defined hour 0–2, locked before parallel build)

### State Schema
```python
class QuoteState(TypedDict):
    session_id: str
    messages: List[dict]
    customer_name: str
    # Diagnostic
    symptoms: List[str]
    job_classification: Optional[str]
    confidence: Optional[str]        # "high" | "medium" | "low"
    # Checks
    availability_slots: Optional[List[dict]]
    calculated_price: Optional[dict] # {min, max, midpoint}
    urgency_tier: Optional[str]
    ulez_applicable: Optional[bool]
    negotiation_result: Optional[dict]
    authority_status: Optional[str]  # "auto_confirm" | "review" | "blocked"
    # Guards
    injection_detected: bool
    safety_escalation: bool
    fields_stripped: List[str]
    # Output
    quote_reference: Optional[str]
    quote_sent: bool
    escalated: bool
    # GuardPanel broadcast
    events: List[dict]
```

### Event Format
```json
{
  "id": "uuid",
  "timestamp": "14:32:01",
  "level": "green | yellow | red",
  "node": "input_guard | intent | diagnostic | job_classifier | availability | pricing | negotiation | authority | output_guard",
  "message": "Human readable description",
  "detail": {}
}
```

### API Contract
```
POST /api/session/start     → { session_id, customer_name }
POST /api/chat              → { session_id, message } → { response, events[] }
WS   /ws/{session_id}       → streams events to GuardPanel in real time
GET  /api/quotes            → quote history list
```

---

## Repo Structure
```
/agent          → Person A (LangGraph nodes, graph, KB)
/api            → Person B (FastAPI, WebSocket, data files)
/frontend       → Person C (Next.js, chat, GuardPanel)
/contracts.md   → shared, no changes without team agreement
/business_config.json → Person B owns
/README.md      → Person B writes
```

---

## 24-Hour Build Plan

| Hours | Activity | Who |
|---|---|---|
| 0–2 | Lock contracts, shared schema, repo setup, business_config.json | All together |
| 2–14 | Parallel build (agent / API / frontend) | Split |
| 10 | First integration check — demo scenario 1 end-to-end | All together |
| 14–16 | Full integration, all 5 scenarios | All together |
| 16–20 | Fix bugs, time scenario 1 (<60s target), polish | All together |
| 20–22 | Lock demo scripts, UI polish, red alert validation | All together |
| 22–24 | Pitch deck (6 slides), demo video, GitHub README, submit | All together |

---

## Pitch Narrative

**Opening (30s):**
*"It's 11pm. Your boiler's dead. You've sent three enquiry forms and heard nothing. One company has FixFlow. You get a quote in 47 seconds — after the agent diagnosed your fault, checked availability, calculated the price, and handled your negotiation attempt. No human touched it."*

**Demo (90s):**
Run scenario 1 (clean path, all green), then scenario 4 (gas smell, red alert). Let the GuardPanel speak for itself.

**Close (30s):**
*"There are 200,000 trades businesses in the UK who lose revenue every night because they can't respond fast enough. FixFlow is the proof of concept. This is what autonomous service agents look like."*

---

## What is Explicitly Out of Scope (MVP)

- Civic Auth (add last hour only if time allows)
- Real calendar / booking system
- Email or WhatsApp channel (web chat only)
- Payment processing
- CRM or customer database persistence
- Multi-language support
- Admin portal for business owner to edit config
- Real engineer dispatch
