# 機能分析書

## 1. ユースケース一覧

### 1.1 システム全体のユースケース

| ユースケース | エントリーポイント | 処理概要 |
|------------|------------------|----------|
| トップページ表示 | GET `/` | システムのトップページを表示する |
| 会員番号有効性を確認する | エントリーポイントなし | 会員番号の妥当性と有効性を検証する共通処理 |
| 所蔵品貸出可否を判定する | エントリーポイントなし | 所蔵品の状態から貸出可能性を判定する共通処理 |
| 貸出制限を判定する | エントリーポイントなし | 会員の現在の貸出状況から新規貸出の可否を判定する共通処理 |
| 貸出登録画面表示 | GET `/loan/register` | 貸出登録フォームを表示する |
| 貸出を登録する | POST `/loan/register` | 会員と所蔵品を指定して貸出処理を実行する |
| 貸出完了確認 | GET `/loan/register/completed` | 貸出処理の完了を確認する画面を表示する |
| 所蔵品を検索する | GET `/reservation/entries/search` | キーワードで所蔵品目を検索する |
| 予約登録開始 | GET `/reservation/register` | 予約登録プロセスを開始（検索画面へ誘導） |
| 予約フォーム表示 | GET `/reservation/register?entry={entryNumber}` | 指定品目の予約登録フォームを表示する |
| 予約を登録する | POST `/reservation/register` | 会員と品目を指定して予約処理を実行する |
| 予約完了確認 | GET `/reservation/register/completed` | 予約処理の完了を確認する画面を表示する |
| 予約制限を判定する | エントリーポイントなし | 会員の現在の予約状況から新規予約の可否を判定する共通処理 |
| 予約一覧を表示する | GET `/retentions/requests` | 未準備の予約一覧を表示する |
| 取置フォーム表示 | GET `/retentions/requests/{reservationNumber}` | 指定予約の取置登録フォームを表示する |
| 予約を取り消す | POST `/retentions/requests/canceled?notAvailable={reservationNumber}` | 指定された予約を取り消す |
| 取置一覧を表示する | GET `/retentions` | 現在の取置一覧を表示する |
| 取置を登録する | POST `/retentions` | 予約から取置への変換処理を実行する |
| 取置品を貸し出す | POST `/retentions/loans?loaned={itemNumber}` | 取置品を実際に貸し出す |
| 取置を期限切れにする | POST `/retentions/loans?expired={retentionNumber}` | 期限切れの取置を解放する |
| 返却登録画面表示 | GET `/returns` | 返却登録フォームを表示する |
| 返却を登録する | POST `/returns` | 所蔵品の返却処理を実行する |
| 返却完了確認 | GET `/returns/completed` | 返却処理の完了を確認する画面を表示する |
| 延滞状況を確認する | エントリーポイントなし | 貸出の返却期限超過を確認する共通処理 |
| 取置期限切れを処理する | エントリーポイントなし | 取置期限を超過した資料を解放する定期処理 |

### 1.2 アクター別ユースケース

#### 図書館利用者
| ユースケース | 使用頻度 | 重要度 |
|------------|---------|--------|
| 所蔵品を検索する | 高 | 高 |
| （予約フォーム経由で）予約を登録する | 中 | 高 |
| （司書経由で）貸出を登録する | 高 | 最高 |
| （司書経由で）返却を登録する | 高 | 最高 |

#### 司書
| ユースケース | 使用頻度 | 重要度 |
|------------|---------|--------|
| 貸出を登録する | 高 | 最高 |
| 返却を登録する | 高 | 最高 |
| 予約一覧を表示する | 中 | 高 |
| 取置を登録する | 中 | 高 |
| 取置一覧を表示する | 中 | 高 |
| 取置品を貸し出す | 中 | 高 |
| 取置を期限切れにする | 低 | 中 |
| 予約を取り消す | 低 | 中 |

#### 図書館管理者
| ユースケース | 使用頻度 | 重要度 |
|------------|---------|--------|
| （実装予定）貸出状況一覧を表示する | 低 | 中 |
| （実装予定）延滞状況一覧を表示する | 低 | 高 |
| （実装予定）業務統計を表示する | 低 | 中 |

## 2. ユースケースごとのシーケンス図

### 2.1 貸出を登録する

```mermaid
sequenceDiagram
    participant U as 利用者
    participant L as 司書
    participant C as LoanRegisterController
    participant LS as LoanScenario
    participant MQS as MemberQueryService
    participant IQS as ItemQueryService
    participant LQS as LoanQueryService
    participant LRS as LoanRecordService
    participant DB as データベース

    U->>L: 貸出を申し込む
    L->>C: POST /loan/register
    C->>LS: loanBook(request)
    
    Note over LS: 会員確認
    LS->>MQS: findBy(memberNumber)
    MQS->>DB: 会員情報検索
    DB-->>MQS: 会員情報
    MQS-->>LS: Member
    
    Note over LS: 所蔵品確認
    LS->>IQS: findBy(itemNumber)
    IQS->>DB: 所蔵品情報検索
    DB-->>IQS: 所蔵品情報
    IQS-->>LS: ItemWithStatus
    
    Note over LS: 貸出可否判定
    LS->>LQS: findLoanabilityOf(member)
    LQS->>DB: 現在の貸出情報検索
    DB-->>LQS: 貸出一覧
    LQS-->>LS: Loanability
    
    Note over LS: 貸出実行
    LS->>LRS: registerLoan(loanRequest)
    LRS->>DB: 貸出記録登録
    DB-->>LRS: 登録完了
    LRS-->>LS: 貸出番号
    
    LS-->>C: 貸出完了
    C-->>L: 貸出完了画面
    L-->>U: 貸出完了通知
```

### 2.2 予約を登録する

```mermaid
sequenceDiagram
    participant U as 利用者
    participant C as ReservationController
    participant RS as ReservationScenario
    participant MQS as MemberQueryService
    participant MTS as MaterialQueryService
    participant RQS as ReservationQueryService
    participant RRS as ReservationRecordService
    participant DB as データベース

    U->>C: GET /reservation/entries/search
    C-->>U: 検索画面
    U->>C: 検索実行
    C-->>U: 検索結果
    
    U->>C: GET /reservation/register?entry=XXX
    C->>MTS: searchByKeyword()
    MTS->>DB: 品目情報検索
    DB-->>MTS: 品目情報
    MTS-->>C: Entries
    C-->>U: 予約フォーム
    
    U->>C: POST /reservation/register
    C->>RS: reserve(request)
    
    Note over RS: 会員確認
    RS->>MQS: findBy(memberNumber)
    MQS->>DB: 会員情報検索
    DB-->>MQS: 会員情報
    MQS-->>RS: Member
    
    Note over RS: 予約可否判定
    RS->>RQS: reservationAvailability(member)
    RQS->>DB: 現在の予約情報検索
    DB-->>RQS: 予約一覧
    RQS-->>RS: ReservationAvailability
    
    Note over RS: 予約実行
    RS->>RRS: reserve(request)
    RRS->>DB: 予約記録登録
    DB-->>RRS: 登録完了
    RRS-->>RS: 予約番号
    
    RS-->>C: 予約完了
    C-->>U: 予約完了画面
```

### 2.3 取置を登録する

```mermaid
sequenceDiagram
    participant L as 司書
    participant C as RetentionController
    participant RTS as RetentionScenario
    participant RQS as ReservationQueryService
    participant IQS as ItemQueryService
    participant RRS as RetentionRecordService
    participant DB as データベース

    L->>C: GET /retentions/requests
    C->>RQS: 未準備予約検索
    RQS->>DB: 予約情報検索
    DB-->>RQS: 予約一覧
    RQS-->>C: Reservations
    C-->>L: 予約一覧画面
    
    L->>C: GET /retentions/requests/{number}
    C->>RQS: 予約詳細取得
    RQS-->>C: Reservation
    C-->>L: 取置フォーム
    
    L->>C: POST /retentions
    C->>RTS: retain(retention)
    
    Note over RTS: 予約確認
    RTS->>RQS: showReservation(number)
    RQS->>DB: 予約情報検索
    DB-->>RQS: 予約情報
    RQS-->>RTS: ReservedMaterial
    
    Note over RTS: 所蔵品確認
    RTS->>IQS: findBy(itemNumber)
    IQS->>DB: 所蔵品情報検索
    DB-->>IQS: 所蔵品情報
    IQS-->>RTS: ItemWithStatus
    
    Note over RTS: 資料照合
    RTS->>RTS: validateMaterialMatch()
    
    Note over RTS: 取置実行
    RTS->>RRS: registerRetention(retention)
    RRS->>DB: 取置記録登録
    DB-->>RRS: 登録完了
    RRS-->>RTS: 完了
    
    RTS-->>C: 取置完了
    C-->>L: 取置一覧画面へ
```

### 2.4 返却を登録する

```mermaid
sequenceDiagram
    participant U as 利用者
    participant L as 司書
    participant C as ReturnMaterialController
    participant RS as ReturnsScenario
    participant RMS as ReturnMaterialRecordService
    participant DB as データベース

    U->>L: 返却を申し込む
    L->>C: GET /returns
    C-->>L: 返却フォーム
    
    L->>C: POST /returns
    C->>RS: returned(returned)
    
    Note over RS: 返却処理
    RS->>RMS: registerReturning(returned)
    RMS->>DB: 返却記録登録
    DB-->>RMS: 登録完了
    RMS-->>RS: 完了
    
    RS-->>C: 返却完了
    C-->>L: 返却完了画面
    L-->>U: 返却完了通知
```

### 2.5 取置品を貸し出す

```mermaid
sequenceDiagram
    participant U as 利用者
    participant L as 司書
    participant C as RetentionController
    participant RES as RetentionExpireScenario
    participant RQS as RetentionQueryService
    participant LS as LoanScenario
    participant DB as データベース

    U->>L: 取置品の受け取り
    L->>C: GET /retentions
    C->>RQS: retentionAvailabilities()
    RQS->>DB: 取置情報検索
    DB-->>RQS: 取置一覧
    RQS-->>C: RetainedList
    C-->>L: 取置一覧画面
    
    L->>C: POST /retentions/loans?loaned=XXX
    C->>RES: toLoaned(itemNumber)
    
    Note over RES: 取置情報取得
    RES->>RQS: retainedBy(itemNumber)
    RQS->>DB: 取置情報検索
    DB-->>RQS: 取置情報
    RQS-->>RES: Retained
    
    Note over RES: 貸出依頼変換
    RES->>RES: retained.toLoanRequest()
    
    Note over RES: 貸出実行
    RES->>LS: loanBook(loanRequest)
    LS->>DB: 貸出処理
    DB-->>LS: 完了
    LS-->>RES: 貸出完了
    
    RES-->>C: 貸出完了
    C-->>L: 貸出完了画面へ
```

### 2.6 予約を取り消す

```mermaid
sequenceDiagram
    participant L as 司書
    participant C as ReservationController
    participant RCS as ReservationCancellationScenario
    participant RRS as ReservationRecordService
    participant DB as データベース

    L->>C: GET /retentions/requests
    C-->>L: 予約一覧画面
    
    L->>C: POST /retentions/requests/canceled?notAvailable=XXX
    C->>RCS: cancel(reservationNumber)
    
    Note over RCS: 予約取消処理
    RCS->>RRS: cancel(reservationNumber)
    RRS->>DB: 予約取消処理
    DB-->>RRS: 処理完了
    RRS-->>RCS: 完了
    
    RCS-->>C: 取消完了
    C-->>L: 予約一覧画面へ
```

### 2.7 延滞状況を確認する（内部処理）

```mermaid
sequenceDiagram
    participant System as システム
    participant LQS as LoanQueryService
    participant LR as LoanRepository
    participant DB as データベース
    participant DueDate as DueDate
    participant Dues as Dues

    System->>LQS: findLoanabilityOf(member)
    
    Note over LQS: 会員の貸出情報取得
    LQS->>LR: findLoansBy(memberNumber)
    LR->>DB: 貸出情報検索
    DB-->>LR: 貸出一覧
    LR-->>LQS: Loans
    
    Note over LQS: 返却期限チェック
    loop 各貸出
        LQS->>DueDate: isOverdue()
        DueDate-->>LQS: 延滞状態
    end
    
    Note over LQS: 延滞日数集計
    LQS->>Dues: delayStatus()
    Dues-->>LQS: DelayStatus
    
    Note over LQS: 貸出可否判定
    LQS->>LQS: applyRestrictions()
    LQS-->>System: Loanability
```

### 2.8 取置期限切れを処理する

```mermaid
sequenceDiagram
    participant L as 司書
    participant C as RetentionController
    participant RES as RetentionExpireScenario
    participant RRS as RetentionRecordService
    participant DB as データベース

    L->>C: GET /retentions
    C-->>L: 取置一覧（期限切れ表示）
    
    L->>C: POST /retentions/loans?expired=XXX
    C->>RES: expire(retentionNumber)
    
    Note over RES: 期限切れ処理
    RES->>RRS: expire(retentionNumber)
    RRS->>DB: 取置解放処理
    DB-->>RRS: 処理完了
    RRS-->>RES: 完了
    
    RES-->>C: 期限切れ処理完了
    C-->>L: 取置一覧画面へ
```

## チェックリスト更新

- [x] ユースケース一覧作成
- [x] ユースケースごとにシーケンス図を作成