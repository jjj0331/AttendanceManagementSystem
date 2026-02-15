# 月次締め 画面マップ

## 概要

月次締め集約のAPI設計書から導出された画面一覧。

**集約:** 月次締め
**コンテキスト:** 月次処理
**ベースパス:** `/api/v1/monthly-closings`

---

## エンドポイント分析

| メソッド | パス | pathPattern | hasIdParam | fieldCount | 説明 |
|---------|------|-------------|-----------|------------|------|
| POST | `/actions/preliminary-close` | action | No | 2 | 仮締めする |
| POST | `/actions/finalize` | action | No | 2 | 本締めする |
| POST | `/actions/export-payroll` | action | No | 2 | 給与データをエクスポートする |
| GET | `/dashboard` | dashboard | No | 0 | 月次締め状況ダッシュボード |
| GET | `/{monthlyClosingId}` | detail | Yes | 0 | 月次締め詳細取得 |
| GET | `/{monthlyClosingId}/export-status` | detail(sub) | Yes | 0 | エクスポートステータス確認 |

---

## 導出された画面

| # | 画面名 | 画面種別 | 優先度 | 由来 | ステータス |
|---|--------|---------|--------|------|-----------|
| 1 | 月次締めダッシュボード | dashboard | required | GET /dashboard | missing |
| 2 | 月次締め詳細 | detail | required | GET /{monthlyClosingId} | missing |

---

## 画面別操作マッピング

### 1. 月次締めダッシュボード

| 操作 | エンドポイント | 表示形式 |
|------|--------------|---------|
| ダッシュボード表示 | GET /dashboard | KPIカード + テーブル |
| 仮締め実行 | POST /actions/preliminary-close | 行アクションボタン |
| 本締め実行 | POST /actions/finalize | 行アクションボタン |

### 2. 月次締め詳細

| 操作 | エンドポイント | 表示形式 |
|------|--------------|---------|
| 詳細表示 | GET /{monthlyClosingId} | 詳細画面 |
| 仮締め実行 | POST /actions/preliminary-close | アクションボタン |
| 本締め実行 | POST /actions/finalize | アクションボタン |
| エクスポート実行 | POST /actions/export-payroll | アクションボタン |
| エクスポートステータス | GET /{monthlyClosingId}/export-status | ステータス表示セクション |

---

## カバレッジ

| 指標 | 値 |
|------|-----|
| 総エンドポイント数 | 6 |
| カバー済み | 6 |
| 未カバー | 0 |
| カバー率 | 100% |
