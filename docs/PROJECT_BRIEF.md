# MoneyVerse — Project Brief

**Team:** Viltrumites  
**Project:** MoneyVerse  
**Theme:** MoneyVerse — Empowering the Next Generation by Financial Literacy

*Money choices shape more than your wallet. Can you reach your goals while keeping your happiness?*

This document is the single source of truth for anyone (human or AI) working on the shared game framework of MoneyVerse.

## 1. What we are building

A financial-literacy web game for Grade 11–12 students in Hyderabad, many on low-end phones.

- **Overworld.** A Cuphead-style painted Hyderabad bazaar with the Charminar in the centre. The player walks a character to one of four shop doorways. Each doorway opens one pillar mini-game:
  1. Plan & Budget
  2. Save & Manage
  3. Understand Investing
  4. Stay Financially Safe
- **Charminar lamps.** Four minaret lamps light up as the pillars are completed.
- **Goal bar.** Lighting all four earns the student's college fee. The goal bar always shows how far along they are.
- **Happy Index.** A wellbeing meter from 0 to 100. If it is low when a pillar is cleared, that minaret lamp is dimmer and counts for less toward the goal. It teaches: do not focus only on profit; balance the future and yourself.

## 2. Scope: shared game framework (shell) and mini-games

**The shell** is the shared application surrounding the mini-games. It handles the opening story, overworld, player controls, HUD, Charminar progress page, Happy Index, game host, ending, teacher dashboard, English-language content, and offline gameplay. It provides the navigation and progress systems that connect the four mini-games into one experience.

**The four mini-games** are part of MoneyVerse. The shared game framework connects them through the contracts in Section 3.


## 3. Mini-game contracts

All four are designed for a portrait 360 px mobile layout, work offline, and do not collect students' names or contact information. **MoneyVerse is English-only.** Any language parameter is fixed to `en`; no language chooser or other translations are required.

### Festival Countdown (Plan & Budget)

```tsx
<FestivalCountdown seed={seed} cohortCode={cohortCode} lang="en" festival={festival} onComplete={onComplete} />
```

Payload:

```ts
{
  game: 'festival', cohortCode, seed, festival,
  stars: { ghar, gift, dost, plan },
  score: { needs, plan, shopping, goal, balance, total },
  evidence: { ... },
  lessonsSeen: string[]
}
```

Cleared at 3 or more stars, or `total >= 70`. A player who misses still finishes and gets advice.

### Pay Run (Save & Manage)

```tsx
<PayRun seed={seed} cohortCode={cohortCode} lang="en" onComplete={onComplete} />
```

Payload: five habit-score parts, number of shifts, whether the goal was reached, lessons seen. Cleared when the jar reaches 5000 within 12 shifts, **or** the cumulative habit score is `>= 70` at shift 12. A player who has learned the habits but not reached the goal still lights the lamp.

### Deal Board, Lite mode (Understand Investing)

```tsx
<DealBoard mode="lite" seed={seed} cohortCode={cohortCode} lang="en" onComplete={onComplete} />
```

Payload: five skill scores (research, skepticism, spread, calm, habits; 20 points each) and counters. Cleared at a judgement score `>= 60` out of 100. Profit never decides the result.

### Case Files: Mohalla Detective (Stay Safe)

Hub contract rather than a plain callback.

- **Inputs from the hub:** `avatar` (`'male' | 'female'`), `language` (`'en'` only), `level` (`'beginner' | 'confident'`), `classCode?`, `installId`.
- **Outputs to the hub:** `status` (`'locked' | 'in_progress' | 'complete'`), `shieldPieces` (subset of protect / verify / pause), `diamonds` (int), `title` (string), `playbookReady` (bool).

## 4. Orientation and layout: follow the current repo

**Standing decision: keep whatever orientation and layout the current code already has.** Do not convert the shell to landscape, and do not add a landscape stage, rotate prompts or a phone frame. If a later request proposes an orientation change, treat it as out of scope for this brief until the team explicitly revises this decision. The overworld stays full-screen and scales to the window as it does today; the story keeps its current cinema-frame stage. The mini-games are mounted into the existing screen as they are, at the size the shell gives them.

## 5. Controls: follow the current repo

**Standing decision: keep the current input as it is.** Do not add new input systems, an on-screen A button, a context stack or key remapping without an explicit team decision.

What exists today and must be kept working:

- **Keyboard:** WASD or arrow keys to walk; Enter to go into a doorway you are standing at.
- **Pointer:** tap or drag to walk with A* pathfinding; tap a shop to walk to it and enter.
- **Touch:** the existing virtual joystick (shown on touch or narrow screens).
- **Story:** Space, Enter, arrows, swipe, tap and Escape as they are now.
- **Other actions:** click or tap on-screen controls.

## 6. Hard guardrails

- Everything is fictional: no real brands or products.
- Educate, do not advise. No investment recommendations.
- Show **"Made-up numbers. Real life is different."** on money screens.
- **Students do not sign in.** Do not collect their names, email addresses, phone numbers or other direct contact details. If they choose to share progress, use the pseudonymous class-code and roll-number flow.
- **Teachers sign in** only for the teacher dashboard, where access is restricted to their own classes.
- No real-money transactions.
- No dependency on audio.
- Student gameplay works offline. Sending progress to the teacher and loading live teacher reports require a network connection.
- English-only interface, stories and game content.

## 7. UI and art are not final

Everything visual must go through **design tokens** and **thin presentational components**, so a reskin is cheap. Use `TODO(final-ui)` to identify visuals that still need final design review.
