# スクリーンマップ: シフト

生成日時: 2026-02-09

---

## 1. 画面一覧

| # | 画面名 | 画面種別 | 優先度 | ステータス | 関連集約 |
|---|--------|---------|--------|-----------|---------:|
| 1 | シフトパターン一覧 | list | required | designed | シフト |
| 2 | シフトパターン登録 | modal | required | designed | シフト |
| 3 | シフトカレンダー | list | required | designed | シフト |
| 4 | シフト割当 | modal | required | designed | シフト |

---

## 2. API→画面マッピング

### シフトパターン一覧

| API | メソッド | 画面での用途 |
|-----|---------|-------------|
| `/shifts/patterns` | GET | 一覧表示（検索・ソート・ページネーション） |
| `/shifts/patterns/{patternId}` | GET | 行クリック時の詳細表示（一覧に統合） |
| `/shifts/patterns/{patternId}/actions/deactivate` | POST | 無効化ボタン |
| `/shifts/patterns/{patternId}/actions/reactivate` | POST | 再有効化ボタン |

### シフトパターン登録

| API | メソッド | 画面での用途 |
|-----|---------|-------------|
| `/shifts/patterns` | POST | 新規パターン登録フォーム送信 |

### シフトカレンダー

| API | メソッド | 画面での用途 |
|-----|---------|-------------|
| `/shifts/schedules` | GET | カレンダー形式の一覧表示 |
| `/shifts/schedules/{scheduleId}` | GET | スケジュール詳細表示（一覧に統合） |
| `/shifts/schedules/{scheduleId}/actions/publish` | POST | 公開ボタン |

### シフト割当

| API | メソッド | 画面での用途 |
|-----|---------|-------------|
| `/shifts/schedules` | POST | 新規シフト割当フォーム送信 |
| `/shifts/schedules/{scheduleId}` | PUT | シフト変更フォーム送信 |

---

## 3. カバレッジレポート

| 指標 | 値 |
|------|-----|
| 全エンドポイント数 | 10 |
| 画面割当済み | 10 |
| 未割当 | 0 |
| カバレッジ率 | 100% |

### 未割当エンドポイント（0件）

（なし）

---

## 4. ロール別画面アクセスマトリクス

| 画面名 | 従業員本人 | 管理職 | 人事担当者 |
|--------|:---------:|:-----:|:---------:|
| シフトパターン一覧 | - | R | CRUD |
| シフトパターン登録 | - | - | C |
| シフトカレンダー | R | CR | - |
| シフト割当 | - | CU | - |

---

## 5. 既存画面との差分

### 不足画面

| # | 画面名 | 画面種別 | 導出元API |
|---|--------|---------|----------|
| 1 | シフトパターン一覧 | list | GET /patterns, GET /patterns/{id}, POST /actions/deactivate, POST /actions/reactivate |
| 2 | シフトパターン登録 | modal | POST /patterns |
| 3 | シフトカレンダー | list | GET /schedules, GET /schedules/{id}, POST /actions/publish |
| 4 | シフト割当 | modal | POST /schedules, PUT /schedules/{id} |

### 更新推奨画面

（なし -- 全画面が新規作成）

---

## 6. 推奨実装順序

| 順序 | 画面名 | 理由 |
|------|--------|------|
| 1 | シフトパターン一覧 | マスタデータ管理。パターンが存在しないとスケジュール割当ができない |
| 2 | シフトパターン登録 | パターン一覧と対で使用。一覧の「+ 新規登録」ボタンから起動 |
| 3 | シフトカレンダー | スケジュール管理のメイン画面。パターン定義後に利用開始 |
| 4 | シフト割当 | カレンダーの「+ シフト割当」ボタンから起動。カレンダーと対で使用 |
