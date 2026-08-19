# Midnight Konbini Pixel Asset Specification v1

この文書を、`midnight-konbini` における **実装用ピクセルアセットの正本** とする。

目的は「見栄えの良い画像を作ること」ではなく、**Seseragi の DOM 実装にそのまま載せられる、寸法・視点・透過・命名・アニメーション構造が確定したアセットを生成すること**。

仕様に合わない生成物は、見た目が良くても採用しない。

---

## 1. 基本方針

- 背景は完全透過 PNG。
- fixture は **1 asset = 1 file**。
- character は **1 character = 1 sprite sheet**。
- portrait は character 本体とは別 file。
- v1 では atlas 化しない。
- 生成後に Python 等でサイズ・余白・背景・座標を無理に補正する運用はしない。
- 実装側が画像に合わせるのではなく、**画像がこの仕様に従う**。
- 床・壁・通路などのベース環境は DOM / CSS で表現し、fixture sprite に焼き込まない。

---

## 2. グリッド / 表示倍率

### 基準グリッド

- 1 tile = **16 × 16 px**
- render scale = **3x**
- 実装上の 1 tile = **48 × 48 CSS px**

### 実装計算

```text
renderWidth  = tilesW * 16 * 3
renderHeight = tilesH * 16 * 3
```

画像自体は下記仕様の native pixel size で生成する。CSS では nearest-neighbor 系の拡大を前提とする。

---

## 3. 共通アートディレクション

### 視点

- 2D top-down 寄りの俯瞰。
- 完全な真上視点ではなく、fixture の正面・上面が少し見える角度。
- 全 asset でカメラ角度を統一する。

### 画風

- 高品質ピクセルアート。
- アニメ寄りにしすぎず、ゲーム用 sprite として輪郭とシルエットが読みやすいこと。
- 商品は細かく描くが、3x 表示時にノイズにならない密度に抑える。
- 外周は濃い輪郭線。
- 内部ディテールは 1px〜2px 相当のまとまりを意識する。

### 光源

- 左上からの店内照明。
- 影は弱く短い。
- asset ごとに影方向を変えない。
- 大きな落ち影を画像内へ焼き込まない。

### 色

- 店内 UI の dark navy とぶつからない、やや明るめの店舗色。
- fixture 本体は neutral gray / navy / white を基調にし、商品色をアクセントにする。
- キャラクターは背景床上でシルエットが読める明度差を持たせる。

---

## 4. 背景 / 透過仕様

- canvas 全体の背景は **完全透過**。
- 白・黒・単色背景の焼き込みは禁止。
- 半透明の背景 halo 禁止。
- 影を入れる場合も asset 本体周辺だけに限定し、外周へ不要な alpha ノイズを作らない。
- fixture と character の周囲に白フチを作らない。

---

## 5. Fixture 仕様

fixture は静的画像。v1 では animation を持たない。

| ID | 内容 | Tile | Native px |
|---|---|---:|---:|
| `atm` | ATM | 1×2 | 16×32 |
| `auto_door` | 自動ドア | 3×2 | 48×32 |
| `register_counter` | レジカウンター | 4×2 | 64×32 |
| `shelf_small` | 小型棚 | 2×1 | 32×16 |
| `shelf_snack` | お菓子棚 | 3×1 | 48×16 |
| `shelf_magazine` | 雑誌・書籍棚 | 2×2 | 32×32 |
| `wall_fridge_drink` | 壁面冷蔵庫・ドリンク | 4×1 | 64×16 |
| `shelf_bento` | 弁当・惣菜棚 | 3×1 | 48×16 |
| `freezer_case` | 冷凍食品ケース | 3×1 | 48×16 |
| `shelf_daily_goods` | 日用品棚 | 3×1 | 48×16 |
| `staff_room_door` | 従業員室ドア | 1×2 | 16×32 |
| `bulletin_board` | 掲示板 | 2×2 | 32×32 |
| `plant` | 観葉植物 | 1×2 | 16×32 |
| `trash_bin` | ゴミ箱 | 1×2 | 16×32 |
| `banner_onigiri` | おにぎりのぼり | 1×3 | 16×48 |
| `terminal_sub` | 補助レジ端末 | 1×2 | 16×32 |

### Fixture layout rule

- 指定 native px canvas からはみ出さない。
- 接地物は canvas 下端を基準に置く。
- 左右の透明余白は原則均等。
- 上部看板を含む場合も指定サイズ内に収める。
- fixture の床・壁面背景は含めない。

---

## 6. Character sprite sheet 仕様

店内で動く人物は固定立ち絵ではなく、**4方向 × 3フレーム** の sprite sheet とする。

### 1 frame

- frame size = **16 × 32 px**
- tile footprint = **1 × 2**
- render size = **48 × 96 CSS px**

### Direction

方向順は固定。

```text
down
left
right
up
```

### Frames per direction

各方向 3 frame。

```text
idle
walk_1
walk_2
```

### Sheet layout

```text
row 0: down_idle   down_walk_1   down_walk_2
row 1: left_idle   left_walk_1   left_walk_2
row 2: right_idle  right_walk_1  right_walk_2
row 3: up_idle     up_walk_1     up_walk_2
```

sheet size は **48 × 128 px**。

### Character anchor rule

- 各 frame の足元位置を揃える。
- 各方向で頭頂位置・体格が不自然に変わらない。
- idle / walk 間でキャラ全体が左右へジャンプしない。
- walk は足と腕の差分を主とし、顔や髪型を別人物のように変えない。
- down / left / right / up で服装・髪型・バッグ等の特徴を保持する。
- 全 frame で canvas 外へのはみ出し禁止。

---

## 7. v1 Character lineup

最初に作る playable / NPC 用キャラは以下。

| ID | 用途 |
|---|---|
| `clerk_male` | 男性店員 |
| `shopper_male_office` | 男性会社員客 |
| `shopper_female_adult` | 成人女性客 |
| `shopper_male_student` | 男子大学生客 |

追加候補は v1 最小セット完成後に扱う。

---

## 8. Portrait 仕様

右側の「選択中の顧客」パネル等に使う UI portrait は character sheet と別 asset。

- 1 character = 1 file
- native size = **16 × 16 px**
- render size = **48 × 48 CSS px**
- 背景完全透過
- 正面顔中心
- character sheet と髪型・服装・色を一致させる

命名例:

```text
portrait_clerk_male.png
portrait_shopper_male_office.png
```

---

## 9. ファイル命名 / 配置

すべて `snake_case`。

```text
public/
  sprites/
    fixtures/
      atm.png
      auto_door.png
      register_counter.png
      shelf_small.png
      shelf_snack.png
      shelf_magazine.png
      wall_fridge_drink.png
      freezer_case.png
      ...

    characters/
      clerk_male_sheet.png
      shopper_male_office_sheet.png
      shopper_female_adult_sheet.png
      shopper_male_student_sheet.png

    portraits/
      portrait_clerk_male.png
      portrait_shopper_male_office.png
      portrait_shopper_female_adult.png
      portrait_shopper_male_student.png
```

---

## 10. Manifest contract

実装側は、画像そのものから寸法を推測しない。

将来的な asset metadata は以下の contract を基準とする。

```json
{
  "tileSize": 16,
  "renderScale": 3,
  "fixtures": {
    "register_counter": {
      "tilesW": 4,
      "tilesH": 2,
      "path": "/sprites/fixtures/register_counter.png"
    }
  },
  "characters": {
    "clerk_male": {
      "frameWidth": 16,
      "frameHeight": 32,
      "directions": ["down", "left", "right", "up"],
      "framesPerDirection": 3,
      "path": "/sprites/characters/clerk_male_sheet.png"
    }
  },
  "portraits": {
    "portrait_clerk_male": {
      "width": 16,
      "height": 16,
      "path": "/sprites/portraits/portrait_clerk_male.png"
    }
  }
}
```

---

## 11. v1 最小アセットセット

### Fixtures

```text
atm
auto_door
register_counter
shelf_small
shelf_snack
shelf_magazine
wall_fridge_drink
freezer_case
```

### Characters

```text
clerk_male
shopper_male_office
shopper_female_adult
shopper_male_student
```

### Portraits

上記 4 character 分。

---

## 12. 生成時の固定ルール

画像生成時は、毎回以下を明示する。

- asset ID
- native canvas size
- transparent background
- exact camera angle
- same pixel density
- same outline thickness
- same lighting direction
- same world scale
- no floor / no wall background
- no text unless the fixture itself requires signage
- no extra props outside the requested asset

character sheet の場合は追加で以下を明示する。

- 48×128 px sheet
- 16×32 px frame
- 3 columns × 4 rows
- row order: down / left / right / up
- column order: idle / walk_1 / walk_2
- identical character identity across all 12 frames
- consistent foot anchor across all frames

---

## 13. 品質ゲート

以下を 1 つでも満たさない asset は本編へ入れない。

### Technical

- 完全透過 PNG
- native size が仕様どおり
- 白背景 / 黒背景なし
- 余計な alpha halo なし
- canvas 外にはみ出さない
- 3x nearest-neighbor 表示で輪郭が崩れない

### Visual

- 他 asset と視点が一致
- 他 asset と scale が一致
- 光源方向が一致
- ピクセル密度が一致
- fixture の設置面が一致
- character の足元 anchor が一致
- character の方向違いで別人化しない

### Implementation

- asset 単体で DOM `<img>` として配置可能
- fixture は crop / background-position を必要としない
- character だけ frame crop で扱う
- runtime で Python 加工や画像再生成を必要としない

---

## 14. 実装ルール

- fixture は `<img>` を絶対配置。
- character は sprite sheet を 16×32 frame window で表示。
- `direction` と `frameIndex` から crop 位置を決定する。
- 床・壁・通路・選択ハイライトは DOM / CSS。
- character animation は将来の Clock / Fiber / game loop 実装後に `frameIndex` を更新する。
- v1 の asset correctness を確認するまで atlas 化しない。

---

## 15. Source of truth

この文書の数値・命名・row/column order を正とする。

生成プロンプト、manifest、Seseragi 実装のいずれかと矛盾した場合は、**この文書を先に更新してから他を追従させる**。

仕様変更時は document title の version を更新する。