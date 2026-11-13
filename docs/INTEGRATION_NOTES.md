# Integration Notes: Findings for the Game Owners

## Festival Showdown — Plan Budget

### Current observations

- Festival Showdown is located in `src/games/showdown/`.
- The game currently uses its own internal gameplay and scoring flow.
- Its completion data includes:
  - `score.goals`
  - `score.onTime`
  - `score.resilience`
  - `score.smart`
  - `score.total`
  - `stars.goals`
  - `stars.onTime`
  - `stars.resilient`
  - `stars.cleanBooks`
  - `evidence`
  - `minaretLit`
  - `lessonsSeen`

### Data and scoring considerations

- The score fields provide useful information about the player's budgeting and decision-making.
- `goals` can represent goal-setting and prioritisation.
- `onTime` can represent planning and timely financial decisions.
- `resilience` can represent the ability to recover from unexpected expenses or setbacks.
- `smart` can represent awareness of scams and the ability to make informed decisions.
- `total` represents the overall score.
- The star values provide additional information about the quality of the player's performance.

### Items to review

1. **Friendship evidence:** The current Showdown result does not consistently include a dedicated friendship signal. If friendship or collaboration is an intended learning outcome, consider recording it explicitly.
2. **Ring Toss:** Confirm that the Ring Toss result is represented consistently as a boolean value.
3. **Minaret progress:** `minaretLit` should accurately represent whether the player has completed the relevant activity.
4. **Completion timing:** Ensure the final result is generated only after all scoring and evidence updates have finished.
5. **Lessons:** Make sure `lessonsSeen` contains the lessons the player actually encountered.

### Recommendations

- Keep the score fields consistent across all game outcomes.
- Include explicit evidence for every intended learning outcome.
- Avoid deriving friendship or collaboration outcomes from unrelated score fields.
- Verify that the final completion data is available immediately after the game finishes.

---

## Pay Run

### Current observations

Pay Run is located in `src/games/payrun/`.

The game currently exposes a limited completion signal through `onGoalReached()`. Its internal state contains information such as:

- Current jar balance
- Goal amount
- Number of shifts completed

The available result is primarily based on whether the player reaches the savings goal.

### Data limitations

The current completion information does not provide a complete picture of the player's financial habits.

The following information is not currently included in the completion result:

- Detailed habit scores
- Individual financial skill scores
- Joy-related counters
- Lessons completed or encountered
- A structured record of the player's financial decisions

The game also maintains its own seed and local persistence behaviour.

### Partial completion

If a player exits before reaching the goal, the available result represents an incomplete run. This is useful for distinguishing a completed game from an abandoned attempt, but it does not provide enough detail to understand the player's progress.

### Recommendations

If Pay Run is intended to be used again, consider introducing a structured result containing:

- `goalReached`
- `jar`
- `goal`
- `shifts`
- Habit or behaviour scores
- Financial skill indicators
- Relevant Joy-related counters
- `lessonsSeen`

This would make it possible to understand not only whether the player reached the goal, but also how they managed money along the way.

---

## Deal Board — Invest

### Current observations

Deal Board is located in `src/games/dealboard/`.

The game records decision-making information through fields such as:

- `judgement`
- `skills.research`
- `skills.skepticism`
- `skills.spread`
- `skills.calm`
- `skills.habits`
- `trapsAvoided`
- `prematureSells`
- `fees`
- `concentrationPeak`

The game also supports a lightweight mode.

### Skill interpretation

The available skill indicators provide useful information about the player's investment behaviour:

| Game signal | Learning area |
|---|---|
| Research | Planning and research |
| Skepticism | Scam awareness |
| Spread | Risk management |
| Calm | Patience |
| Habits | Saving and consistency |

### Important considerations

- `judgement` is used as the main indicator of decision quality.
- A judgement score of 60 or above is treated as a successful outcome.
- The current result does not include net worth or profit as completion indicators.
- Avoid treating profit alone as proof of good investment decisions. A player can make a profit while taking unnecessarily high risks.

### Recommendations

- Continue prioritising decision quality over profit.
- Preserve the individual skill indicators because they provide more useful learning information than a single final score.
- Keep recording avoided traps, premature selling, fees and portfolio concentration.
- Ensure that the game rewards informed and balanced decisions rather than simply encouraging higher returns.

---

## Case Files — Safe

### Current observations

Case Files is located in `src/pillar4/`.

Its progress information includes:

- `status`
- `shieldPieces`
- `diamonds`
- `title`
- `playbookReady`
- `stage`
- `firstTryAccuracy`
- `lessonsSeen`

The game maintains its own progress and completion information.

### Completion and quality

- A completed status indicates that the player has finished the experience.
- `firstTryAccuracy` provides an indication of how accurately the player understood the material.
- `shieldPieces` provides another measure of progress.
- Diamonds are recorded as part of the player's progress and rewards.

### Recommendations

1. Keep completion status and learning quality as separate measures.
2. Preserve `firstTryAccuracy` because it helps distinguish understanding from simply completing activities.
3. Ensure `lessonsSeen` reflects the lessons actually encountered.
4. Keep reward information separate from learning-performance measurements.
5. Test progress reset behaviour to ensure a new attempt does not accidentally retain completion data from an earlier run.

---

## Shared Game Resources

The following shared resources are available in the project:

| File | Purpose |
|---|---|
| `src/lib/rng.ts` | Random-number generation utilities |
| `src/lib/gameResult.ts` | Shared game-result definitions |
| `src/components/Scenery.tsx` | Shared scenery components |
| `src/components/gameMotion.css` | Shared game animations |
| `src/games/shared/Toon.tsx` | Shared character and illustration components |
| `src/games/shared/palette.ts` | Shared colour palette |

### Recommendations

- Reuse shared components wherever possible.
- Keep visual and interaction patterns consistent across games.
- Avoid duplicating common utilities when an existing shared implementation is suitable.
- Keep game-specific behaviour inside the relevant game rather than introducing unnecessary dependencies between games.

---

## Randomness and Reproducibility

Some games use random values or internal seeds to determine gameplay behaviour.

For reliable testing, it is useful to make random outcomes reproducible.

### Recommendations

- Use a consistent seed when testing the same scenario repeatedly.
- Avoid changing random outcomes unexpectedly during a single run.
- Verify that restarting a game produces the intended behaviour.
- Keep random generation separate from scoring calculations wherever practical.

---

## Testing Checklist

The following checks should be performed before considering the games ready:

### Festival Showdown

- [ ] Verify all score fields.
- [ ] Verify star calculations.
- [ ] Confirm Ring Toss result handling.
- [ ] Confirm `minaretLit` is accurate.
- [ ] Verify friendship evidence, if applicable.
- [ ] Confirm lessons are recorded correctly.
- [ ] Check that the final result is generated after all updates finish.

### Pay Run

- [ ] Verify savings-goal calculations.
- [ ] Verify jar balance and shift counts.
- [ ] Test successful completion.
- [ ] Test early exit and partial progress.
- [ ] Review whether structured habit and skill results should be added.

### Deal Board

- [ ] Verify judgement calculations.
- [ ] Test all five skill indicators.
- [ ] Verify trap, fee and premature-selling counters.
- [ ] Test the success threshold.
- [ ] Confirm that profit is not being used as the only measure of good decisions.

### Case Files

- [ ] Verify completion status.
- [ ] Verify shield-piece progress.
- [ ] Check first-try accuracy.
- [ ] Confirm lesson tracking.
- [ ] Test progress reset behaviour.

### General

- [ ] Run `npm test`.
- [ ] Run `npm run build`.
- [ ] Test completed and incomplete game sessions.
- [ ] Verify that scores and learning evidence remain consistent across repeated runs.

---

## Final Notes for Game Owners

The most important improvement across the games is to make completion data meaningful and consistent.

A final score alone does not explain what a player learned. Each game should provide enough information to understand:

- What the player achieved.
- Which financial skills they demonstrated.
- Which decisions they made.
- Which lessons they encountered.
- Whether they completed the experience or left partway through.

The goal is to measure learning and decision-making, not just game completion or rewards.
