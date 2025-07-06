# 業務フロー分析

## ジョブステップシーケンスと判断ポイント特定

### 1. 業務フロー分析の概要

#### 1.1 分析方針
- **ジョブステップシーケンス**: 各ジョブの実行順序と依存関係を分析
- **判断ポイント特定**: 業務の分岐点・決定点を明確化
- **UCとジョブステップの関連付け**: どのステップでどのUCが利用されるかを詳細化
- **例外フロー**: 正常フローと例外フローの両方を考慮

#### 1.2 フロー表記法
- **順次**: ステップの連続実行
- **分岐**: 条件による処理の分岐
- **並行**: 複数処理の同時実行
- **例外**: エラー・例外発生時の処理

### 2. 主要業務フロー図

#### 2.1 図書貸出の統合フロー

```mermaid
flowchart TD
    Start([利用者来館]) --> Select[図書選択]
    Select --> Counter[カウンター来訪]
    Counter --> MemberCheck{会員有効性確認}
    
    MemberCheck -->|無効| MemberError[会員エラー]
    MemberCheck -->|有効| ItemCheck{所蔵品状態確認}
    
    ItemCheck -->|貸出不可| ItemError[所蔵品エラー]
    ItemCheck -->|貸出可能| LoanCheck{貸出制限確認}
    
    LoanCheck -->|制限超過| LimitError[制限エラー]
    LoanCheck -->|制限内| DelayCheck{延滞状況確認}
    
    DelayCheck -->|延滞制限| DelayError[延滞エラー]
    DelayCheck -->|制限なし| LoanProcess[貸出登録処理]
    
    LoanProcess --> LoanComplete[貸出完了]
    LoanComplete --> HandOver[図書引渡し]
    HandOver --> End([利用者退館])
    
    MemberError --> ErrorEnd[処理終了・説明]
    ItemError --> ErrorEnd
    LimitError --> ErrorEnd
    DelayError --> ErrorEnd
    ErrorEnd --> End
```

#### 2.2 図書予約の統合フロー

```mermaid
flowchart TD
    Start([予約開始]) --> Search[図書検索]
    Search --> Found{図書発見?}
    
    Found -->|未発見| SearchAgain[再検索]
    SearchAgain --> Search
    Found -->|発見| Select[図書選択]
    
    Select --> MemberCheck{会員有効性確認}
    MemberCheck -->|無効| MemberError[会員エラー]
    MemberCheck -->|有効| ReserveCheck{予約制限確認}
    
    ReserveCheck -->|制限超過| LimitError[予約制限エラー]
    ReserveCheck -->|制限内| AvailableCheck{図書利用可能性}
    
    AvailableCheck -->|在庫あり| InstantLoan[即時貸出可能]
    AvailableCheck -->|貸出中| ReserveProcess[予約登録処理]
    
    InstantLoan --> LoanFlow[貸出フローへ]
    ReserveProcess --> ReserveComplete[予約完了]
    ReserveComplete --> WaitNotice[取置通知待ち]
    
    WaitNotice --> RetentionFlow[取置フローへ]
    
    MemberError --> ErrorEnd[処理終了・説明]
    LimitError --> ErrorEnd
    ErrorEnd --> End([予約終了])
    
    ReserveComplete --> End
```

#### 2.3 予約取置の統合フロー

```mermaid
flowchart TD
    Start([取置業務開始]) --> CheckUnprepared[未準備予約確認]
    CheckUnprepared --> HasReservation{未準備予約あり?}
    
    HasReservation -->|なし| NoWork[作業なし]
    HasReservation -->|あり| SelectReservation[予約選択]
    
    SelectReservation --> FindItem[該当図書探索]
    FindItem --> ItemFound{図書発見?}
    
    ItemFound -->|未発見| ItemNotFound[図書未発見エラー]
    ItemFound -->|発見| StatusCheck{図書状態確認}
    
    StatusCheck -->|在庫中以外| StatusError[状態エラー]
    StatusCheck -->|在庫中| MatchCheck{予約図書一致?}
    
    MatchCheck -->|不一致| MatchError[一致エラー]
    MatchCheck -->|一致| RetentionRegister[取置登録]
    
    RetentionRegister --> NotifyUser[利用者通知]
    NotifyUser --> RetentionManage[取置管理]
    
    RetentionManage --> UserComes{利用者来館?}
    UserComes -->|来館| LoanProcess[貸出処理]
    UserComes -->|期限切れ| ExpireProcess[期限切れ処理]
    
    LoanProcess --> LoanComplete[貸出完了]
    ExpireProcess --> ReserveCancel[予約キャンセル]
    
    NoWork --> End([取置業務終了])
    ItemNotFound --> ProcessNext[次の予約処理]
    StatusError --> ProcessNext
    MatchError --> ProcessNext
    LoanComplete --> ProcessNext
    ReserveCancel --> ProcessNext
    ProcessNext --> CheckUnprepared
```

### 3. ジョブステップ詳細分析

#### 3.1 図書館利用者のジョブフロー

##### ジョブ: 図書を借りる
| ステップ順序 | ステップ名 | 判断ポイント | 利用UC | 次ステップ | 例外処理 |
|-------------|------------|-------------|--------|------------|----------|
| 1 | 来館 | - | - | 図書選択 | - |
| 2 | 図書選択 | 図書発見? | - | カウンター来訪 | 再探索 |
| 3 | カウンター来訪 | - | - | 会員確認 | - |
| 4 | 会員確認 | 会員有効? | 会員番号有効性確認 | 所蔵品確認 | エラー終了 |
| 5 | 所蔵品確認 | 貸出可能? | 所蔵品貸出可否判定 | 制限確認 | エラー終了 |
| 6 | 制限確認 | 制限内? | 貸出制限判定 | 貸出処理 | エラー終了 |
| 7 | 貸出処理 | - | 貸出登録 | 図書引渡し | - |
| 8 | 図書引渡し | - | - | 完了 | - |

##### ジョブ: 図書を予約する
| ステップ順序 | ステップ名 | 判断ポイント | 利用UC | 次ステップ | 例外処理 |
|-------------|------------|-------------|--------|------------|----------|
| 1 | 図書検索 | 図書発見? | 図書検索 | 図書選択 | 再検索 |
| 2 | 図書選択 | - | 図書詳細確認 | 会員確認 | - |
| 3 | 会員確認 | 会員有効? | 会員番号有効性確認 | 予約制限確認 | エラー終了 |
| 4 | 予約制限確認 | 制限内? | 予約制限判定 | 利用可能性確認 | エラー終了 |
| 5 | 利用可能性確認 | 在庫あり? | - | 即時貸出/予約処理 | - |
| 6a | 即時貸出 | - | 貸出登録 | 完了 | - |
| 6b | 予約処理 | - | 予約登録 | 完了 | - |

#### 3.2 司書のジョブフロー

##### ジョブ: 取置業務を管理する
| ステップ順序 | ステップ名 | 判断ポイント | 利用UC | 次ステップ | 例外処理 |
|-------------|------------|-------------|--------|------------|----------|
| 1 | 未準備予約確認 | 予約あり? | 未準備予約一覧表示 | 予約選択/終了 | - |
| 2 | 予約選択 | - | 予約確認 | 図書探索 | - |
| 3 | 図書探索 | 図書発見? | - | 状態確認 | 次予約処理 |
| 4 | 状態確認 | 在庫中? | 所蔵品状態確認 | 一致確認 | 次予約処理 |
| 5 | 一致確認 | 予約図書一致? | 予約図書一致確認 | 取置登録 | 次予約処理 |
| 6 | 取置登録 | - | 取置登録 | 利用者通知 | - |
| 7 | 利用者通知 | - | - | 取置管理 | - |
| 8 | 取置管理 | 利用者来館? | 取置一覧表示 | 貸出処理/期限処理 | - |
| 9a | 貸出処理 | - | 取置貸出処理 | 次予約処理 | - |
| 9b | 期限処理 | - | 取置期限切れ処理 | 次予約処理 | - |

### 4. 判断ポイント詳細分析

#### 4.1 主要判断ポイント

##### 会員有効性判断
- **判断基準**: 会員番号の存在・有効期限
- **分岐**: 有効 → 処理続行、無効 → エラー終了
- **影響範囲**: 全ての利用者向けサービス
- **例外処理**: 会員登録案内、一時利用案内

##### 貸出制限判断
- **判断基準**: 
  - 年齢別制限（小学生以下15点、中学生以上20点）
  - 資料別制限（視聴覚資料5点）
  - 延滞制限（15日以上延滞で新規貸出停止）
- **分岐**: 制限内 → 貸出可能、制限超過 → 貸出不可
- **影響範囲**: 貸出・予約サービス
- **例外処理**: 制限説明、返却促進案内

##### 予約制限判断
- **判断基準**:
  - 全体予約数制限（15点まで）
  - 視聴覚資料予約制限（5点まで）
- **分岐**: 制限内 → 予約可能、制限超過 → 予約不可
- **影響範囲**: 予約サービス
- **例外処理**: 制限説明、既存予約確認案内

##### 図書利用可能性判断
- **判断基準**: 所蔵品の現在状態
- **分岐**: 在庫中 → 即時貸出、貸出中 → 予約待ち
- **影響範囲**: 貸出・予約サービス
- **例外処理**: 代替資料案内、予約案内

##### 取置期限判断
- **判断基準**: 取置登録日から7開館日経過
- **分岐**: 期限内 → 取置継続、期限切れ → 予約キャンセル
- **影響範囲**: 取置管理業務
- **例外処理**: 期限延長（特例）、再予約案内

### 5. ビジネスルール適用フロー

#### 5.1 複合判定フロー

```mermaid
flowchart TD
    Start([貸出要求]) --> MemberValidation{会員有効性}
    MemberValidation -->|無効| MemberFail[会員無効]
    MemberValidation -->|有効| MemberTypeCheck{会員種別確認}
    
    MemberTypeCheck --> AgeRestriction{年齢別制限}
    AgeRestriction -->|小学生以下| Elementary[15点制限適用]
    AgeRestriction -->|中学生以上| MiddleHigh[20点制限適用]
    
    Elementary --> CurrentLoans{現在貸出数}
    MiddleHigh --> CurrentLoans
    
    CurrentLoans -->|制限超過| QuantityFail[数量制限エラー]
    CurrentLoans -->|制限内| MaterialType{資料種別}
    
    MaterialType -->|視聴覚資料| AVCheck{視聴覚資料数}
    MaterialType -->|一般資料| DelayCheck{延滞状況}
    
    AVCheck -->|5点超過| AVFail[視聴覚制限エラー]
    AVCheck -->|5点以内| DelayCheck
    
    DelayCheck -->|延滞なし| LoanApprove[貸出承認]
    DelayCheck -->|15日未満延滞| LoanApprove
    DelayCheck -->|15日以上延滞| DelayFail[延滞制限エラー]
    DelayCheck -->|2ヶ月以上延滞| SuspendFail[貸出停止エラー]
    
    LoanApprove --> LoanProcess[貸出処理実行]
```

#### 5.2 状態変更フロー

```mermaid
flowchart TD
    ItemAvailable[所蔵品:在庫中] --> LoanRequest{貸出要求}
    LoanRequest -->|承認| ItemLoaned[所蔵品:貸出中]
    LoanRequest -->|拒否| ItemAvailable
    
    ItemLoaned --> DueCheck{返却期限}
    DueCheck -->|期限内| ItemLoaned
    DueCheck -->|期限切れ| ItemOverdue[所蔵品:延滞中]
    
    ItemOverdue --> ReturnRequest{返却要求}
    ItemLoaned --> ReturnRequest
    ReturnRequest -->|返却| ItemAvailable
    
    ItemAvailable --> ReserveRequest{予約要求}
    ReserveRequest -->|予約| ItemReserved[所蔵品:予約済]
    ItemReserved --> RetentionReady[取置準備完了]
    RetentionReady --> ItemRetention[所蔵品:取置中]
    
    ItemRetention --> RetentionExpire{取置期限}
    RetentionExpire -->|期限切れ| ItemAvailable
    RetentionExpire -->|貸出| ItemLoaned
```

### 6. UC連携フロー分析

#### 6.1 UC実行順序

##### 貸出処理のUC連携
```
会員番号有効性確認 → 所蔵品貸出可否判定 → 貸出制限判定 → 貸出登録 → 貸出状況提示
```

##### 予約処理のUC連携
```
図書検索 → 図書詳細確認 → 会員番号有効性確認 → 予約制限判定 → 予約登録
```

##### 取置処理のUC連携
```
未準備予約一覧表示 → 予約確認 → 所蔵品状態確認 → 予約図書一致確認 → 取置登録 → 取置一覧表示
```

#### 6.2 UCの再利用パターン

| UC名 | 利用ジョブ | 利用頻度 | 共通度 |
|------|------------|----------|--------|
| 会員番号有効性確認 | 貸出、予約、返却 | 高 | 高 |
| 図書検索 | 予約、蔵書管理 | 中 | 中 |
| 貸出制限判定 | 貸出、取置貸出 | 高 | 中 |
| 予約制限判定 | 予約 | 中 | 低 |
| 所蔵品状態確認 | 貸出、取置 | 中 | 中 |

### 7. 例外フロー・エラーハンドリング

#### 7.1 例外パターン分類

##### システムエラー
- **データベース接続エラー**: システム一時停止、手動処理へ移行
- **データ整合性エラー**: 管理者アラート、データ修復処理
- **ネットワークエラー**: 自動リトライ、タイムアウト処理

##### 業務エラー
- **会員無効エラー**: 会員登録案内、一時利用案内
- **制限超過エラー**: 制限内容説明、代替案提示
- **図書未発見エラー**: 再検索案内、類似図書案内

##### 運用エラー
- **操作ミスエラー**: 操作取消、再入力案内
- **権限エラー**: 適切な権限者への引継ぎ
- **時間外エラー**: 開館時間案内、緊急連絡先案内

#### 7.2 エラー回復フロー

```mermaid
flowchart TD
    Error[エラー発生] --> ErrorType{エラー種別}
    
    ErrorType -->|システムエラー| SystemError[システムエラー処理]
    ErrorType -->|業務エラー| BusinessError[業務エラー処理]
    ErrorType -->|運用エラー| OperationError[運用エラー処理]
    
    SystemError --> Retry{リトライ可能?}
    Retry -->|可能| RetryProcess[自動リトライ]
    Retry -->|不可能| ManualFallback[手動処理移行]
    
    BusinessError --> UserGuide[利用者案内]
    UserGuide --> Alternative[代替案提示]
    
    OperationError --> StaffGuide[職員向け案内]
    StaffGuide --> Correction[操作修正]
    
    RetryProcess --> Success{成功?}
    Success -->|成功| NormalFlow[正常フロー復帰]
    Success -->|失敗| ManualFallback
    
    ManualFallback --> IncidentReport[インシデント報告]
    Alternative --> PartialSuccess[部分的解決]
    Correction --> NormalFlow
    
    IncidentReport --> Recovery[システム復旧]
    PartialSuccess --> NormalFlow
    Recovery --> NormalFlow
```

### 8. 業務フロー最適化ポイント

#### 8.1 効率化機会
- **自動判定**: 複雑な制限判定の自動化
- **バッチ処理**: 期限切れ処理等の一括実行
- **プロアクティブ通知**: 期限前の事前通知
- **ワンストップ処理**: 関連処理の統合実行

#### 8.2 品質向上機会
- **入力検証**: 早期エラー検出
- **処理確認**: 重要処理の確認ステップ
- **ログ記録**: 処理履歴の完全記録
- **状態同期**: リアルタイム状態更新

#### 8.3 ユーザビリティ向上
- **進行状況表示**: 処理進行の可視化
- **ガイダンス**: 操作手順の明確化
- **ショートカット**: 熟練者向け高速操作
- **元に戻す**: 誤操作の取消機能