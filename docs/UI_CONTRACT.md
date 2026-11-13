# MoneyVerse — UI Contract

> **Purpose:** Define the behaviour, data contracts, routes, accessibility requirements and test hooks that every MoneyVerse UI must preserve.
>
> The visual design may change completely. The implementation must continue to work with the existing headless logic, game engines, Zustand store, game adapters and automated tests.

**Repository:** [universon-07/MoneyVerse](https://github.com/universon-07/MoneyVerse)  
**Default branch:** `main`  
**Application:** React 19 + TypeScript + Vite  
**Status:** Living implementation contract

---

# 1. Project Architecture

MoneyVerse is an interactive financial-literacy learning application built around four pillars:

| Pillar ID | Learning area | Game |
| --- | --- | --- |
| `plan` | Planning | Festival Showdown |
| `save` | Saving | Pocket Panic |
| `invest` | Investing | Deal Board |
| `safe` | Financial safety | Case Files |

The application uses three primary architectural layers.

| Layer | Location | Responsibility |
| --- | --- | --- |
| Headless logic | `src/host`, `src/features`, `src/progress`, `src/happy`, `src/store` | State, calculations, game integration, view-models and actions |
| UI | `src/ui`, `src/home`, `src/screens`, `src/components` | Rendering, navigation, interactions and presentation |
| Game implementations | `src/games`, `src/pillar4` | Individual game engines, game state, learning content and game-specific UI |

Additional application infrastructure:

- `src/teacher`: Teacher portal, analytics, classes, reports and student progress sharing.
- `src/island`: Island map, movement and interaction.
- `src/content`: Pillar definitions, personas and shared application data.
- `src/locales`: Translation resources.
- `src/art`: Shared art components, palette and visual effects.
- `src/lib`: Shared utilities, randomisation, development flags and performance helpers.

## Core architectural rules

1. UI components must not independently recalculate business or progress metrics.
2. Shared progress calculations must remain in pure functions.
3. Screens should consume existing hooks and view-models.
4. Game-specific calculations belong to their respective game engines.
5. Games communicate with the main application through the host integration contract.
6. A redesign must not bypass the Zustand store.
7. Existing game saves and migration compatibility must be preserved.
8. Visual styling must never become a dependency of application logic.

The final design, theme, layout and input scheme are not fixed by this document.

---

# 2. Technology and State Management

## Application stack

| Responsibility | Technology |
| --- | --- |
| UI framework | React 19 |
| Language | TypeScript |
| Build tool | Vite |
| Routing | React Router DOM 7 |
| State management | Zustand 5 |
| Persistent storage | Zustand persist + localStorage |
| Animation | Framer Motion |
| Styling | Tailwind CSS 4 + CSS |
| Internationalisation | i18next + react-i18next |
| Game pathfinding | EasyStar.js |
| Backend | Supabase |
| Unit testing | Vitest |
| Component testing | Testing Library |
| Browser testing | Playwright |

## Global store

Primary file: `src/store/game.ts`

The global application store is exposed through:

`useGame`

The UI must use the store's existing actions instead of directly modifying persisted data.

Important stored values include:

- `language`
- `persona`
- `storySeen`
- `celebrated`
- `liteMode`
- `happyToastSeen`
- `coachTipsSeen`
- `avatar`
- `gender`
- `festival`
- `installId`
- `classCode`
- `rollNo`
- `startAmount`
- `goalAmount`
- `happy`
- `pillars`
- `skills`
- `diamonds`
- `shieldPieces`
- `title`
- `lifeMomentsUsed`
- `lifeMomentSpend`
- `sessionsSinceBreak`

Derived values:

- `savedAmount`
- `completedPillars`

Derived values must never be treated as independently persisted state.

### Store actions

Important existing actions include:

- `setLanguage`
- `setPersona`
- `setStorySeen`
- `setAvatar`
- `setGender`
- `setFestival`
- `setClassCode`
- `setRollNo`
- `applyGameResult`
- `finishSession`
- `applyLifeMoment`
- `applyBurnout`
- `markHappyToastSeen`
- `markCoachTipSeen`
- `dismissAllCoachTips`
- `resetProgress`

Development-only actions:

- `devResetTips`
- `devSetHappy`
- `devForceComplete`

### Persistence requirements

- Preserve the existing Zustand persistence mechanism.
- Do not replace the storage key or version casually.
- Keep migration support in `src/store/persisted.ts`.
- Never overwrite an existing player's progress during a UI redesign.
- Preserve multiple saved-game slots managed by `src/home/slots.ts`.
- Keep autosave and restore behaviour working.

---

# 3. Routing Contract

The main route registration is in `src/App.tsx`.

The shared route table is in `src/ui/screens.ts`.

A new UI should update the appropriate route entries rather than introduce an unrelated routing system.

## Registered routes

| Route | Current component | Purpose |
| --- | --- | --- |
| `/` | `Landing` | Main application landing / island experience |
| `/story` | `Story` | Introductory story |
| `/dashboard` | `Dashboard` | Existing legacy dashboard |
| `/play/:pillar` | `GameHostScreen` | Game hosting |
| `/progress` | `ProgressScreen` | Overall learning progress |
| `/teacher` | `TeacherScreen` | Teacher portal |
| `/wire` | `WireIndex` | Developer UI index |
| `/welcome` | `OnboardingScreen` | Onboarding |
| `/hub` | `HubScreen` | Pillar hub |

### Route ownership

`src/ui/screens.ts` contains the route table consumed by `App.tsx` for the headless-feature screens.

Its current entries are:

- `/play/:pillar`
- `/progress`
- `/teacher`
- `/wire`
- `/welcome`
- `/hub`

The legacy routes `/`, `/story` and `/dashboard` are registered directly in `App.tsx`.

The home/save-slot experience is handled by `src/home/Home.tsx` before the regular application screens are mounted.

**Important:** `buildSummaryView()` exists, but a dedicated `/summary` route is not currently registered. A redesign must not assume that this route already exists.

### Invalid routes

An invalid pillar passed to `/play/:pillar` must redirect to `/`.

The valid pillar IDs are:

- `plan`
- `save`
- `invest`
- `safe`

### Developer routes

`/wire` is intended for development and wireframe inspection.

Developer functionality is controlled through `src/lib/dev.ts`.

Development tools must not become mandatory for normal application operation.

---

# 4. Game Host Contract

Primary files:

- `src/host/useGameHost.ts`
- `src/host/flow.ts`
- `src/host/types.ts`
- `src/host/registry.ts`
- `src/host/config.ts`
- `src/host/session.ts`
- `src/host/resultView.ts`
- `src/host/adapters/`
- `src/host/wrappers/`
- `src/host/placeholders/`

UI implementation:

`src/ui/wire/GameHostScreen.tsx`

Route:

`/play/:pillar`

## Host responsibilities

The host is responsible for:

1. Validating the pillar.
2. Selecting the correct game.
3. Generating a deterministic seed.
4. Providing standard game props.
5. Loading the real game or development placeholder.
6. Handling game completion.
7. Preventing duplicate completion.
8. Adapting game-specific results.
9. Committing progress.
10. Handling replay and exit.
11. Displaying the result.

The host owns these responsibilities. A new UI must not duplicate them.

## Host flow

The host state machine supports:

| Phase | Meaning |
| --- | --- |
| `entry` | Initial host state |
| `playing` | Game is running |
| `result` | Game has finished |
| `error` | Game completion or loading failed |

Transitions:

`entry → playing → result`

Replay:

`result → playing`

Failure:

`playing → error`

The current `GameHostScreen` automatically calls `host.start()` when it enters the initial phase. It does not currently render a separate entry panel.

A future design may introduce a new entry experience, but it must coordinate with the existing host lifecycle.

## Pillar registry

`PILLAR_GAMES` is defined in `src/host/registry.ts`.

Each registry entry provides:

- `pillar`
- `nameKey`
- `placeholder`
- `adapter`
- `load()`

Game folder mapping:

| Pillar | Folder | Entry |
| --- | --- | --- |
| Plan | `src/games/showdown` | `ShowdownGame.tsx` |
| Save | `src/games/cafe` | `PocketPanic.tsx` |
| Invest | `src/games/dealboard` | `ui/DealBoard.tsx` |
| Safe | `src/pillar4` | `Pillar4.tsx` |

The registry uses lazy loading and wrappers to integrate the games.

Do not hardcode individual game implementations into the host screen.

## Game props

Every host-integrated game receives the standard `GameProps` interface:

```ts
interface GameProps {
  seed: string
  cohortCode: string | null
  lang: Lang
  festival?: Festival
  avatar?: Avatar
  level?: GameLevel
  classCode?: string | null
  installId?: string
  onComplete: (payload: unknown) => void
  onExit?: () => void
}
```

The game must call `onComplete` with its own result payload.

The adapter converts that payload into the shared `GameResult`.

Games must not directly update global progress.

## Game result contract

```ts
interface GameResult {
  pillar: PillarId
  cleared: boolean
  quality: number
  partial: number
  skills: Partial<Record<Skill, number>>
  lessonsSeen: string[]
  joySignals: JoySignals
  rewards?: GameRewards
  raw: unknown
}
```

Important rules:

- `quality` is a learning/judgement score, not financial profit.
- `partial` represents progress toward clearing.
- `skills` represent skills practised.
- `lessonsSeen` records learning outcomes.
- `joySignals` provides optional Happy Index signals.
- `rewards` contains optional game rewards.
- `raw` preserves the original game payload for logs and debugging.

## Session safety

`SessionGuard` in `src/host/session.ts` ensures a session is processed once.

`finishSession()` performs:

1. Capture the before snapshot.
2. Apply the game result.
3. Apply burnout exactly once.
4. Capture the after snapshot.

The store exposes `finishSession(result)`.

A new UI must not separately apply the result and burnout after the host has already committed the session.

## Seed and replay

`seedFor(installId, pillar, salt)` is responsible for seed generation.

Rules:

- Normal replay uses the same seed.
- A new game session receives a new session ID.
- The developer new-seed action increments the salt.
- Replaying must not accidentally change the seed.
- Game-specific saved state must be reset through the existing restart mechanism.

## Host layout requirements

The game area must:

- Use available width and height.
- Support a minimum width of 360px.
- Avoid unnecessary fixed dimensions.
- Support portrait-oriented game layouts.
- Support wide games where configured.
- Keep the host header outside the game area.
- Keep the informational banner outside the game area.
- Allow the game to fill its allocated container.

Current wide games:

- `plan`
- `save`
- `invest`

The `safe` game uses the portrait-oriented host layout.

## Exit behaviour

- Exiting while playing requires confirmation.
- Exiting outside the playing phase leaves immediately.
- Confirmation must provide both confirm and cancel actions.
- The game must never be left in a broken intermediate state.

## Crash handling

`GameErrorBoundary` must catch rendering errors.

Lazy loading must display a loading state.

A game crash must never leave the user with a blank screen.

The error panel must provide a way to return to the main application.

## Host test IDs

Keep these identifiers stable:

| Element | Test ID |
| --- | --- |
| Host root | `game-host` |
| Exit button | `game-exit` |
| Developer seed button | `game-new-seed` |
| Exit confirmation | `game-exit-confirm` |
| Confirm exit | `game-exit-yes` |
| Cancel exit | `game-exit-no` |
| Game container | `game-mount` |
| Loading state | `game-loading` |
| Informational banner | `game-banner` |
| Error panel | `game-error` |
| Error back button | `game-error-back` |

---

# 5. Result Screen Contract

Files:

- `src/host/resultView.ts`
- `src/ui/wire/ResultScreen.tsx`
- `src/ui/wire/BalanceCard.tsx`
- `src/ui/wire/TiredNote.tsx`

The result screen consumes:

`buildResultView(outcome)`

It must not independently calculate the outcome.

## ResultView

```ts
interface ResultView {
  pillar: PillarId
  cleared: boolean

  state: {
    status: PillarStatus
    brightness: Brightness | null
    partialPercent: number
  }

  improved: boolean
  qualityPercent: number

  skills: {
    skill: Skill
    percent: number
  }[]

  lessons: string[]

  saved: {
    before: number
    after: number
    delta: number
  }

  happy: {
    before: number
    after: number
    delta: number
  }
}
```

## Result screen responsibilities

The result screen may display:

- Whether the pillar was cleared.
- Current pillar state.
- Whether the player improved.
- Quality percentage.
- Skills practised.
- Lessons learned.
- Goal/savings change.
- Happy Index change.
- Balance card after a clear.
- Tired note when due.
- Continue action.
- Replay action.
- Back action.
- Progress link.

The balance card and tired note are conditional UI elements.

## Result navigation

- Continue navigates to the next unfinished pillar.
- If all pillars are completed, Continue returns to the main application.
- Replay restarts the same pillar.
- Back returns to the main application.
- Progress opens `/progress`.

## Result test IDs

| Element | Test ID |
| --- | --- |
| Result root | `result-screen` |
| Heading | `result-title` |
| Pillar state | `result-state` |
| Score | `result-quality` |
| Score ring | `result-ring` |
| Goal change | `result-goal` |
| Saved amount | `result-saved-after` |
| Happy change | `result-happy` |
| Happy amount | `result-happy-after` |
| Skills | `result-skills` |
| Lessons | `result-lessons` |
| Continue | `result-continue` |
| Replay | `result-replay` |
| Back | `result-back` |
| Progress link | `result-progress-link` |

---

# 6. Progress System

Files:

- `src/features/progress/view.ts`
- `src/features/progress/hooks.ts`
- `src/progress/`
- `src/ui/wire/ProgressScreen.tsx`

Pure function:

`buildProgressView(state)`

Hook:

`useProgressView()`

## ProgressView

```ts
interface ProgressView {
  goal: {
    saved: number
    target: number
    percent: number
  }

  pillars: PillarView[]

  counts: {
    total: number
    lit: number
    bright: number
    dim: number
    flicker: number
  }

  summary: {
    key: string
    params: {
      lit: number
      total: number
      dim: number
      flicker: number
    }
  }

  happyPenalty: {
    active: boolean
    lostAmount: number
    pillarCount: number
  }
}
```

## PillarView

```ts
interface PillarView {
  id: PillarId
  nameKey: string
  status: PillarStatus

  state:
    | 'notStarted'
    | 'partial'
    | 'bright'
    | 'dim'
    | 'flicker'

  stateLabelKey: string

  icon:
    | 'empty'
    | 'half'
    | 'full'
    | 'dim'
    | 'faint'

  partial: number
  partialPercent: number

  bestQuality: number
  qualityPercent: number

  attempts: number
  lessonsSeen: string[]
  skillsTouched: Skill[]

  actionKey:
    | 'start'
    | 'continue'
    | 'replay'
    | 'brighten'
}
```

## Progress rules

- Every pillar has a visible state.
- State must be communicated with an icon and text label.
- Colour alone must never communicate progress.
- Partial progress must display its percentage.
- Quality and attempts must come from the view-model.
- Lessons and skills must come from the stored progress.
- The action label must come from `actionKey`.

### Happy penalty

A completed pillar that is not bright may reduce the Happy Index contribution.

When `happyPenalty.active` is true, explain:

1. What is being held back.
2. Why it is being held back.
3. How the player can recover.

The explanation must be supportive, simple and non-judgemental.

## Progress test IDs

| Element | Test ID |
| --- | --- |
| Screen | `progress-screen` |
| Back | `progress-back` |
| Summary | `progress-summary` |
| Goal meter | `goal-meter` |
| Goal value | `goal-meter-value` |
| Happy meter | `happy-meter` |
| Happy label | `happy-label` |
| Happy value | `happy-value` |
| Happy penalty | `happy-penalty` |
| Pillar card | `pillar-card-<id>` |
| Pillar state | `pillar-state-<id>` |
| Partial progress | `pillar-partial-<id>` |
| Details toggle | `pillar-details-toggle-<id>` |
| Details content | `pillar-detail-<id>` |
| Play action | `pillar-play-<id>` |

`pillar-card-<id>` must retain its `data-state` attribute.

---

# 7. Happy Index

Files:

- `src/features/happy/view.ts`
- `src/features/happy/hooks.ts`
- `src/happy/`

Pure function:

`buildHappyView(state)`

Hook:

`useHappyView()`

## HappyView

```ts
interface HappyView {
  value: number
  valueRounded: number
  percent: number

  band: 'glowing' | 'ok' | 'tired'

  labelKey: string

  needsBreak: boolean
  sessionsSinceBreak: number
}
```

## Happy Index rules

The Happy Index reflects the player's wellbeing within the learning experience.

The UI must:

- Display the current value.
- Display an appropriate face/icon.
- Display a text label.
- Show the meter.
- Show a gentle break reminder when needed.
- Avoid shaming the player.
- Never calculate the band independently.

### Low-Happy tip

Functions:

- `shouldShowHappyToast(state)`
- `isToastSuppressed(pathname)`

Hook:

`useHappyToast()`

Returns:

```ts
{
  visible: boolean
  dismiss: () => void
}
```

The tip is shown only when appropriate and must not cover:

- `/play`
- `/story`
- `/welcome`

Dismissal must use the existing store action `markHappyToastSeen`.

Test IDs:

- `tired-note`
- `toast-happy`
- `toast-happy-dismiss`

The tired note is rendered only when `needsBreak` is true.

---

# 8. Life Moments

Files:

- `src/features/moments/view.ts`
- `src/features/moments/hooks.ts`
- `src/happy/moments.ts`
- `src/happy/moments.json`
- `src/ui/wire/MomentsSheet.tsx`

Pure function:

`buildMomentsView(state, onlyId?)`

Hook:

`useMoments(focusId?)`

The feature allows the player to spend part of their goal progress on wellbeing activities.

## MomentView

```ts
interface MomentView {
  id: string
  nameKey: string
  descriptionKey: string

  cost: number
  gainNow: number

  faded: boolean
  affordable: boolean

  timesUsed: number
}
```

## Rules

- Display the price before the player chooses.
- Display the Happy gain before the player chooses.
- Show whether the activity is affordable.
- Show when the Happy gain has faded through repeated use.
- Show the number of previous uses where appropriate.
- Disable unaffordable choices.
- Use the existing `choose(id)` action.
- Never calculate prices or rewards in the UI.

## Test IDs

| Element | Test ID |
| --- | --- |
| Open moments | `moments-open` |
| Sheet | `moments-sheet` |
| Close | `moments-close` |
| Moment | `moment-<id>` |
| Cost | `moment-cost-<id>` |
| Happy gain | `moment-gain-<id>` |
| Unaffordable message | `moment-unaffordable-<id>` |
| Choose button | `moment-choose-<id>` |

---

# 9. Balance Card

Files:

- `src/features/balance/view.ts`
- `src/features/balance/hooks.ts`
- `src/ui/wire/BalanceCard.tsx`

Pure function:

`balanceFor(state)`

Hook:

`useBalance()`

## BalanceView

```ts
interface BalanceView {
  breaksTaken: number

  earned: number
  spentOnSelf: number
  net: number

  selfSharePercent: number

  tone:
    | 'none'
    | 'allWork'
    | 'balanced'
    | 'selfHeavy'

  whyKey: string
  nextStepKey: string
}
```

## Rules

The balance card explains the relationship between:

- Progress toward the financial goal.
- Money spent on looking after oneself.
- The importance of balancing both.

It educates. It does not provide financial advice.

The UI must:

- Display earned amounts.
- Display spending on oneself.
- Explain the balance.
- Show the relevant next-step message.
- Use the supplied `tone`.
- Never calculate the figures independently.

It must retain the educational disclaimer:

**"Made-up numbers. Real life is different."**

## Test IDs

- `balance-card` with `data-tone`
- `balance-earned`
- `balance-spent`
- `balance-next`

---

# 10. Hub and Island Experience

Files:

- `src/features/hub/view.ts`
- `src/features/hub/hooks.ts`
- `src/features/hub/config.ts`
- `src/island/`
- `src/content/island.json`
- `src/content/island.diorama.json`
- `src/ui/wire/HubScreen.tsx`
- `src/ui/wire/PillarIntroCard.tsx`
- `src/ui/wire/CoachTips.tsx`

The current product direction is a calm floating island viewed from a high three-quarter isometric perspective.

The island is the central navigation experience.

A redesign may change its visual implementation, but must preserve the underlying interactions and progress data.

## HubView

Pure function:

`buildHubView(state)`

Hook:

`useHub()`

Returns:

```ts
{
  pillars: HubPillarView[]
  enterPillar: (id: PillarId) => void
  openMoments: () => void
}
```

Each pillar includes:

- `id`
- `nameKey`
- `taglineKey`
- `teachesKey`
- `estimatedMinutes`
- `state`
- `bestQuality`
- `attempts`
- `ctaKey`
- `canReplay`

## Pillar CTA behaviour

| State | CTA |
| --- | --- |
| Not started | Start |
| In progress | Continue |
| Complete and bright | Replay |
| Complete but dim/flickering | Brighten |

Labels must come from the existing locale keys.

## Pillar introduction

Hook:

`usePillarIntro(id)`

Provides:

- What the pillar teaches.
- Estimated completion time.
- Current state.
- Best quality.
- Primary CTA.
- Optional secondary CTA.

The introduction must not independently determine whether a pillar is completed.

## Island navigation

The existing island implementation supports:

- Selecting a location.
- Moving the character/token.
- Path-based movement.
- Hotspot interaction.
- Panning.
- Zooming.
- Touch and pointer interaction.

The island configuration is data-driven.

The UI must not hardcode hotspot coordinates that belong in the existing island configuration.

## HUD

The existing shared UI includes:

- Goal meter.
- Happy meter.
- Settings/profile controls.
- Life-moments access.

These should remain accessible without obscuring the island or its interactive areas.

## Hub test IDs

- `hub-screen`
- `hub-open-moments`
- `hub-pillar-<id>`
- `hub-cta-<id>`
- `pillar-intro-<id>`
- `intro-start`
- `intro-back`

---

# 11. Onboarding

Files:

- `src/features/onboarding/hooks.ts`
- `src/features/onboarding/script.ts`
- `src/features/onboarding/config.ts`
- `src/features/onboarding/controls.ts`
- `src/ui/wire/OnboardingScreen.tsx`

Route:

`/welcome`

Hook:

`useOnboarding()`

## Responsibilities

Onboarding collects the choices required to initialise a player's experience.

Current choices include:

- Persona.
- Festival.
- Avatar.

The hook returns:

```ts
{
  stepIndex: number
  totalSteps: number
  currentStep: OnboardingStep
  progressPercent: number

  next: () => void
  back: () => void
  skip: () => void

  canNext: boolean

  choices: {
    persona: PersonaId | null
    setPersona: (id: PersonaId) => void

    festival: Festival | null
    setFestival: (id: Festival) => void

    avatar: Avatar | null
    setAvatar: (id: Avatar) => void
  }
}
```

The exact step sequence is defined by `ONBOARDING_SCRIPT`.

## Rules

- The user cannot advance past a required selection without choosing it.
- Back returns to the previous step.
- Skip uses the existing default persona behaviour.
- Completing onboarding marks the story as seen.
- Existing selections must be preserved.
- Do not independently initialise financial goals in the UI.

The onboarding screen currently redirects to `/` if `storySeen` is already true.

## Test IDs

- `onboarding-screen`
- `onboarding-step-counter`
- `onboarding-skip`
- `onboarding-title-<kind>`
- `onboarding-persona-<id>`
- `onboarding-festival-<id>`
- `onboarding-avatar-<id>`
- `onboarding-back`
- `onboarding-next`

---

# 12. Coach Tips

Files:

- `src/features/hub/coach.ts`
- `src/features/hub/coach.test.ts`
- `src/ui/wire/CoachTips.tsx`

Hook:

`useCoachTips()`

The coach system provides contextual guidance.

The UI must:

- Display only eligible tips.
- Allow individual dismissal.
- Support dismissing all tips.
- Preserve dismissal state through the existing store.
- Avoid repeatedly displaying dismissed tips.

Test IDs:

- `coach-tips-container`
- `coach-dismiss-all`
- `coach-tip-<id>`
- `coach-dismiss-<id>`

---

# 13. Summary Feature

Files:

- `src/features/summary/view.ts`
- `src/features/summary/summary.test.ts`
- `src/features/summary/share.ts`

Pure function:

`buildSummaryView(state)`

The summary view-model currently exists, but a dedicated summary screen and route are not registered.

## SummaryView

```ts
interface SummaryView {
  complete: boolean

  goal: {
    saved: number
    target: number
    percent: number
  }

  pillars: {
    id: PillarId
    nameKey: string
    state: PillarVisualState
    stateLabelKey: string
  }[]

  skills: {
    id: Skill
    labelKey: string
    level: number
  }[]

  lessonsSeen: string[]

  balance: BalanceView

  toBrighten: PillarId[]

  closingKey:
    | 'summary.closing.allBright'
    | 'summary.closing.someDim'
    | 'summary.closing.keepGoing'
}
```

## Rules

- Display all four pillars.
- Display the overall goal.
- Display skills practised.
- Display lessons learned.
- Display the balance information.
- Identify pillars that could be brightened.
- Use the supplied closing message.
- Never calculate summary metrics in the UI.

A future summary screen must be registered explicitly in the application routing contract.

---

# 14. Teacher Portal

Files:

- `src/teacher/`
- `src/teacher/TeacherScreen.tsx`
- `src/teacher/Auth.tsx`
- `src/teacher/ShareProgress.tsx`
- `src/teacher/cloud.ts`
- `src/teacher/analytics.ts`
- `src/teacher/reports.ts`
- `src/teacher/roster.ts`
- `src/teacher/storage.ts`

Route:

`/teacher`

The teacher portal is a separate experience from the student's learning hub.

Its responsibilities include:

- Teacher authentication.
- Class management.
- Student progress sharing.
- Learning analytics.
- Reports.
- Lessons and educational material.
- Student insights.

## Rules

- Respect existing Supabase integration.
- Keep authentication separate from student game state.
- Preserve privacy and sharing permissions.
- Do not expose student data without the existing sharing mechanism.
- Preserve teacher analytics and report calculations.
- The teacher portal must remain accessible without requiring a student game session.

---

# 15. Home and Save Slots

Files:

- `src/home/Home.tsx`
- `src/home/NewGameForm.tsx`
- `src/home/WelcomeScene.tsx`
- `src/home/slots.ts`
- `src/home/welcome.css`

The home experience is separate from the island hub.

It provides:

- New game creation.
- Existing game loading.
- Save-slot management.
- Rename.
- Delete with confirmation.
- Teacher portal access.
- Student progress sharing.

## Save-slot rules

- Multiple saved games are supported.
- Each slot has its own identity.
- A slot may have a custom name.
- Loading a slot must restore the correct game state.
- Starting a new game must not overwrite other slots.
- Deleting a slot requires confirmation.
- Autosave runs periodically and when the tab becomes hidden or closes.
- Save-slot migration must remain compatible with existing users.

The home flow uses a page reload after swapping the active saved game so that the Zustand store rehydrates correctly.

Do not replace this mechanism with a purely local component state implementation.

---

# 16. Game-Specific UI Boundaries

Each game owns its internal gameplay interface.

The shared MoneyVerse UI must not assume that every game has identical controls, orientation, layout or gameplay flow.

## Festival Showdown

Location: `src/games/showdown/`

Contains:

- Game engine.
- Financial concepts.
- Prediction and scoring systems.
- Learning feedback.
- Reports.
- Game-specific interface.

## Pocket Panic

Location: `src/games/cafe/`

Contains:

- Economy engine.
- Customer and mood systems.
- Pressure and balance calculations.
- Game state.
- Game-specific interface.
- Learning content.

## Deal Board

Location: `src/games/dealboard/`

Contains:

- Investment simulation.
- Company financial models.
- Market simulation.
- Investment decisions.
- Health checks.
- Judgement and review.
- Lessons.
- Analytics.
- Final report.

The game uses a learning-first approach: players must understand company information and the consequences of their decisions.

## Case Files

Location: `src/pillar4/`

Contains:

- Case engine.
- Validation.
- Progress tracking.
- Case content.
- Analyst interface.
- Investigation board.
- Report sheet.
- Replay and feedback.

It is a separate game implementation integrated through the host wrapper.

## Rules for all games

- Preserve game-specific engine logic.
- Preserve game-specific stores.
- Preserve game-specific content.
- Preserve scoring and learning outcomes.
- Preserve existing game tests.
- Do not replace game-specific state with arbitrary UI state.
- Do not directly modify the global progress store from game UI.

---

# 17. Design System and Visual Direction

Reference:

`DESIGN.md`

The current visual direction is a calm, illustrated, isometric island experience.

The latest design decisions supersede the earlier street-market hub direction.

## Current visual characteristics

- High three-quarter isometric perspective.
- Floating island environment.
- Pale blue water.
- Paper-like surfaces.
- Muted, warm colours.
- Thin ink outlines.
- DM Serif Display titles.
- Gentle character and environmental animation.
- Simple cross-fades between screens.
- Gold progression markers.
- Accessible interaction hotspots.

The design system is documented in `DESIGN.md`.

The visual direction can evolve, but the UI must preserve:

- Clear visual hierarchy.
- Readable text.
- Distinguishable interactive elements.
- Accessible focus states.
- Touch-friendly controls.
- Reduced-motion support.
- Consistent feedback.
- Responsive layouts.

## Performance

The existing application supports Lite mode.

`useLite()` and `autoLite()` are defined in `src/store/game.ts`.

The UI must respect reduced-motion and low-performance device settings.

Do not introduce animations that make the application unusable on lower-end devices.

---

# 18. Internationalisation

Current locale directory:

`src/locales/`

The active locale resource is:

`src/locales/en.json`

The application currently has an English-only product configuration.

Do not assume that Hindi and Telugu translations already exist.

All new user-facing text should:

- Use i18next.
- Use meaningful translation keys.
- Avoid hardcoded UI copy where practical.
- Preserve existing translation keys.
- Keep educational language clear and age-appropriate.

Important namespaces include:

- `host.*`
- `result.*`
- `state.*`
- `skills.*`
- `pillars.*`
- `progress.*`
- `happy.*`
- `moments.*`
- `balance.*`
- `hub.*`
- `onboarding.*`
- `coach.*`
- `summary.*`
- `home.*`

Game-specific content should continue using the relevant game content and translation systems.

---

# 19. Accessibility and Testing Contract

The UI must be testable independently of its visual styling.

Tests should locate elements using:

1. `data-testid`
2. Accessible roles
3. Accessible names
4. Semantic HTML

Tests must not depend on:

- CSS class names.
- Colours.
- Pixel coordinates.
- Specific visual layouts.
- Animation timing.
- Internal styling structure.

## Test ID convention

Use kebab-case.

Examples:

- `game-host`
- `result-screen`
- `goal-meter`
- `happy-meter`
- `hub-screen`
- `pillar-card-plan`

For dynamic elements:

`<thing>-<id>`

For example:

`moment-cost-chai`

## Accessibility requirements

- Buttons must have accessible names.
- Inputs must have associated labels.
- Modals must expose appropriate dialog semantics.
- Progress meters must expose progressbar semantics.
- State must be represented by an icon and a text label.
- Keyboard navigation must remain available.
- Focus indicators must remain visible.
- Interactive elements must not rely on hover alone.

## Testing commands

```bash
npm test
npm run lint
npm run check:tokens
npm run p4:test
npm run build
```

The build also runs the Pillar 4 validation script.

A UI redesign must not weaken existing test coverage.

---

# 20. Current UI Components

The current wireframe UI is located in `src/ui/wire/`.

| Component | Responsibility |
| --- | --- |
| `GameHostScreen` | Game lifecycle |
| `GameErrorBoundary` | Crash handling |
| `ResultScreen` | Game results |
| `ProgressScreen` | Progress overview |
| `GoalMeter` | Goal progress |
| `HappyMeter` | Happy Index |
| `TiredNote` | Break reminder |
| `BalanceCard` | Balance education |
| `MomentsOpener` | Open life moments |
| `MomentsSheet` | Life moments selection |
| `HubScreen` | Pillar navigation |
| `PillarIntroCard` | Pillar introduction |
| `OnboardingScreen` | Onboarding |
| `CoachTips` | Contextual guidance |
| `ToastHost` | Notifications |
| `WireIndex` | Developer wireframe index |

These components are replaceable presentation implementations.

The underlying headless logic should remain the source of truth.

---

# 21. Features Not to Break

The following are critical application behaviours:

1. Four-pillar progression.
2. Deterministic game seeds.
3. Same-seed replay.
4. Game adapters.
5. SessionGuard protection.
6. Result processing.
7. Happy Index calculations.
8. Burnout calculations.
9. Life moments.
10. Balance calculations.
11. Goal progression.
12. Saved-game slots.
13. Persistent player identity.
14. Store migration.
15. Teacher progress sharing.
16. Teacher analytics.
17. Game-specific learning engines.
18. Island navigation.
19. Lite mode.
20. Existing automated tests.

---

# 22. Not Yet Part of This Contract

The following should only be added after their implementation exists:

- A dedicated summary route and screen.
- A redesigned teacher portal.
- New settings and dashboard routes.
- Additional supported languages.
- New game-host entry screens.
- New game mechanics.
- Additional progression systems.

A future feature must document its routes, view-models, hooks, actions, test IDs and locale keys before becoming part of this contract.

---

# 23. Rules for Future UI Redesigns

Any developer or AI agent redesigning MoneyVerse must follow these rules:

1. Inspect the existing implementation before changing components.
2. Read `DESIGN.md` before proposing visual changes.
3. Read this contract before modifying shared screens.
4. Preserve the existing headless architecture.
5. Reuse existing hooks and view-models.
6. Preserve the global Zustand store.
7. Preserve all game adapters and game interfaces.
8. Preserve all registered routes unless explicitly changing the routing contract.
9. Preserve existing `data-testid` values.
10. Preserve accessible roles and names.
11. Never introduce duplicate calculations in UI components.
12. Never silently remove existing features.
13. Never replace a working game with a visual mockup.
14. Preserve mobile and desktop functionality.
15. Run the relevant tests after implementation.
16. Update this document when the actual architecture changes.

**The fundamental rule: MoneyVerse's UI is replaceable; its working behaviour is not.**

A redesign is successful only when the new interface looks and feels different while the existing application logic, games, persistence, progression and tests continue to work.
