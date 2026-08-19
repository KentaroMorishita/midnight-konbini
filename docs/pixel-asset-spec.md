# Midnight Konbini Pixel Asset Specification v1.1

この文書を `midnight-konbini` の実装用ピクセルアセット仕様の正本とする。

目的は、画像生成モデルに小さい実ピクセル寸法を無理に保証させることではなく、**統一したアートディレクションで生成した透過マスターを、固定ルールだけで実装可能な production sprite にすること**。

---

## 1. 基本方針

- 背景は完全透過 PNG。
- fixture は `1 asset = 1 PNG`。
- character は `1 character = 1 sprite sheet PNG`。
- portrait は character 本体とは別 PNG。
- v1 では atlas 化しない。
- 画像生成モデルの出力解像度は production size と一致しなくてよい。
- production 化では、構図の描き直し・生成し直し・手作業の切り貼りをしない。
- production 化で許可するのは、固定 grid 分割、alpha trim、aspect ratio を保った縮小、固定 anchor への配置、PNG の最適化だけ。
- floor / wall / aisle / selection highlight は DOM / CSS で描画する。

---

## 2. Grid と production raster

### Logical grid

- logical tile = **16 × 16**
- render scale = **3**
- production tile = **48 × 48 px**

Seseragi 側では production sprite を原則 1:1 で表示する。

```text
productionWidth  = tilesW * 48
productionHeight = tilesH * 48
```

`16 × 16` は world / placement の論理単位であり、production PNG の実解像度ではない。

---

## 3. Art direction

- 2D top-down 寄りの斜め俯瞰。
- fixture の正面と上面が少し見える固定 camera angle。
- 高品質 pixel-art look。
- dark outline。
- 左上からの店内照明。
- 大きな cast shadow は焼き込まない。
- fixture 本体は gray / navy / white を基調に、商品色を accent とする。
- character は全方向・全 frame で同一人物に見えること。
- 白背景、黒背景、床、壁を asset に含めない。

---

## 4. Fixture production size

fixture は静的 PNG。

| ID | 内容 | Tile footprint | Production px |
|---|---|---:|---:|
| `atm` | ATM | 1×2 | 48×96 |
| `auto_door` | 自動ドア | 3×2 | 144×96 |
| `register_counter` | レジカウンター | 4×2 | 192×96 |
| `shelf_small` | 小型棚 | 2×1 | 96×48 |
| `shelf_snack` | お菓子棚 | 3×1 | 144×48 |
| `shelf_magazine` | 雑誌・書籍棚 | 2×2 | 96×96 |
| `wall_fridge_drink` | 壁面冷蔵庫 | 4×1 | 192×48 |
| `shelf_bento` | 弁当・惣菜棚 | 3×1 | 144×48 |
| `freezer_case` | 冷凍食品ケース | 3×1 | 144×48 |
| `shelf_daily_goods` | 日用品棚 | 3×1 | 144×48 |
| `staff_room_door` | 従業員室ドア | 1×2 | 48×96 |
| `bulletin_board` | 掲示板 | 2×2 | 96×96 |
| `plant` | 観葉植物 | 1×2 | 48×96 |
| `trash_bin` | ゴミ箱 | 1×2 | 48×96 |
| `banner_onigiri` | のぼり | 1×3 | 48×144 |
| `terminal_sub` | 補助端末 | 1×2 | 48×96 |

### Fixture anchor

- production canvas からはみ出さない。
- 接地面は canvas 下端基準。
- 左右透明余白は原則均等。
- crop / background-position は不要。DOM `<img>` でそのまま置けること。

---

## 5. Character sprite sheet

店内 character は **4 directions × 3 frames**。

### Frame

- frame = **48 × 96 px**
- logical footprint = **1 × 2 tile**

### Sheet

- columns = 3
- rows = 4
- sheet = **144 × 384 px**

```text
row 0: down_idle   down_walk_1   down_walk_2
row 1: left_idle   left_walk_1   left_walk_2
row 2: right_idle  right_walk_1  right_walk_2
row 3: up_idle     up_walk_1     up_walk_2
```

### Anchor

- 足元 baseline は全 12 frame で固定。
- frame 内で水平中央に置く。
- idle / walk で全身位置が飛ばない。
- direction が変わっても髪型、服、体格、色を維持する。

### v1 lineup

- `clerk_male`
- `shopper_male_office`
- `shopper_female_adult`
- `shopper_male_student`

---

## 6. Portrait

- 1 character = 1 PNG
- production size = **48 × 48 px**
- 背景完全透過
- character sheet と髪型・服装・色を一致させる

---

## 7. Source master -> production sprite

画像生成は production 解像度より大きい透過 master を作る。

### Fixture

1. 透明背景 master を生成。
2. asset 全体を production canvas の aspect ratio に合わせる。
3. aspect ratio を維持したまま production size へ縮小。
4. alpha を維持して PNG 保存。

### Character

1. `3 columns × 4 rows` の透明 sprite master を生成。
2. master を固定 3×4 grid で分割。
3. 各 cell 内だけ alpha bbox を取得。
4. character identity を変更せず frame 内へ aspect-fit。
5. 水平中央、固定 foot baseline に配置。
6. `48×96 × 12` を `144×384` に再 pack。

禁止:

- AI による後処理で frame を描き直す
- frame ごとの手動スケール調整
- arbitrary crop
- 背景除去の推測処理
- asset ごとに異なる補正ルール

---

## 8. File layout

```text
public/
  sprites/
    fixtures/
      shelf_snack.png
      ...
    characters/
      clerk_male_sheet.png
      ...
    portraits/
      portrait_clerk_male.png
      ...

assets/
  manifest.json
```

すべて `snake_case`。

---

## 9. Manifest contract

```json
{
  "tileSize": 16,
  "renderScale": 3,
  "fixtures": {
    "shelf_snack": {
      "tilesW": 3,
      "tilesH": 1,
      "nativeWidth": 144,
      "nativeHeight": 48,
      "path": "/sprites/fixtures/shelf_snack.png"
    }
  },
  "characters": {
    "clerk_male": {
      "frameWidth": 48,
      "frameHeight": 96,
      "sheetWidth": 144,
      "sheetHeight": 384,
      "directions": ["down", "left", "right", "up"],
      "framesPerDirection": 3,
      "path": "/sprites/characters/clerk_male_sheet.png"
    }
  }
}
```

実装側は画像から寸法を推測しない。

---

## 10. Quality gate

- PNG transparency が維持されている。
- production dimensions が仕様と完全一致する。
- frame grid が整数 pixel で切れる。
- 白 / 黒 background がない。
- alpha halo が目立たない。
- perspective / light / scale が他 asset と一致する。
- 1:1 表示で読みやすい。
- character の全 12 frame で identity と foot anchor が一致する。

---

## 11. First accepted production assets

- `public/sprites/fixtures/shelf_snack.png`: **144 × 48**
- `public/sprites/characters/clerk_male_sheet.png`: **144 × 384** (`48 × 96` per frame)

この 2 asset を v1 の scale / rasterization reference とし、後続 asset は同じ pipeline で production 化する。
