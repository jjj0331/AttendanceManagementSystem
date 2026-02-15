# スクリーンマップ: 勤怠記録

生成日時: 2026-02-08

---

## 1. 画面一覧

| # | 画面名 | 画面種別 | 優先度 | ステータス | 関連集約 |
|---|--------|---------|--------|-----------|---------|
| 1 | 勤怠打刻 | modal | required | designed | 勤怠記録 |
| 2 | 日次勤怠一覧 | list | required | designed | 勤怠記録 |
| 3 | 月次勤怠サマリー | dashboard | required | designed | 勤怠記録 |
| 4 | 部門別勤怠ダッシュボード | dashboard | required | designed | 勤怠記録 |

---

## 2. API→画面マッピング

### 勤怠打刻

| API | メソッド | 画面での用途 |
|-----|---------|-------------|
| `/attendances/today` | GET | 初期ロード時の当日ステータス取得 |
| `/attendances/clock-in` | POST | 出勤打刻ボタン |
| `/attendances/clock-out` | POST | 退勤打刻ボタン |
| `/attendances/break-start` | POST | 休憩開始ボタン |
| `/attendances/break-end` | POST | 休憩終了ボタン |

### 日次勤怠一覧

| API | メソッド | 画面での用途 |
|-----|---------|-------------|
| `/attendances/daily` | GET | 一覧表示（検索・ソート・ページネーション） |

### 月次勤怠サマリー

| API | メソッド | 画面での用途 |
|-----|---------|-------------|
| `/attendances/monthly-summary` | GET | KPIカード・月次集計テーブル表示 |
| `/attendances/monthly-summary/export` | GET | CSV出力 |

### 部門別勤怠ダッシュボード

| API | メソッド | 画面での用途 |
|-----|---------|-------------|
| `/attendances/department-dashboard` | GET | 部門別KPI・アラート件数・出勤率表示 |
| `/attendances/department-dashboard/export` | GET | CSV出力 |

---

## 3. カバレッジレポート

| 指標 | 値 |
|------|-----|
| 全エンドポイント数 | 13 |
| 内部API（画面不要） | 3 |
| 画面対象エンドポイント | 10 |
| 画面割当済み | 10 |
| 未割当 | 0 |
| カバレッジ率 | 100% |

### 未割当エンドポイント（0件）

（なし）

### スキップされた内部API

| API | メソッド | 理由 |
|-----|---------|------|
| `/internal/attendances/correct` | POST | 打刻修正申請集約から内部呼び出し |
| `/internal/attendances/register` | POST | 手動登録申請集約から内部呼び出し |
| `/internal/attendances/finalize` | POST | 月次締めSagaから内部呼び出し |

---

## 4. ロール別画面アクセスマトリクス

| 画面名 | 従業員本人 | 上長 | 人事 | 管理職 |
|--------|:---------:|:----:|:----:|:-----:|
| 勤怠打刻 | ✅ | - | - | - |
| 日次勤怠一覧 | ✅ | ✅ | ✅ | - |
| 月次勤怠サマリー | - | ✅ | ✅ | - |
| 部門別勤怠ダッシュボード | - | - | ✅ | ✅ |

---

## 5. 既存画面との差分

### 不足画面

| # | 画面名 | 画面種別 | 導出元API |
|---|--------|---------|----------|
| 1 | 勤怠打刻 | modal | POST /clock-in, /clock-out, /break-start, /break-end |
| 2 | 日次勤怠一覧 | list | GET /daily |
| 3 | 月次勤怠サマリー | dashboard | GET /monthly-summary |
| 4 | 部門別勤怠ダッシュボード | dashboard | GET /department-dashboard |

### 更新推奨画面

（なし — 全画面が新規作成）

---

## 6. 推奨実装順序

| 順序 | 画面名 | 理由 |
|------|--------|------|
| 1 | 勤怠打刻 | コア操作。従業員が最も頻繁に使う画面 |
| 2 | 日次勤怠一覧 | 打刻結果の確認に必須。打刻画面と対で使用 |
| 3 | 月次勤怠サマリー | 上長の月次管理に必要 |
| 4 | 部門別勤怠ダッシュボード | 管理職・人事向け。他画面完成後でも可 |
