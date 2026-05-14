# /panel — Expert Panel Skill for Claude

A Claude skill that facilitates a structured expert panel discussion to help you make a decision or design something. Spawns 6-7 distinct expert personas (including a mandatory contrarian) who independently propose ideas, then debate, then vote keep/drop/modify on each item.

Built to solve a specific frustration: "give me suggestions" prompts produce flat, polite lists that converge on the obvious answer. This skill produces opinionated, dissenting, structured debate instead.

## What's in this repo

| File | Purpose |
|---|---|
| `panel.skill` | The packaged skill, ready to install in Claude.ai |
| `SKILL.md` | The source markdown, for reading or editing before install |
| `README.md` | This file |

## Install

1. Open Claude.ai
2. Go to **Settings → Capabilities → Skills**
3. Click **Upload skill** and select `panel.skill`
4. Done — the skill is now available in every conversation

If the install UI has moved (skills are still evolving in Claude.ai), check Anthropic's current documentation at [docs.claude.com](https://docs.claude.com).

## Use

In any Claude conversation, type one of:

- `/panel`
- "Spawn an expert panel to discuss X"
- "I want a panel of experts to weigh in on Y"
- "Set up a roundtable on Z"

Claude will trigger the skill, ask for the topic in plain text, then present tappable multiple-choice prompts for the rest of the parameters (output type, panel size, composition, constraints). No typing needed beyond the topic itself.

## How the discussion runs

1. **Topic capture** — you type the topic
2. **Parameter selection** — tappable prompts for output type, panel size, composition, constraints
3. **Panel proposal** — Claude proposes a roster with named personas; you approve, swap, or simplify
4. **Round 1** — each panelist independently proposes 5-6 ideas in their own voice
5. **Round 2** — Claude consolidates and de-duplicates into a candidate list
6. **Round 3** — multi-round debate, with the contrarian getting explicit airtime
7. **Round 4** — contrarian last word on marginal items
8. **Round 5** — vote table with keep/drop/modify per panelist, ties broken by Claude
9. **Final report** — narrative summary + decision table

After the report, you get a tappable prompt to accept it, change votes, build an implementation prompt from the keepers, or re-open the discussion.

## Why a contrarian matters

The single most important design choice in this skill is the mandatory contrarian. Without one, panels degrade into agreement machines that affirm whatever you were already going to do. The skill's "rules of engagement" enforce that the contrarian's default position is "this is wrong, here's why" — not soft devil's-advocate hedging.

If you find Claude's contrarian going soft, push back on it directly. The skill instructs Claude to re-open rounds rather than just changing answers to please you.

## Use cases

The skill includes panel composition templates for four common scenarios:

- **Product/feature design** — user, domain expert, builder, monetizer, SEO, skeptic
- **Strategic decision** — optimist, pessimist, numbers, operator, customer, contrarian
- **Risk assessment** — auditor, engineer, business owner, legal, historian, contrarian
- **Creative critique** — craftsperson, audience, marketer, editor, historian, contrarian

For other topics, Claude adapts the roster to fit. You can always swap or specify roles after the initial proposal.

## Output format

Final reports include:

**Narrative summary** (2-4 paragraphs):
- Shape of the final recommendation
- Biggest unanimous decision
- Most contested decision and how it resolved
- Any pivots where discussion reframed an item
- Anything the panel didn't surface that's worth flagging

**Decision table** with default columns:

| # | Item Name | Description | Keep/Drop/Modify | Why |

Columns adapt to the topic — risk assessments get Likelihood/Impact/Mitigation, ranked options get Rank/Score/Trade-offs.

## Editing the skill

To modify behavior, edit `SKILL.md` and repackage. The skill-creator skill in Claude can help, or you can re-zip manually:

```bash
# Manual repack (the .skill format is a zip archive)
cd /path/to/working-dir
mkdir panel
cp SKILL.md panel/
zip -r panel.skill panel/
```

Common edits worth considering:

- **Default panel size** — change "Standard (6-7 panelists)" to your preferred default
- **Output types** — add domain-specific options (e.g., "Architectural review" if you do a lot of system design)
- **Panel templates** — add a template for your specific industry
- **Tone** — make the contrarian more or less aggressive

## Known limitations

- **Tappable prompts only work in Claude's consumer apps.** If you paste this skill into the API directly, into Claude Code, or into third-party tools that don't expose `ask_user_input_v0`, the prompts degrade to plain text questions. The skill still works, just less smoothly.

- **Topic must be typed.** Every topic is unique, so it can't be a preset button. Everything after the topic is tappable.

- **No persistence between sessions.** Each panel runs fresh. If you want to continue a panel from a previous session, paste the prior report back in as context.

- **Skill spec is evolving.** Anthropic's skill format may change. If install fails or behavior shifts, check the latest skill-creator documentation and repackage.

## License

MIT. Do whatever you want with it.

## Credits

Built using the `skill-creator` skill in Claude.ai. The structure (Round 1 independent proposals → Round 2 consolidation → Round 3 debate → Round 4 contrarian → Round 5 vote) is borrowed from common product-review and strategic-planning frameworks, adapted for LLM-driven facilitation.
