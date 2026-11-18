---
name: game-design
description: >
  Game design document for Project TD: the rules, systems, and vocabulary of the
  game. Use when implementing or modifying units, waves, economy, grids, shops,
  upgrades, or any other gameplay feature.
---

# Game Design — Project TD

## How to Use This Document
- When a design describes a relationship between systems (e.g., gold funds workers, workers produce wood), preserve that dependency chain in code architecture.

## Core Concept

Two teams (2v2 or 4v4) compete to destroy the opposing team's king. Each player owns a lane. Waves of attackers assault all lanes simultaneously. If attackers survive a player's lane, they leak through and damage the team's king. Last team with a living king wins.

## Map Layout

The map for each team is H-shaped. Each leg of the H is one player's lane. Attackers spawn at the outer tips of each leg and walk inward through the player's defenders. Leaked attackers funnel into the central horizontal bridge and travel along it toward the king, which sits at one end of the bridge.

The bridge is a shared pressure gauge — the team can visually see accumulated leaks gathering there.

## Core Loop (per round)

### Planning Phase (~30 seconds)

Players spend gold to buy and place defenders on their lane grid. Defenders are auto-battlers positioned on a grid. Each round offers a new shop roll of defenders to pick from (see Defender Shop). Some defenders can upgrade to stronger versions for gold.

Players interact through UI — no physical builder unit. Click a shop card then click a grid cell to buy and place. Drag to reposition. Right-click to sell (see Defender Shop for refunds). Repositioning and selling input are not implemented yet.

Players can also spend wood to queue mercenaries against the enemy team. Mercenary sends are hidden until battle begins.

Key decisions each round:
- Which defenders to buy and where to position them
- Whether to invest in economy (workers) or defense (defenders)
- Whether to spend wood on mercenaries to pressure the enemy

### Combat Phase

Attackers spawn at the outer tip of each lane leg and walk inward. Combat resolves automatically — defenders attack, cast abilities, clear the wave or get overwhelmed.

- Clear: all attackers die before leaving the lane.
- Leak: surviving attackers reach the bridge and walk toward the king. Leaking has no direct gold penalty and gives the opposing team no bonus; the cost is the kill bounties the player missed.

After battle, all defenders are fully restored. No damage carries between rounds.

### Bridge Reinforcement (design note — not implemented)

When a player clears their lane, their surviving defenders teleport to the bridge to catch teammates' leaks. Kill bounties earned there go to the defender's owner. Leaked attackers killed by the king award no bounty.

### Arena Rounds (design note — not implemented)

Every 10th round is an arena round where the teams fight each other. During arena rounds, every player's economy is visible to everyone.

## Central Strategic Axis

Economy vs defense. Spending more on defense means you survive this wave but fall behind economically. Investing in economy means a stronger late game but risking a leak now. Every round forces this tradeoff.

## Economy

Two currencies:

**Gold** — spent on defenders. Earned from three sources:
- **Kill bounty**: each attacker's gold reward, paid immediately to the owner of the defender that landed the killing blow.
- **Wave reward**: a per-wave amount, paid to every player when the wave completes, whether or not they leaked.
- **Income**: a per-player amount, paid at the end of each wave. It grows as you invest in economy.

A player's gold and income are hidden from the opposing team (except during arena rounds). Teammates can see each other's income.

**Wood** — generated passively by workers over time. Workers cost gold to train (this is the economy investment). Wood is spent on mercenaries. Spending wood also permanently increases income.

The economy loop: Gold → Workers → Wood → Mercenaries (pressure enemies) + Income (more gold next round).

## Defender Shop

Each player has a private shop panel of defender cards. Only the owner can see their shop — not teammates, not opponents.

### Rolls
- The shop shows a fixed number of cards (default 5).
- Every player rolls independently from the shop's defender list. There is no shared or limited pool.
- Every defender in the list has equal odds. There are no tiers or rarities.
- A roll never contains the same defender twice. If the list has fewer defenders than shop slots, the shop shows one card per defender.
- A fresh roll happens at the start of every planning phase, including round 1. Unbought cards are discarded. There is no shop lock.

### Buying
- Buying does not consume the card. A player can buy the same card any number of times as long as they can afford it, so shop slots are never empty.
- Buying and placing are one action: the player clicks a card, then clicks a free cell on their own lane. Gold is spent only when the defender is placed. There is no bench.
- Cards the player cannot afford are disabled.
- The only limit on the number of defenders is free grid cells.

### Rerolling
- During the planning phase a player can pay gold (default 2) to reroll. A reroll replaces every card.
- Rerolls are unlimited.

### Selling
- Selling a defender in the same round it was bought refunds 100% of the price paid.
- Selling it in any later round refunds 50% of the price paid, rounded down.
- Defenders not bought from the shop (server-authored layouts) refund nothing.

### Visibility
The shop panel is shown for the whole planning phase and hidden during combat. Starting combat cancels any placement in progress.

### Not part of the shop
No star upgrades from duplicates, no synergies or traits, no bench, no player levels, no unit cap.

## Mercenaries (Offensive Mechanic)

Players spend wood to hire mercenaries that spawn alongside regular wave attackers in opponent lanes. This means:
- You defend against waves plus any mercenaries sent at you
- You choose when to send mercenaries to pressure the enemy
- Sending when the enemy is greedy or has a weak wave matchup can cause a leak
- Sends are hidden until battle begins, creating mind games

## Wave Matchups

Each wave has specific attack and defense types. Some defenders are strong against certain waves and weak against others. Planning builds around upcoming waves is a key skill.

## Camera

The camera is either Focused on one target (a lane on the player's own team, their own base, or the enemy base) or in an Overview of the whole map. There is no free cam. The camera starts each match, and returns at each planning phase, on the player's own lane. When a king dies, the game-over sequence takes over the camera.

Teammate awareness is also handled through UI panels (status indicators, income, lane health), not only the camera.

Not designed: camera behavior during arena rounds, direct lane-to-lane navigation without passing through the Overview, and the game-over sequence.

Full design: `Docs/Design/camera-navigation.md`

## Fog of War (design note — not implemented)

Enemy lanes are hidden from the opposing team.

- The fog covers each enemy lane from its attacker portal up to the lane/bridge junction. Inside it the player sees no defenders, attackers, or mercenaries — including their own mercenaries.
- The enemy bridge and king are never fogged. Any unit standing there during combat is visible, including units the player's team leaked or sent as mercenaries. Own mercenaries vanish into the fog and reappear when they reach the enemy bridge.
- Own-team lanes, bridge, and king are always fully visible.
- In Overview, fogged lanes show their terrain darkened with no units, so the enemy layout stays readable and the darkening signals "not scoutable". They are not clickable.
- The fog lifts when the match ends.
- Fog is a separate feature from the camera. Until it is built, enemy lanes are simply visible but not clickable in Overview.
- Fog behavior during arena rounds is not designed.

## Systems Not Yet Designed

The following systems have no approved design. Do not implement or stub their behavior without explicit instruction; if a task touches one of these, stop and ask.

- Defender upgrade paths (star system, linear, branching)
- Synergies or composition bonuses between defender types
- Bridge defense beyond bridge reinforcement (e.g., can players build directly on the bridge? Is it a shared build zone?)
- Cross-lane interaction mechanics beyond mercenaries
- Fog of war implementation, and fog/camera behavior during arena rounds
- Commander/avatar system during combat phase
- Comeback mechanics for struggling players
- Win condition details (king HP scaling, sudden death, tiebreakers)
