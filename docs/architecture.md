# Architecture

The first playable slice intentionally keeps the model and view in one Seseragi module so compiler feedback stays easy to isolate.

## Boundaries

- `GameState` is the complete reactive state for the first slice.
- `GameAction` describes player operations.
- `update` is the pure state transition.
- `MutableSignal<GameState>` is the runtime state holder.
- `page` projects the current state into pure HTML.
- The convenience-store floor, fixtures, products and customers are all DOM elements styled through `html.style`.

## Deliberate constraints

- no Canvas
- no game engine
- no screenshot/background image used as the play field
- no Playground-only utility CSS dependency

After this vertical slice builds cleanly, split model, store floor, HUD and customer feature into separate modules.
