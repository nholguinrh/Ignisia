# 💡 Ignisia — Idea Prioritization & Maturation Platform
 
>  The place where ideas catch fire and become real. Ignite-IA, A living backlog system where ideas compete, evolve, and earn their way into development through dynamic, team-driven scoring.
 
---
 
## 🧭 Table of Contents
 
1. [Vision & Problem Statement](#vision--problem-statement)
2. [Core Concepts](#core-concepts)
3. [How It Works](#how-it-works)
4. [Scoring System](#scoring-system)
5. [Feature Management](#feature-management)
6. [Rating Sessions (Review Meetings)](#rating-sessions-review-meetings)
7. [Idea Maturity Lifecycle](#idea-maturity-lifecycle)
8. [Use Cases & Examples](#use-cases--examples)
9. [Frameworks & Inspiration](#frameworks--inspiration)
10. [Data Model](#data-model)
11. [Tech Stack (Proposed)](#tech-stack-proposed)
12. [Roadmap](#roadmap)
13. [Contributing](#contributing)
 
---
 
## Vision & Problem Statement
 
Most teams have more ideas than capacity. The challenge is not coming up with ideas — it's:
 
- **Knowing which ideas are worth pursuing** at a given moment.
- **Letting ideas mature** — some ideas start vague and become brilliant only after discussion.
- **Evolving the evaluation criteria** — a metric that didn't exist last month might be the most important factor today.
- **Avoiding gut-feel decisions** — replacing "whoever speaks loudest in the meeting" with a transparent, data-driven priority score.
 
**Ignisia** solves this by providing a backlog system where:
- Ideas are submitted and can evolve over time.
- Teams define **scoring features** (dimensions to evaluate ideas on).
- Periodic **Rating Sessions** allow the team to collaboratively score ideas on each feature.
- The **Weighted Priority Score** of each idea determines which one gets team resources next sprint.
 
---
 
## Core Concepts
 
### 1. 💡 Idea
An **Idea** is a proposal for something the team could build, research, or explore. It starts as a raw concept and matures through documentation, discussion, and scoring.
 
An Idea contains:
- Title & description
- Author and creation date
- Current maturity stage
- Assigned features and scores
- History of rating sessions
- Comments and discussion thread
- Links to related ideas or dependencies
 
### 2. 🏷️ Feature (Scoring Dimension)
A **Feature** is a configurable evaluation axis that the team uses to rate ideas. Features are created by team members and can be added to any idea.
 
Examples of Features:
| Feature Name | Description | Scale |
|---|---|---|
| Complexity | How technically hard is this? | 1 (simple) → 5 (very complex) |
| Profitability | Expected revenue or cost savings | 1 (low) → 5 (high) |
| Time to Market | How fast can this reach users? | 1 (months) → 5 (days) |
| Cost | Estimated implementation cost | 1 (cheap) → 5 (expensive) |
| Disruption | Does this change the market? | 1 (incremental) → 5 (game-changing) |
| Versatility | Can this apply across products? | 1 (narrow) → 5 (broad) |
| Strategic Fit | Aligns with company vision? | 1 (unrelated) → 5 (core) |
| Risk | Implementation or market risk | 1 (low) → 5 (high) |
 
> 💡 Features are **not fixed**. The team can create new features at any time. New features can be applied retroactively to existing ideas and scored in the next Rating Session.
 
### 3. 🗳️ Rating Session
A **Rating Session** is a structured team meeting (or async process) where participants score one or more ideas across one or more features.
 
Each session produces:
- A set of scores per idea/feature combination
- A new **Weighted Priority Score** for each idea
- A session summary with discussion notes
 
### 4. 📊 Priority Score
The **Priority Score** is a calculated number that determines where an idea sits in the backlog. It is **recalculated after every Rating Session**.
 
The score is computed from the individual feature ratings combined with their weights (defined per feature). The team can configure which features increase or decrease the score (e.g., high Complexity might reduce score, while high Profitability increases it).
 
### 5. 🌱 Maturity Stage
Ideas progress through defined stages that reflect how well-understood and ready they are:
 
```
Raw → Exploring → Defined → Validated → Ready → In Development → Done / Discarded
```
 
An idea's stage is not automatic — it requires deliberate team action (e.g., a "promote" vote in a Rating Session).
 
---
 
## How It Works
 
```
┌─────────────────────────────────────────────────────────────┐
│                        IDEA LIFECYCLE                        │
│                                                             │
│  Someone has          Team discusses        Score is        │
│  an idea     ──────►  in Rating    ──────►  calculated  ──► │
│  (submits)            Session               & ranked        │
│                                                             │
│  ◄─────────────────────────────────────────────────────────►│
│  Ideas with top scores are assigned to the next sprint      │
│                                                             │
│  Ideas with low scores wait, get refined, or are discarded  │
└─────────────────────────────────────────────────────────────┘
```
 
### Step by Step
 
1. **Submit an Idea** — Any team member can submit an idea with a title, description, and optional initial notes. No scoring happens yet.
 
2. **Features are Assigned** — The idea author (or moderator) assigns relevant scoring features to the idea. Not all features apply to all ideas.
 
3. **Rating Session is Scheduled** — Periodically (weekly, bi-weekly), a Rating Session is created and ideas to review are selected.
 
4. **Team Scores Each Idea** — During the session, each participant rates each idea on the assigned features. Scores are averaged across participants.
 
5. **Priority Score is Computed** — The system calculates the Weighted Priority Score for each idea.
 
6. **Backlog is Reordered** — Ideas are ranked by Priority Score. The top ideas get assigned to the next sprint.
 
7. **Cycle Repeats** — Each sprint cycle, ideas can be re-rated, new features can be introduced, and the backlog re-orders.
 
---
 
## Scoring System
 
### Feature Polarity
Each feature has a **polarity** — it either adds to or subtracts from the priority score:
 
| Polarity | Meaning | Example |
|---|---|---|
| `POSITIVE` | Higher rating = higher priority | Profitability, Disruption |
| `NEGATIVE` | Higher rating = lower priority | Complexity, Cost, Risk |
 
### Feature Weight
Each feature also has a **weight** (e.g., 1.0 to 3.0) that controls how much it influences the final score. The team can adjust weights globally.
 
### Score Formula
 
```
Priority Score = Σ (feature_score × feature_weight × polarity_multiplier)
 
Where:
  - feature_score = average of all participant ratings for this feature (1–5)
  - feature_weight = configured weight for this feature
  - polarity_multiplier = +1 for POSITIVE features, -1 for NEGATIVE features
```
 
### Example Calculation
 
Idea: *AI Phone Attendant for Realty*
 
| Feature | Score | Weight | Polarity | Contribution |
|---|---|---|---|---|
| Profitability | 5 | 2.0 | + | +10.0 |
| Disruption | 4 | 1.5 | + | +6.0 |
| Time to Market | 3 | 1.0 | + | +3.0 |
| Complexity | 4 | 1.5 | - | -6.0 |
| Cost | 3 | 1.0 | - | -3.0 |
| **Total Priority Score** | | | | **10.0** |
 
Next Rating Session, the team adds the new feature **Versatility** (weight 1.0, POSITIVE) and rates this idea a 5:
 
| New Feature | Score | Weight | Polarity | Contribution |
|---|---|---|---|---|
| Versatility | 5 | 1.0 | + | +5.0 |
| **New Total** | | | | **15.0** |
 
This could move the idea from position #4 to position #1 in the backlog.
 
---
 
## Feature Management
 
Features are **shared resources** across the entire backlog. They are not locked to a single idea.
 
### Creating a Feature
 
Any team member can propose a new feature. Features have:
- `name` — e.g., "Versatility"
- `description` — what it measures and how to rate it
- `polarity` — POSITIVE or NEGATIVE
- `default_weight` — starting weight (teams can override per session)
- `scale_description` — what 1, 3, and 5 mean for this feature
- `created_by` / `created_at`
 
### Assigning Features to Ideas
 
Once a feature exists, it can be assigned to any idea. When assigned, the idea's next Rating Session will include that feature for scoring.
 
### Feature Evolution
 
- Features can be **retired** (hidden from new sessions but preserved in history).
- Teams can **rename or redefine** features as their understanding evolves.
- **Feature scores are versioned** — each Rating Session captures the scores independently, so the history is never overwritten.
 
---
 
## Rating Sessions (Review Meetings)
 
A Rating Session is the **heartbeat of Ignisia**. It is a structured event where the team aligns on idea quality.
 
### Session Types
 
| Type | Description |
|---|---|
| **Full Review** | All active ideas in the backlog are scored |
| **Spot Review** | Only newly submitted ideas are scored |
| **Feature Audit** | Team reviews and adjusts feature weights globally |
| **Maturity Review** | Team votes to advance ideas to next maturity stage |
 
### Session Flow
 
1. **Create Session** — Moderator creates a session, selects which ideas and features to include.
2. **Invite Participants** — Team members are notified.
3. **Individual Scoring** — Each participant scores independently (anonymous until all submit).
4. **Reveal & Discuss** — Scores are revealed. Outliers trigger discussion.
5. **Finalize Scores** — Scores are saved. Priority Score recalculated.
6. **Backlog Updated** — The ranked backlog is published for the team.
 
### Score Reveal Mechanics (Anti-Anchoring)
 
To prevent anchoring bias (copying the loudest person's opinion), Ignisia uses a **hidden voting model**:
- Participants submit scores without seeing others' ratings.
- All scores are revealed simultaneously after everyone submits.
- Large variance (e.g., one person gives 5, another gives 1) triggers a mandatory discussion flag.
 
---
 
## Idea Maturity Lifecycle
 
Ideas should not stay in the backlog forever as vague concepts. The maturity system forces clarity.
 
```
┌─────────┐    ┌───────────┐    ┌─────────┐    ┌───────────┐    ┌───────┐
│   RAW   │───►│ EXPLORING │───►│ DEFINED │───►│ VALIDATED │───►│ READY │
└─────────┘    └───────────┘    └─────────┘    └───────────┘    └───────┘
                                                                    │
                                                                    ▼
                                                            ┌──────────────┐
                                                            │ IN DEVELOPMENT│
                                                            └──────────────┘
                                                                    │
                                                          ┌─────────┴────────┐
                                                          ▼                  ▼
                                                       ┌──────┐        ┌──────────┐
                                                       │ DONE │        │ DISCARDED│
                                                       └──────┘        └──────────┘
```
 
### Stage Definitions
 
| Stage | Description | Exit Criteria |
|---|---|---|
| **Raw** | Just submitted. Title and a few lines. | Author adds full description |
| **Exploring** | Team has discussed in at least one session | Has at least 3 features assigned and scored once |
| **Defined** | Clear problem, solution, and scope | Has acceptance criteria and estimated complexity |
| **Validated** | Team agrees it's viable and worth doing | Priority Score above team threshold AND maturity vote passed |
| **Ready** | Fully specified and approved for development | All blockers resolved |
| **In Development** | Work has started | Sprint assigned |
| **Done** | Shipped | Retrospective completed |
| **Discarded** | Decided not to pursue | Reason documented |
 
---
 
## Use Cases & Examples
 
### Use Case 1: The AI Phone Attendant
 
**Scenario**: A team at a real estate company wants to build an AI system that receives phone calls and handles customer inquiries in real time using an LLM.
 
**Week 1 — Submission**:
- Team member submits the idea: *"AI Phone Attendant for Realty Clients"*
- Features assigned: Complexity (NEGATIVE), Profitability (POSITIVE), Cost (NEGATIVE), Time to Market (POSITIVE), Disruption (POSITIVE)
- After Rating Session: Priority Score = **10.0** → Position #4 in backlog
 
**Week 2 — New Feature Added**:
- A team member notices "Versatility" is not being tracked. They create it as a new POSITIVE feature.
- It's assigned to several ideas and scored in the next Rating Session.
- The AI Phone Attendant scores 5 on Versatility (it can work for any industry, not just realty).
- New Priority Score = **15.0** → Jumps to Position #1
 
**Week 3 — Team works on the AI Phone Attendant**.
 
---
 
### Use Case 2: Idea That Needs to Mature
 
**Scenario**: Someone submits: *"Use blockchain for document verification"*.
 
- Week 1: Raw stage, minimal description. No features assigned yet.
- Week 2: Team discusses and assigns Complexity (5 — very high), Profitability (2 — unclear), Risk (5 — high). Score is very low.
- Weeks 3–6: Idea stays in backlog. Author refines the scope to "blockchain-lite hash verification for contract signing."
- Week 7: Re-rated. Complexity drops to 3, Profitability rises to 4. Score improves. Advances to **Defined** stage.
- Week 10: Validated and Ready. Gets scheduled.
 
This demonstrates the **maturation dynamic** — good ideas that start unclear can earn their way up.
 
---
 
### Use Case 3: Feature Weight Audit
 
**Scenario**: After 2 months, the team realizes Cost has been over-weighted and is blocking too many good ideas.
 
- A **Feature Audit session** is called.
- Team votes to reduce Cost's weight from 2.0 to 1.0.
- All Priority Scores are recalculated with the new weight.
- This is logged as a historical event so the team can trace why the backlog order changed.
 
---
 
## Frameworks & Inspiration
 
Ignisia draws from multiple well-known product and innovation frameworks:
 
| Framework | Contribution to Ignisia |
|---|---|
| **RICE Scoring** (Reach, Impact, Confidence, Effort) | Foundation for multi-dimensional weighted scoring |
| **ICE Scoring** (Impact, Confidence, Ease) | Lightweight scoring model for quick sessions |
| **Weighted Shortest Job First (WSJF)** | SAFe Agile concept for prioritizing by business value ÷ cost |
| **Stage-Gate Process** | The Maturity Lifecycle with defined exit criteria per stage |
| **Planning Poker** | Anti-anchoring, async voting for rating sessions |
| **Opportunity Scoring** | Identifying underserved needs via importance vs. satisfaction |
| **Impact/Effort Matrix** | 2x2 quadrant basis, extended to N dimensions |
| **OKR Alignment** | Ideas can be linked to company Objectives & Key Results |
 
Ignisia's unique contribution is **dynamic feature management** — unlike RICE or ICE which have fixed dimensions, Ignisia's features are user-defined, weighted, polarized, and evolve over time.
 
---
 
## Contributing
 
We welcome contributions! Please read our [CONTRIBUTING.md](CONTRIBUTING.md) for:
- How to submit ideas (yes, we use Ignisia to manage Ignisia 🙂)
- Code style guidelines
- PR process
- How to propose new Features (scoring dimensions) for the system
 
---
 
## License
 
MIT License — see [LICENSE](LICENSE) for details.
 
---
 
*Built for teams who believe the best ideas should win — not just the loudest ones.*
