# Midnight Konbini — Mobile Game Design v2

## 1. Product Direction

Midnight Konbini is a **mobile-first tactical time-management game** about surviving a late-night convenience-store shift alone.

The game is **not** a management dashboard and **not** a free-roaming virtual-joystick game.

The store itself is the main screen. The player reads the situation visually, taps a person or fixture, chooses an action, and the world advances.

The core fantasy is:

> 「深夜コンビニを一人で回して、次に何を片付けるかを瞬時に決める」

The fun comes from prioritization under pressure:

- checkout now, or restock the shelf first?
- clean the spill before someone complains, or stop a suspicious customer?
- help the impatient customer, or prepare the hot-food order?
- spend energy to fix a problem immediately, or accept a small loss and keep the line moving?

---

## 2. Platform / Input

### Primary platform

- smartphone browser
- portrait orientation first
- touch-first UI
- one-handed play should be possible

### Input model

No virtual joystick.

1. Tap a customer, fixture, queue, or incident in the store.
2. A contextual action tray appears at the bottom.
3. Tap one action.
4. The clerk performs the action.
5. Time advances by one or more ticks.
6. All customers and incidents update.

The store remains visible during almost every interaction.

---

## 3. Session Structure

A single shift should take approximately **5–8 real minutes**.

Initial target:

- Shift: 00:00 → 05:00
- 24–36 meaningful player actions
- Increasing customer pressure over the shift
- Final score / rank at dawn

The game should be restartable immediately.

Long-term progression can be added later, but the first playable version must already be fun without meta-progression.

---

## 4. Core Loop

```text
Customer / incident appears
        ↓
Player reads the store visually
        ↓
Tap target
        ↓
Choose contextual action
        ↓
Action consumes time + energy
        ↓
World advances
        ↓
Customers move / patience drops / events develop
        ↓
Money, reputation and risk change
        ↓
Repeat until 05:00
```

The key design rule is:

> Every action must create an opportunity cost.

There should rarely be an obviously correct action.

---

## 5. Game State

The first playable version should keep state small and legible.

```text
ShiftState
├─ time
├─ money
├─ reputation
├─ energy
├─ combo
├─ customers
├─ incidents
├─ stock
├─ queue
└─ result modifiers
```

### Top HUD

Only four persistent values:

- `02:15` — shift time
- `¥4,820` — sales
- `★ 83` — reputation
- `⚡ 61` — energy

No desktop-style KPI cards.

---

## 6. Customers

Customers are autonomous actors.

### Customer phases

```text
Entering
→ Browsing
→ WantsHelp? / Choosing
→ CarryingItems
→ Queueing
→ Checkout
→ Leaving
```

Alternative outcomes:

```text
Impatient → Leaves
Suspicious → TheftAttempt
Angry → Complaint
```

### Customer attributes

Each customer has:

- patience
- basket value
- archetype
- current intent
- target fixture
- risk

### Initial archetypes

Use the existing sprite sheets:

- office worker
  - high basket value
  - low patience
- adult woman
  - average patience
  - may ask for help
- student
  - low basket value
  - higher chance of browsing / suspicious event

The same visual archetype can receive different randomized behavior per run.

---

## 7. Player Actions

Actions are contextual. Do not show a permanent wall of buttons.

### Customer

- `会計する`
- `案内する`
- `声をかける`
- `見守る`

### Shelf / fridge

- `補充する`
- `前出しする`

### Floor incident

- `清掃する`

### Register

- `レジに入る`
- `まとめて会計`

### Self

- `小休憩`

Each action has:

- tick cost
- energy cost
- result
- possible side effects

Example:

| Action | Time | Energy | Effect |
|---|---:|---:|---|
| Checkout | 1 | 2 | sale + queue reduction |
| Help customer | 1 | 1 | patience up / basket value chance |
| Restock | 2 | 4 | stock recovery |
| Clean | 2 | 3 | removes complaint risk |
| Watch suspicious customer | 1 | 1 | lowers theft risk |
| Short break | 2 | -12 | energy recovery, world still advances |

---

## 8. Pressure Systems

The game needs overlapping problems, not raw stat management.

### 8.1 Queue pressure

Customers waiting at the register lose patience every tick.

Ignoring the queue may cause:

- lost sale
- reputation penalty
- angry customer event

### 8.2 Stock pressure

Popular fixtures slowly lose stock.

Low stock causes:

- lower basket values
- customers asking for help
- lost purchases

### 8.3 Cleanliness incidents

Random spill / trash incidents appear.

Ignoring them increases complaint probability over time.

### 8.4 Suspicious customer

A customer may gain a subtle suspicious indicator.

The player can spend time watching / speaking to them.

Ignoring them may result in theft, but reacting unnecessarily wastes time.

This creates a real decision rather than a mandatory chore.

### 8.5 Energy

Actions consume energy.

Low energy makes expensive actions cost additional ticks.

Taking a break recovers energy but allows the entire store to progress without the player.

---

## 9. Combo / Skill Expression

The game should reward reading the room and batching actions.

Examples:

- checkout several ready customers consecutively → checkout combo
- restock before a customer reaches an empty shelf → proactive bonus
- resolve an incident just before its danger threshold → clutch bonus
- finish the shift with no abandoned baskets → perfect service bonus

The player should feel that a good run was caused by good decisions, not random numbers.

---

## 10. Random Events

Events should create tactical situations rather than modal text walls.

Initial event pool:

- delivery arrives
- coffee machine trouble
- floor spill
- sudden customer rush
- suspicious customer
- customer asks where an item is
- freezer warning

Events appear visually in the store with a small icon/bubble.

Do not stop the game with large dialogs unless it is a major shift event.

---

## 11. Win / Loss / Scoring

The player always reaches dawn unless the store completely collapses.

This is primarily a score game, not a binary survival game.

At 05:00 show:

- sales
- completed customers
- abandoned customers
- theft loss
- complaints
- remaining energy
- combo bonus
- final rank

Ranks:

```text
S / A / B / C / D
```

A bad run should still be short and funny enough to retry.

---

## 12. Mobile UI

### Portrait layout

```text
┌──────────────────────────┐
│ 02:15   ¥4,820   ★83 ⚡61 │  compact HUD
├──────────────────────────┤
│                          │
│                          │
│        STORE WORLD       │
│                          │
│      customers / FX      │
│                          │
│                          │
├──────────────────────────┤
│ selected target / status │
│ [ action ] [ action ]    │  bottom action tray
│ [       action        ]  │
└──────────────────────────┘
```

### Store viewport

- fills most of the screen
- no permanent sidebars
- no minimap
- no desktop management menu
- no giant title header
- no KPI card grid

The player may pan the store only if necessary; the preferred v1 layout should fit the critical store area inside one portrait viewport.

### Interaction feedback

- selected target gets a soft outline / glow
- urgent customer gets a small patience bubble
- incident gets an icon above the affected position
- successful action shows a short floating result (`+¥420`, `★+2`)

---

## 13. Store Layout

The store layout should be compact and readable on a phone.

Recommended v1 zones:

```text
back wall: drink fridge / ATM
middle: snack shelf / bento shelf / freezer
front-left: register
front-center: entrance
front-right: small utility area
```

Do not try to represent an entire realistic convenience store.

The store is a game board.

Each fixture exists because it creates a decision or customer route.

---

## 14. Existing Assets We Can Reuse

Current asset manifest already provides enough visual material for the first playable version:

### Characters

- clerk male
- clerk female
- office worker
- adult woman
- student

### Fixtures

- ATM
- automatic door
- coffee machine
- freezer
- register counter / terminal
- snack shelf
- bento shelf
- drink shelf / wall fridge
- trash bin
- plant
- security camera
- etc.

No additional art is required before implementing the first playable loop.

---

## 15. Architecture v2

Old dashboard-oriented code is not a compatibility requirement.

Target source structure:

```text
src/
├─ app.ssrg              # production entry / root composition
├─ model.ssrg            # ADTs and state structures
├─ simulation.ssrg       # pure world update rules
├─ customers.ssrg        # customer behavior / intents
├─ actions.ssrg          # player action definitions + costs
├─ world.ssrg            # store world rendering
├─ hud.ssrg              # compact top HUD
├─ action_tray.ssrg      # mobile contextual controls
└─ assets.ssrg           # sprite paths
```

### Important boundary

`simulation.ssrg` should be UI-independent.

Conceptually:

```text
GameAction -> GameState -> GameState
```

DOM / signals only connect the pure simulation to the UI.

This allows the gameplay rules to evolve without rebuilding the entire view.

---

## 16. State Modeling

Prefer ADTs for domain state.

Example concepts:

```text
CustomerPhase
  = Entering
  | Browsing
  | NeedsHelp
  | Queueing
  | Leaving

Incident
  = Spill
  | TheftRisk
  | MachineTrouble
  | Delivery

Target
  = CustomerTarget Int
  | FixtureTarget FixtureId
  | IncidentTarget Int
  | NoTarget
```

Avoid encoding meaningful game state as collections of unrelated booleans.

---

## 17. First Playable Milestone

The first playable build is complete when all of the following work:

1. portrait mobile layout
2. store fills the main viewport
3. three customers enter and progress autonomously after player actions
4. customers browse and queue
5. checkout action produces money
6. customer patience decreases
7. at least one customer can abandon the store
8. restock action affects customer purchases
9. one random incident type exists
10. shift ends and displays a result rank

This is the point where the project becomes a **game**.

Anything that does not help reach this milestone is lower priority.

---

## 18. Explicitly Discarded From the Old UI

The following concepts should not return as primary UI:

- desktop admin sidebar
- KPI card grid
- minimap panel
- task queue panel
- WORLD STATE debug panel
- manual `1 step / 4 steps` debug controls
- large static management dashboard layout

Debug information may exist behind a development-only flag, but never as the production game surface.

---

## 19. Design Principle

The guiding question for every feature is:

> 「今この瞬間、どれを先にやるか迷うか？」

If a feature does not create an interesting priority decision, it should probably not be in the core game.
