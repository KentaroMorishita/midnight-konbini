# Midnight Konbini / 深夜コンビニ

Seseragiで作る2Dコンビニ経営ゲーム。

Canvasやゲーム用フレームワークに逃げず、店舗・棚・顧客・HUD・状態管理・操作をSeseragiのWeb UI surfaceで組み立てます。

## Run

```sh
seseragi dev --open
```

Production build:

```sh
seseragi build .
```

## Deployment

`main` へのpushでGitHub ActionsがSeseragiのproduction buildを作成し、`VERCEL_TOKEN`が設定されていればVercelへproduction deployします。

## Current slice

- 2D top-down convenience-store floor built from DOM elements
- sales / satisfaction / inventory / cleanliness / energy HUD
- selectable customers and customer detail panel
- restock / clean / checkout-priority / recommend / suspicious actions
- manual game clock controls and event log
- Signal-backed game state
- no Canvas
- no background screenshot masquerading as the game board

The first milestone is intentionally a single playable vertical slice. Once the compiler/build feedback is clean, the model and UI will be split into feature modules.
