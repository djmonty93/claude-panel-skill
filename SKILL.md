---
name: panel
description: Facilitate a structured expert panel discussion to make a decision or design something. Spawns 6-7 distinct expert personas (including a mandatory contrarian) who independently propose ideas, then debate, then vote keep/drop/modify on each item. Use this skill whenever the user invokes "/panel", asks for an "expert panel," "panel of experts," "roundtable discussion," "advisory board," "panel discussion," or asks Claude to "spawn agents/experts/personas to discuss" a topic. Also use this skill when the user wants to stress-test a decision, get multiple perspectives on a design, brainstorm features with structured critique, or wants opinionated debate rather than a flat list of options. The skill uses tappable multiple-choice prompts via ask_user_input_v0 to gather panel parameters without requiring the user to type answers.
---

# Expert Panel Discussion

This skill facilitates a structured expert panel discussion to help the user make a decision or design something. The panel produces opinionated keep/drop recommendations through independent proposals, multi-round debate, and a final vote.

## Why this skill exists

Most "give me suggestions" prompts produce flat, polite lists that converge on the obvious answer. A panel with distinct personalities, real disagreement, and a mandatory contrarian produces *opinionated* output that catches blind spots, surfaces trade-offs, and forces real decisions. The structure (independent proposals first, debate second, vote last) is what prevents groupthink and premature convergence.

## When to use this skill

Trigger this skill when the user:
- Invokes `/panel` or asks for a "panel"
- Asks for an "expert panel," "panel of experts," "roundtable," "advisory board"
- Says "spawn agents/experts/personas to discuss X"
- Wants to stress-test a decision or design with multiple viewpoints
- Wants opinionated debate, not a flat list

Do NOT use this skill for:
- Simple factual questions
- Single-perspective requests ("what do you think about X")
- When the user explicitly wants one expert, not many

## Workflow

### Step 1: Ask for the topic in plain text

The topic is unique to every invocation and cannot be a button. Send a plain message asking:

> What is the topic, decision, or project the panel should discuss?

Wait for the user's typed response before doing anything else.

### Step 2: Gather panel parameters via tappable prompts

After the user gives the topic, use `ask_user_input_v0` to collect parameters. Split into two tool calls to avoid overwhelming the user.

**First parameter call** — three questions in one tool call:

Question 1: "What kind of output do you want?" (single_select)
- Feature list with keep/drop recommendations
- Strategic recommendation
- Risk assessment with mitigations
- Design critique
- Go / no-go decision
- Ranked options with trade-offs

Question 2: "How big should the panel be?" (single_select)
- Small (3-4 panelists)
- Standard (6-7 panelists, includes contrarian)
- Large (8-10 panelists for complex topics)

Question 3: "Panel composition?" (single_select)
- You propose the full roster
- I'll specify some roles, you fill the rest
- I'll specify all roles

**Second parameter call** — one multi-select question:

"Any constraints I should know about? (pick any that apply)" (multi_select)
- Budget-limited (solo / bootstrap)
- Tight timeline (days or weeks)
- Specific tech stack required
- Regulatory or compliance pressure
- Specific target audience matters
- Must integrate with existing systems
- None of these

If the user picks any constraints, ask a brief text follow-up to capture specifics. If they pick "None," proceed.

If the user picked "I'll specify some/all roles" in question 3, ask them in text to name the roles.

### Step 3: Propose the panel

Build a roster matching the requested size. Every roster MUST:

- Give each panelist a distinct name and a one-sentence lens description
- Include AT LEAST ONE explicit contrarian whose default position is skepticism or "don't do this"
- Ensure at least two panelists would meaningfully disagree with each other
- Cover the major decision dimensions for the topic (user, builder, monetizer, domain expert, implementation expert, skeptic — adapt to topic)

Present the proposed panel, then use `ask_user_input_v0`:

"Panel approval?" (single_select)
- Approved, run the discussion
- Swap or add panelists (I'll specify)
- Reduce or simplify the panel

If swap/add or reduce, ask a text follow-up. Otherwise proceed.

### Step 4: Run the discussion in rounds

#### Round 1 — Independent proposals
Each panelist independently proposes 5-6 ideas, features, concerns, or recommendations. Speak in their own voice, with their own priorities. Do NOT reconcile or merge yet. Length: roughly 5-8 sentences per panelist's full block.

#### Round 2 — Consolidated candidate list
De-duplicate similar items from Round 1 into a single numbered list. Note source panelist(s) for each. Flag any items that are obviously "table stakes" (required baseline) and set them aside for separate handling.

#### Round 3 — Debate
Walk through the candidate list in logical groupings (by category, by panelist, or by theme). For each contested item:
- The proposing panelist defends
- Other panelists agree, modify, or attack
- The contrarian gets explicit airtime on marginal items
- Capture actual back-and-forth dialogue, not just conclusions — the disagreements are the value

Use multiple sub-rounds (3A, 3B, 3C...) for substantive items. After Round 3, use `ask_user_input_v0`:

"Continue to vote, or extend debate?" (single_select)
- Continue to Round 4 (contrarian last word)
- Extend Round 3 on specific items (I'll specify)
- Skip ahead to voting

#### Round 4 — Contrarian last word
The contrarian gets final shots at marginal items. They can argue for dropping, simplifying, or deferring anything that survived Round 3 on shaky ground.

#### Round 5 — Vote
Each panelist votes keep / drop / modify on each candidate item. Present results in a table:

| # | Item | P1 | P2 | P3 | P4 | P5 | P6 | P7 | Result | Ruling |

Tally the votes. The facilitator (you) breaks ties and may override on scope, safety, or coherence grounds — but state the reasoning when overriding. Be opinionated; do not hedge ties as "both have merit."

### Step 5: Final report

Produce two artifacts:

**Narrative summary** (2-4 paragraphs):
- Shape of the final recommendation
- The biggest unanimous decision (clearest signal)
- The most contested decision and how it resolved
- Any pivots where the discussion fundamentally reframed an item
- Anything the panel did NOT surface that you think is worth flagging

**Decision table** with these default columns:

| # | Item Name | Description | Keep/Drop/Modify | Why |

Customize columns to fit the topic. For risk assessment: Likelihood, Impact, Mitigation. For ranked options: Rank, Score, Trade-offs.

After delivering the report, use `ask_user_input_v0`:

"Next step?" (single_select)
- Looks good, we're done
- Change some keep/drop decisions
- Build an implementation prompt from the keepers
- Re-open the discussion on specific items

## Rules of engagement (non-negotiable)

These rules separate a useful panel from a sycophantic agreement machine. Violating any of them defeats the purpose of the skill.

1. **Panelists must have real personality and disagreement.** Do not have them all politely agree. If your draft has everyone nodding, rewrite it.

2. **The contrarian must actually push back.** Soft devil's-advocate framing ("on the other hand, one could argue...") is not pushback. The contrarian's default position is "this is wrong, here's why."

3. **Do not sanitize disagreements into consensus too early.** Let tension live in the transcript through Round 3. Resolution happens in Round 5, not before.

4. **Capture actual back-and-forth dialogue in Round 3**, not summarized conclusions. The reader should see "Pete: X. Carl: wrong, because Y. Dana: actually both are missing Z." Not "the panel discussed X and concluded Z."

5. **Be opinionated in tie-breaks.** State your reasoning. Do not hedge with "both sides have merit" — pick one and defend it.

6. **The facilitator is not a panelist.** You orchestrate, summarize, and break ties. You do not vote in Round 5 (except as tiebreaker).

7. **If the user pushes back on a conclusion, re-open the relevant round** rather than just changing the answer to please them.

8. **Use `ask_user_input_v0` at every decision point** with 2-4 discrete options. Use plain text only for open-ended inputs (the topic itself, role specifications, constraint details).

## Panel composition guidance by topic type

These are starting templates. Always adapt to the specific topic.

**Product/feature design:**
- Target user (cares about UX, simplicity)
- Domain expert (cares about technical correctness)
- Builder/implementer (cares about feasibility, cost)
- Monetizer (cares about revenue, conversion)
- SEO/distribution (cares about how anyone finds it)
- Skeptic/contrarian (cares about whether it should exist at all)

**Strategic decision:**
- The optimist (sees upside)
- The pessimist (sees downside)
- The numbers person (cares about ROI, metrics)
- The operator (cares about execution feasibility)
- The customer (cares about whether anyone wants this)
- The contrarian (questions the framing of the decision)

**Risk assessment:**
- The auditor (compliance, regulatory)
- The engineer (technical failure modes)
- The business owner (financial impact)
- The legal advisor (liability exposure)
- The historian (what went wrong last time)
- The contrarian (challenges whether the risk is real)

**Creative critique:**
- The craftsperson (cares about quality of execution)
- The audience (cares about whether it lands)
- The marketer (cares about hook, distinctiveness)
- The editor (cares about clarity, cuts)
- The historian (cares about prior art)
- The contrarian (defends "why bother")

## Output format examples

**Round 1 panelist proposal (correct style):**

> **Pitmaster Pete** (BBQ veteran)
> 1. Cut-specific scoring — brisket cares about wind, ribs hate rain.
> 2. Stall-risk indicator — humidity + wind predict stall severity.
> 3. Overnight cook warning — flags brutal nights for long cooks.
> [...]

**Round 3 dialogue (correct style):**

> **F1 (Cut-specific scoring).**
> - Carl: "95% of users smoke pork shoulder, which is forgiving. Over-engineering."
> - Pete: "Wrong audience read. Free tools that respect the craft retain. A brisket guy who sees brisket-specific advice bookmarks it."
> - Dana: "Defensible from physics. Brisket cooks 12-18 hours, chicken 90 minutes."
> - Wade: "If you give me a dropdown I'll use it once."

**Round 5 vote table (correct style):**

| # | Feature | Pete | Dana | Sam | Carla | Wade | Steve | Carl | Result | Ruling |
|---|---|---|---|---|---|---|---|---|---|---|
| F1 | Cut-specific scoring | K | K | K | K | K | K | D | 6K/1D | **Keep** |

## Pitfalls to avoid

- **Generic panelists.** "Marketing Expert" and "Engineer" are not panelists. "Conversion Carla who optimizes for email-capture rates and thinks every feature should funnel somewhere" is a panelist. Names + specific lenses + opinions.

- **Polite agreement.** If panelists keep saying "great point, building on that...", you've failed. Real experts cut each other off and call ideas dumb.

- **Premature consensus.** Round 3 is for tension. If you collapse to consensus in Round 3, Round 5 has nothing to do.

- **Soft contrarian.** "I see some downsides too..." is not contrarian. "Drop it. Here's why every argument for it is wrong" is contrarian.

- **Hedging tiebreaks.** "Both sides have merit, I'll defer to majority" is failure. Pick a side and defend it.

- **Skipping the tappable prompts.** If you ask "what's your panel size?" as text instead of using `ask_user_input_v0`, you've broken the user experience this skill was designed for.
