# DB設計

## テーブル一覧

### `users`（ユーザー）

| カラム | 型 | 制約 | 説明 |
|---|---|---|---|
| `id` | `UUID` | PK | |
| `email` | `VARCHAR` | NOT NULL, UNIQUE | |
| `created_at` | `TIMESTAMPTZ` | NOT NULL, DEFAULT now() | |

---

### `places`（登録スポット）

| カラム | 型 | 制約 | 説明 |
|---|---|---|---|
| `id` | `UUID` | PK | |
| `user_id` | `UUID` | NOT NULL, FK → users.id | 登録したユーザー |
| `name` | `TEXT` | NOT NULL | スポット名 |
| `stay_duration` | `SMALLINT` | NOT NULL, CHECK (1, 2, 3) | 滞在時間カテゴリ |
| `latitude` | `FLOAT8` | NOT NULL | 緯度 |
| `longitude` | `FLOAT8` | NOT NULL | 経度 |
| `created_at` | `TIMESTAMPTZ` | NOT NULL, DEFAULT now() | |

**stay_duration の定義（アプリ側でマッピング）:**

| 値 | カテゴリ | 目安 | フィルタ用最小時間 |
|---|---|---|---|
| `1` | サクッと | 〜30分 | 30分 |
| `2` | ゆっくり | 1〜2時間 | 60分 |
| `3` | たっぷり | 3時間〜 | 180分 |

---

### `visit_logs`（訪問記録）

| カラム | 型 | 制約 | 説明 |
|---|---|---|---|
| `id` | `UUID` | PK | |
| `place_id` | `UUID` | NOT NULL, FK → places.id | 訪問した場所 |
| `visited_at` | `TIMESTAMPTZ` | NOT NULL, DEFAULT now() | 訪問日時 |

※ 訪問ユーザーは `place_id → places.user_id` で導出可能なため保持しない。

---

## ER図

```
users
  │
  ↓ FK (user_id)
places
  id (PK)
  user_id (FK) ───→ users
  name
  stay_duration (1/2/3)
  latitude
  longitude
  created_at
  │
  ↓ FK (place_id)
visit_logs
  id (PK)
  place_id (FK) ──→ places
  visited_at
```

---

## 検索ロジック（アプリ側）

```
自由時間（分） ≧ 往復移動時間（分） + stay_duration に対応する最小時間（分）
```

1. ユーザーが「自由時間」を入力
2. ブラウザの Geolocation API で現在地（緯度経度）を取得
3. `places` から自分のスポットを全件取得
4. 地図API で各スポットへの片道移動時間を取得
5. 条件を満たすスポットをフィルタリング
6. 候補からランダムに1件を選出（候補0件の場合は最も条件に近い1件を表示）

---

## 外部API

| 用途 | タイミング |
|---|---|
| ジオコーディング（スポット名・住所 → 緯度経度） | 場所登録時 |
| 移動時間計算（現在地 → スポットの片道時間） | 検索時 |
