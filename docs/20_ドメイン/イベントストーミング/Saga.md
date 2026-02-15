# イベントストーミング図: Saga（コンテキスト間連携）

> **更新日**: 2026-02-08
> **種別**: コンテキスト間連携

---

## 全体図

```mermaid
flowchart TB
    subgraph ctx_approval [申請承認コンテキスト]
        E1>休暇申請が<br/>承認された]:::event
        E2>打刻修正が<br/>承認された]:::event
        E2B>打刻修正が<br/>人事承認された]:::event
        E3>残業申請が<br/>承認された]:::event
    end

    subgraph ctx_monthly [月次処理コンテキスト]
        E4>月次が<br/>本締めされた]:::event
    end

    subgraph ctx_leave [休暇管理コンテキスト]
        AG_BAL{{有給残高}}:::aggregate
    end

    subgraph ctx_attendance [勤怠管理コンテキスト]
        AG_ATT{{勤怠記録}}:::aggregate
        P1[/36協定<br/>閾値チェック/]:::policy
    end

    subgraph ctx_notification [通知コンテキスト]
        AG_NTF{{通知}}:::aggregate
    end

    subgraph ctx_external [外部システム]
        EXT_CAL[/Googleカレンダー/]:::external
        EXT_PAY[/給与システム/]:::external
    end

    E1 -->|有給を消化する<br/>or 特別休暇を消化する| AG_BAL
    E1 -->|勤怠ステータスを<br/>休暇にする| AG_ATT
    E1 -->|休暇予定を同期| EXT_CAL
    E2 -->|打刻を修正する| AG_ATT
    E2B -->|打刻を修正する| AG_ATT
    E3 -->|残業を確定する| AG_ATT
    AG_ATT -->|トリガー| P1
    P1 -.->|閾値超過| AG_NTF
    E4 -->|全勤怠記録を<br/>確定する| AG_ATT
    E4 -->|CSV送信| EXT_PAY

    classDef event fill:#FFB570,stroke:#333
    classDef command fill:#90CAF9,stroke:#333
    classDef aggregate fill:#FFF59D,stroke:#333
    classDef policy fill:#B39DDB,stroke:#333
    classDef external fill:#E0E0E0,stroke:#333
```

---

## Saga別詳細図

### 休暇承認 Saga

休暇申請が承認されると、休暇残高の消化と勤怠記録への反映が連動する。
休暇種別により有給消化 or 特別休暇消化に分岐する。

```mermaid
sequenceDiagram
    participant LV as 休暇申請<br/>（申請承認）
    participant BAL as 有給残高<br/>（休暇管理）
    participant ATT as 勤怠記録<br/>（勤怠管理）
    participant CAL as Google<br/>カレンダー

    LV->>LV: 休暇申請が承認された
    alt 有給休暇（年次/半日/時間単位）
        LV->>BAL: 有給を消化する
        BAL->>BAL: 有給が消化された
    else 特別休暇（慶弔/リフレッシュ等）
        LV->>BAL: 特別休暇を消化する
        BAL->>BAL: 特別休暇が消化された
    end
    LV->>ATT: 勤怠ステータスを休暇にする
    ATT->>ATT: 勤怠ステータスが更新された
    LV->>CAL: カレンダーに休暇予定を同期
```

### 打刻修正承認 Saga

打刻修正が承認されると、勤怠記録の打刻データが修正される。
仮締め後の場合は人事承認を経てから勤怠記録に反映する。

```mermaid
sequenceDiagram
    participant FX as 打刻修正申請<br/>（申請承認）
    participant ATT as 勤怠記録<br/>（勤怠管理）

    alt 通常（仮締め前）
        FX->>FX: 打刻修正が承認された
        FX->>ATT: 打刻を修正する
    else 仮締め後（2段階承認）
        FX->>FX: 打刻修正が承認された（上長）
        Note over FX: 仮締め後人事承認<br/>エスカレーションポリシー発動
        FX->>FX: 打刻修正が人事承認された
        FX->>ATT: 打刻を修正する
    end
    ATT->>ATT: 打刻が修正された
    ATT->>ATT: 勤務時間が再計算された
```

### 残業承認 Saga

残業申請が承認されると、勤怠記録に残業が確定記録される。

```mermaid
sequenceDiagram
    participant OT as 残業申請<br/>（申請承認）
    participant ATT as 勤怠記録<br/>（勤怠管理）
    participant NTF as 通知<br/>（通知）

    OT->>OT: 残業申請が承認された
    OT->>ATT: 残業を確定する
    ATT->>ATT: 残業が確定された
    ATT->>ATT: 36協定閾値チェック
    alt 閾値超過
        ATT->>NTF: 通知を送信する（36協定アラート）
    end
```

### 月次本締め Saga

月次本締めにより、勤怠記録が確定し給与データがエクスポートされる。

```mermaid
sequenceDiagram
    participant CLS as 月次締め<br/>（月次処理）
    participant ATT as 勤怠記録<br/>（勤怠管理）
    participant EXT as 給与システム<br/>（外部）

    CLS->>CLS: 月次が本締めされた
    CLS->>ATT: 全勤怠記録を確定する
    ATT->>ATT: 勤怠記録が確定された
    CLS->>CLS: 給与データをエクスポートする
    CLS->>EXT: CSV送信
    CLS->>CLS: 給与データがエクスポートされた
```

---

## 凡例

```mermaid
flowchart LR
    E>イベント<br/>トリガー]:::event
    C[コマンド]:::command
    AG{{集約}}:::aggregate
    P[/ポリシー/]:::policy
    EXT[/外部システム/]:::external

    classDef event fill:#FFB570,stroke:#333
    classDef command fill:#90CAF9,stroke:#333
    classDef aggregate fill:#FFF59D,stroke:#333
    classDef policy fill:#B39DDB,stroke:#333
    classDef external fill:#E0E0E0,stroke:#333
```

---

## トリガーイベント → コマンド マッピング

| トリガーイベント | 発信元 | 発行コマンド | ターゲット |
|-----------------|--------|-------------|-----------|
| 休暇申請が承認された | 申請承認 | 有給を消化する / 特別休暇を消化する | 有給残高（休暇管理） |
| | | 勤怠ステータスを休暇にする | 勤怠記録（勤怠管理） |
| | | カレンダーに休暇予定を同期 | Googleカレンダー（外部） |
| 打刻修正が承認された | 申請承認 | 打刻を修正する | 勤怠記録（勤怠管理） |
| 打刻修正が人事承認された | 申請承認 | 打刻を修正する | 勤怠記録（勤怠管理） |
| 残業申請が承認された | 申請承認 | 残業を確定する | 勤怠記録（勤怠管理） |
| 月次が本締めされた | 月次処理 | 全勤怠記録を確定する | 勤怠記録（勤怠管理） |
| | | 給与データをエクスポートする | 給与システム（外部） |
| 36協定閾値超過検知 | 勤怠管理 | 通知を送信する | 通知（通知） |

---

## Sagaサマリー

| Saga名 | トリガーイベント | 関連コンテキスト | コマンド数 | 補足 |
|--------|-----------------|----------------|-----------|------|
| 休暇承認 | 休暇申請が承認された | 申請承認 → 休暇管理, 勤怠管理, 外部 | 3 | 有給 or 特別休暇消化 + 勤怠反映 + カレンダー同期 |
| 打刻修正承認 | 打刻修正が承認された / 人事承認された | 申請承認 → 勤怠管理 | 1 | 仮締め後は2段階承認（上長→人事）を経由 |
| 残業承認 | 残業申請が承認された | 申請承認 → 勤怠管理, 通知 | 1 | 残業確定 + 36協定チェック（条件分岐→通知） |
| 月次本締め | 月次が本締めされた | 月次処理 → 勤怠管理, 外部 | 2 | 全勤怠確定 + 給与CSVエクスポート |

<!-- 品質チェック結果
- [x] 主要4 Sagaパターンが図示されている
- [x] コンテキスト境界をまたぐ連携が明確
- [x] 外部システム（Google カレンダー、給与システム）が含まれている
- [x] 通知コンテキストが正式なコンテキストとして表現されている
- [x] 条件分岐（36協定閾値超過時のアラート）が表現されている
- [x] 休暇承認Sagaに有給/特別休暇の分岐がある
- [x] 打刻修正承認Sagaに仮締め後2段階承認の分岐がある
- [x] classDef色定義がES図と統一されている
-->
