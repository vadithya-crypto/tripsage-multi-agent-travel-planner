# ✈️ TripSage — Multi-Agent Autonomous Travel Planning Assistant

> Give it an origin, destination, budget and dates. A team of specialized AI agents plans the whole trip: budget split, stay options, travel options, a day-by-day itinerary, and a final budget-checked plan with booking links.

---

## 📌 Overview

Planning a trip means juggling flights, trains, hotels, local transport and sightseeing across many disconnected apps, all while staying inside a budget.

**TripSage** is a multi-agent system built for the Agentic AI course. Instead of one LLM answering everything in a single prompt, the problem is split across specialized agents. Each has its own role, its own system prompt and its own output contract. They run in parallel, hand structured data to each other, and a final agent synthesizes everything into one plan.

The core approach is: **Decompose → Reason in parallel → Merge → Synthesize → Act**

---

## 🚨 Problem Statement

Travellers have to manually research and compare options across many websites, and nothing reasons across the *whole* trip at once. In particular, nothing tells them early and clearly whether the plan fits their budget, and what to change if it doesn't.

TripSage takes a few simple inputs (origin, destination, budget, dates, travelers, interests) and produces a complete, comparable, budget-aware trip plan.

---

## 💡 Proposed Solution

```
User form (HTML frontend)
        │  POST
        ▼
   n8n Webhook
        ├──► Budget Allocator Agent ───────────────────────────┐
        ├──► Stay Data Generator ──► Stay Agent ───────────────┤
        ├──► Travel Data Generator ──► Travel Options Agent ───┼──► Merge (4 inputs)
        └──► Itinerary Agent ──────────────────────────────────┘        │
                                                                         ▼
                                                          Combine Agent Outputs (Code node)
                                                                         │
                                                                         ▼
                                                             Booking Summary Agent
                                                                         │
                                                                         ▼
                                                              Respond to Webhook
                                                                         │
                                                                         ▼
                                                      Frontend renders comparison UI
```

---

## 🤖 Agents

| Agent | Responsibility |
|---|---|
| **Budget Allocator** | Splits the total budget across travel, stay, food, activities and a 10% buffer, based on the destination's cost tier |
| **Stay Data Generator + Stay Agent** | Generates realistic stay options for the destination, then ranks them against the stay budget and trip dates |
| **Travel Data Generator + Travel Options Agent** | Generates realistic flight/train/bus options for the specific route, then ranks them on cost, duration and comfort for the whole group |
| **Itinerary Agent** | Builds a day-by-day plan matched to the traveler's interests, sequenced geographically, with local transport time and cost between stops |
| **Booking Summary Agent** | Synthesizes all four outputs into one structured plan: budget breakdown, whether the plan fits the budget, 2 alternative scenarios, 3 stay options, 3 travel options, and a full itinerary |

**Emergent behaviour:** no single agent calculates whether the whole plan is over budget. That judgement only appears when the Booking Summary Agent sees all the other agents' outputs together, and it then proposes "Budget Saver" and "Comfort Upgrade" alternatives.

---

## 🧰 Tools & Technologies

| Layer | Tool |
|---|---|
| Workflow orchestration | **n8n** (self-hosted, community edition) |
| LLM inference | **Groq API** (`openai/gpt-oss-120b`) via n8n's Groq Chat Model + Basic LLM Chain nodes |
| Parallel execution & sync | n8n branching + **Merge** node |
| Data shaping | n8n **Code** node (JavaScript) |
| API surface | n8n **Webhook** + **Respond to Webhook** |
| Frontend | HTML, CSS, vanilla JavaScript (single file) |
| Editor | VS Code |

---

## 🗂️ Repository Structure

```
tripsage/
│
├── workflow/
│   └── tripsage-orchestrator.json     # exported n8n workflow (import this)
│
├── frontend/
│   └── index.html                     # demo UI (form + comparison results)
│
├── screenshots/
│   ├── n8n-workflow.png
│   ├── ui-budget-and-options.png
│   └── ui-itinerary.png
│
├── .gitignore
└── README.md
```

---

## ▶️ How to Run

1. **Install and start n8n**
   ```
   npm install -g n8n
   n8n start
   ```
   Then open `http://localhost:5678`.
2. **Import the workflow**: Workflows → Import from file → `workflow/tripsage-orchestrator.json`.
3. **Add your Groq credential**: get a free API key at [console.groq.com](https://console.groq.com), create a Groq credential in n8n, and attach it to each Groq Chat Model node (model: `openai/gpt-oss-120b`).
4. **Publish** the workflow so the production webhook is live.
5. **Open `frontend/index.html`** in a browser. The webhook URL is set at the top of the script (`http://localhost:5678/webhook/tripsage-input`).
6. Fill in the form and click **Plan My Trip**. A full run takes roughly 15–40 seconds.

> ⚠️ Groq's free tier has per-model request limits. If you hit a rate-limit error, wait a bit or switch the Groq Chat Model nodes to another model.

---

## 🖼️ Output

The frontend renders:
- **Trip summary** and a **budget-fit badge** (within budget / over by ₹X)
- **Budget breakdown** with a bar per category and the reasoning behind the split
- **Two alternative scenarios** (Budget Saver, Comfort Upgrade) with estimated totals
- **Three stay options** and **three travel options** side by side, with price, rating/timings, duration and a "Book on …" button each
- A **day-by-day itinerary** with real calendar dates, time slots, place descriptions and transport notes

_(Add screenshots to the `screenshots/` folder and link them here.)_

---

## ⚖️ Design Decisions & Honest Scope

- **Option data is LLM-generated, not live inventory.** Hotel and travel options are realistic estimates generated per destination by a data-generator agent. This lets the system work for *any* origin/destination without a hardcoded dataset. It is not real-time pricing or availability.
- **Booking buttons open real searches, not pre-filled checkouts.** The frontend builds URLs from the agents' picks (Google Flights for flights, Google search for hotels/trains/buses). There is no booking API and no payment handling.
- **No MCP.** Agents call the LLM through n8n's native nodes. MCP would be a natural way to plug in live hotel/flight tools later.
- **No agent-to-agent feedback loop yet.** The Booking Summary Agent can detect an over-budget plan and propose alternatives, but it cannot send work back to earlier agents.
- **One model across all agents** keeps output format and behaviour consistent and easier to debug.

---

## 🔭 Future Enhancements

- **Feasibility pre-check agent**: reject impossible requests early (e.g. Pune → Dubai on ₹1,000) before running the full pipeline
- Replace the data-generator agents with live APIs (Google Places, flight/hotel aggregators), ideally via MCP tools
- Feedback loop: when the plan is over budget, automatically re-run the stay/travel agents with tighter constraints
- Deep links to specific listings and, where supported, checkout pages

---

## 👤 Author

**V Adithya** — [25030242063] — SCIT, Pune
Agentic AI course project, 2026
