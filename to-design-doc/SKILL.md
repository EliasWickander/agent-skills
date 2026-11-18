---
name: to-design-doc
description: "Writes a player-facing design document for a gameplay feature, as the source of truth for what the feature does, and links it from the game-design skill. Use when the user wants to write up, record, or document a feature's design. Do not use for implementation specs; use to-spec for those."
disable-model-invocation: true
---

Write one design doc under `Docs/Design/` and link it from [game-design](../game-design/SKILL.md). The doc is the current, final decision on **what** the feature does for players; how it is built belongs in a `to-spec` spec.

- The doc contains only decisions the user made or accepted. Never write an open question, placeholder, or guess into it.
- Only write the doc and the `game-design` link.

## Process

1. **Gather context.**
   - Read `game-design` and use its vocabulary.
   - If `Docs/Design/` already has a doc for this feature, update it in place.
   - Search the source code for shipped behavior the design contradicts or already settles.

2. **Find the gaps.** Walk every rule topic and the template's Edge Cases against what the conversation settled. An inference, a default you would pick, or a recommendation the user never answered is a gap.

3. **Close the gaps.** If there are any, run [grill-me](../grill-me/SKILL.md) on them before writing. Do not reopen decisions already made. In the questions:
   - Recommend a starting default for numbers only playtesting can settle.
   - When shipped code already settles a gap, say so and recommend the code's behavior.
   - If the user defers a gap, leave that behavior out of the doc and list it in the report.

4. **Write the doc** to `Docs/Design/<feature-name>.md` using [the template](assets/design-doc-template.md) and Writing Rules. If the design covers more than one feature that could ship on its own, write one doc per feature. If writing reveals a new gap, close it as in step 3 before saving.

5. **Link it from `game-design`.** Follow Linking.

6. **Check** that every rule traces to something the user said or accepted, that nothing contradicts `game-design` or `AGENTS.md` beyond changes applied per Linking, and that an agent with only this doc and `game-design` could explain the feature to a player.

7. **Report** the doc path, the `game-design` edits made and proposed, and any deferred behavior. Offer `to-spec` as the next step.

## Writing Rules

- **Player perspective.** Describe what players see, decide, and experience. Use game verbs (buy, place, send) over input verbs (click, drag), unless the input itself was decided.
- **No implementation.** No code names: classes, components, networking or event types, data asset types, or packages. Describe rejections as what the player sees.
- **Decisions only.** State each rule as the current decision, with no reasons, alternatives, or history.
- **Checkable.** Two readers must agree whether a rule holds. Replace vague words (smooth, fair, intuitive, quickly) with a number, a property, or an observable outcome.
- **No magic numbers.** Every number is a property with a default, or a fixed rule.
- **Self-contained.** No "as discussed". Define any term that is not in `game-design`.
- **Minimal.** Never say the same thing twice. Omit template sections with nothing to say.

<rule-example>
Bad:
- Healing potions should be easy to use but not overpowered.

Good:
- A player can drink a healing potion to restore `PotionHeal` health.
- A player can drink at most one potion every `PotionCooldown` seconds.
</rule-example>

## Linking

`game-design` stays the summary and index; the doc holds the full rules.

1. Find the `game-design` section that owns the feature. If none fits, add a `##` section with a one-to-three sentence summary above "Systems Not Yet Designed".
2. At the end of that section, add `` Full design: `Docs/Design/<feature-name>.md` ``.
3. If the doc changes a rule that `game-design` states, update that sentence to match.
4. Never delete or condense existing `game-design` content. Propose trims in the report.
5. If the feature is under "Systems Not Yet Designed" or marked "design note — not implemented", append the link and propose removing the entry in the report.
