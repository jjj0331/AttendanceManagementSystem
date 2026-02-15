# Phase 3.5 → Phase 4 申し送り

## 完了サマリー

5コンテキスト間のコンテキスト間連携設計を完了。4つのSaga連携（休暇承認・打刻修正承認・残業承認・月次本締め）、3つのクロスコンテキストクエリ、13種の通知イベントを定義。連携パターンは同期サービス呼び出し + ドメインイベント + Sagaオーケストレーションの3種を使い分け。

## 次のPhaseへの入力

### 連携パターン一覧（決定事項）

| Saga名 | パターン | 技術 |
|--------|---------|------|
| 休暇承認 | 同期 + イベント | @Service DI + ApplicationEventPublisher |
| 打刻修正承認 | 同期 | @Service DI |
| 残業承認 | 同期 + イベント | @Service DI + ApplicationEventPublisher |
| 月次本締め | Saga | ApplicationEventPublisher + Sagaオーケストレータ |

### 設計判断と理由

| 判断 | 理由 |
|------|------|
| クロスコンテキスト直接JOIN（VIEW不要） | モジュラーモノリスで同一DB。VIEWの管理コストを避ける。MS分割時にAPI呼び出しに切り替え |
| 依存性逆転（ポート&アダプタ） | 呼び出し元にインターフェース定義、呼び出し先に実装配置。コンテキスト境界を保つ |
| 通知はアプリ内のみ（MVP） | メール・Slack連携はv2で追加。通知イベントペイロードは拡張可能な設計 |
| JWTペイロード: sub, email, employeeId, role, departmentId, name | 勤怠システムに必要な最小クレーム。roleベースのアクセス制御 |
| 年度4月起算 / Asia/Tokyo固定 | 日本国内専用。海外拠点対応はv2以降 |

### インターフェース定義済みポート

| ポート | 方向 | 用途 |
|--------|------|------|
| `LeaveBalancePort` | 申請承認 → 休暇管理 | 有給・特別休暇の消化 |
| `AttendanceRecordPort` | 申請承認 → 勤怠管理 | 勤怠ステータスの休暇更新 |
| `ClockCorrectionPort` | 申請承認 → 勤怠管理 | 打刻修正 + 勤務時間再計算 |
| `OvertimeConfirmationPort` | 申請承認 → 勤怠管理 | 残業確定 + 36協定チェック |
| `AttendanceFinalizePort` | 月次処理 → 勤怠管理 | 全勤怠記録の確定 |
| `PayrollExportPort` | 月次処理 → 外部 | 給与CSVエクスポート |
| `MonthlyClosingQueryPort` | 申請承認 → 月次処理 | 仮締め状態確認 |
| `LeaveBalanceQueryPort` | 申請承認 → 休暇管理 | 有給残高照会 |
| `OvertimeQueryPort` | 申請承認 → 勤怠管理 | 累計残業時間照会 |

## 未確定事項

- [ ] Googleカレンダー連携の具体的API仕様（実装フェーズで確定）
- [ ] 給与システムCSVフォーマット（Phase 0 から継続、汎用フォーマットで先行）
- [ ] メール・Slack通知の追加（v2）

## 注意点

- 同期サービス呼び出しは同一トランザクション内で実行されるため、呼び出し先の例外が呼び出し元にロールバックを波及させる。例外ハンドリングを明確にすること
- 月次本締めSagaの補償トランザクションは「手動再実行」方式。自動補償（ロールバック）ではない点に注意
- 通知イベント（NotificationRequestedEvent）は `@TransactionalEventListener(AFTER_COMMIT)` で処理。トランザクション失敗時は通知も送信されない
- 共有テーブル（employees）は人事システムからの同期データ。同期タイミング・差分検知の仕組みは実装フェーズで設計
