# アーキテクチャ設計書

## 1. C4 システムコンテキスト図

### 1.1 システムコンテキスト図

```mermaid
graph TB
    subgraph "図書館利用者"
        U[図書館利用者]
    end
    
    subgraph "図書館スタッフ"
        L[司書]
        A[図書館管理者]
    end
    
    subgraph "図書館管理システム"
        LS[図書館管理システム<br/>Spring Boot アプリケーション]
    end
    
    subgraph "データ基盤"
        DB[(データベース<br/>H2/PostgreSQL)]
    end
    
    subgraph "開発・運用基盤"
        JIG[JIG<br/>設計可視化]
        MON[監視システム<br/>Actuator/OpenTelemetry]
        QA[品質管理<br/>SonarCloud]
    end
    
    U -->|図書検索・予約| LS
    L -->|貸出・返却・予約管理| LS
    A -->|延滞管理・レポート| LS
    
    LS -->|データ永続化| DB
    DB -->|データ読込| LS
    
    LS -->|設計ドキュメント生成| JIG
    LS -->|メトリクス・トレース| MON
    LS -->|コード品質分析| QA
```

### 1.2 システムコンテキスト説明

| 名前 | 説明 |
|------|------|
| 図書館利用者 | 図書館の資料を借りたり予約したりする一般利用者 |
| 司書 | 図書館で貸出・返却・予約管理業務を行うスタッフ |
| 図書館管理者 | システムの管理・運用と統計レポートを確認する責任者 |
| 図書館管理システム | 図書館の貸出・返却・予約業務を支援するWebアプリケーション |
| データベース | システムのデータを永続化するリレーショナルデータベース（開発: H2、本番: PostgreSQL） |
| JIG | ソースコードから設計ドキュメントを自動生成するツール |
| 監視システム | アプリケーションの健全性とパフォーマンスを監視するシステム |
| 品質管理 | 静的コード解析により品質を管理するクラウドサービス |

## 2. C4 コンテナ図

### 2.1 コンテナ図

```mermaid
graph TB
    subgraph "ユーザー"
        U[Webブラウザ]
    end
    
    subgraph "図書館管理システム"
        subgraph "Webアプリケーション"
            PC[プレゼンテーション層<br/>Spring MVC<br/>Thymeleaf]
            AC[アプリケーション層<br/>Service/Scenario]
            DC[ドメイン層<br/>Domain Model]
            IC[インフラストラクチャ層<br/>Repository実装]
        end
    end
    
    subgraph "データストア"
        DB[(リレーショナルDB<br/>H2/PostgreSQL)]
    end
    
    subgraph "外部システム"
        MB[MyBatis<br/>O/Rマッパー]
        SB[Spring Boot<br/>Framework]
    end
    
    U -->|HTTP/HTTPS| PC
    PC --> AC
    AC --> DC
    DC --> IC
    IC -->|SQL| MB
    MB -->|JDBC| DB
    
    PC -.->|依存| SB
    AC -.->|依存| SB
    IC -.->|依存| SB
```

### 2.2 コンテナ説明

| 名前 | 説明 |
|------|------|
| Webブラウザ | ユーザーがシステムにアクセスするためのクライアント |
| プレゼンテーション層 | HTTPリクエストを受け付け、HTMLレスポンスを返すWeb層 |
| アプリケーション層 | ビジネスユースケースを実現するサービス・シナリオ層 |
| ドメイン層 | ビジネスルールとエンティティを含む中核的なビジネスロジック |
| インフラストラクチャ層 | データベースアクセスと外部システム連携を担当 |
| リレーショナルDB | アプリケーションデータを永続化するデータベース |
| MyBatis | SQLとオブジェクトをマッピングするO/Rマッパー |
| Spring Boot | アプリケーション全体の基盤となるフレームワーク |

## 3. C4 コンポーネント図

### 3.1 プレゼンテーション層コンポーネント図

```mermaid
graph TB
    subgraph "プレゼンテーション層"
        TC[TopController<br/>トップページ]
        LC[LoanRegisterController<br/>貸出管理]
        RC[ReservationController<br/>予約管理]
        RTC[RetentionController<br/>取置管理]
        REC[ReturnMaterialController<br/>返却管理]
        ESC[EntrySearchController<br/>蔵書検索]
        BCA[BaseControllerAdvice<br/>例外処理]
    end
    
    TC -->|委譲| AS[アプリケーション層]
    LC -->|委譲| AS
    RC -->|委譲| AS
    RTC -->|委譲| AS
    REC -->|委譲| AS
    ESC -->|委譲| AS
    BCA -->|横断的処理| TC
    BCA -->|横断的処理| LC
    BCA -->|横断的処理| RC
```

### 3.2 アプリケーション層コンポーネント図

```mermaid
graph TB
    subgraph "シナリオ"
        LS[LoanScenario<br/>貸出シナリオ]
        RS[ReservationScenario<br/>予約シナリオ]
        RCS[ReservationCancellationScenario<br/>予約取消シナリオ]
        RTS[RetentionScenario<br/>取置シナリオ]
        RES[RetentionExpireScenario<br/>取置期限切れシナリオ]
        RNS[ReturnsScenario<br/>返却シナリオ]
    end
    
    subgraph "サービス"
        MQS[MemberQueryService<br/>会員照会]
        LQS[LoanQueryService<br/>貸出照会]
        LRS[LoanRecordService<br/>貸出記録]
        RQS[ReservationQueryService<br/>予約照会]
        RRS[ReservationRecordService<br/>予約記録]
        IQS[ItemQueryService<br/>蔵書照会]
    end
    
    LS --> MQS
    LS --> LQS
    LS --> LRS
    RS --> RQS
    RS --> RRS
    RS --> MQS
```

### 3.3 ドメイン層コンポーネント図

```mermaid
graph TB
    subgraph "貸出ドメイン"
        L[Loan<br/>貸出]
        LR[LoanRequest<br/>貸出依頼]
        DD[DueDate<br/>返却期限]
        LO[Loanability<br/>貸出可否]
        RES[Restriction<br/>貸出制限]
    end
    
    subgraph "予約ドメイン"
        R[Reservation<br/>予約]
        RR[ReservationRequest<br/>予約依頼]
        RA[ReservationAvailability<br/>予約可否]
        WO[WaitingOrder<br/>予約待ち順位]
    end
    
    subgraph "会員ドメイン"
        M[Member<br/>会員]
        MT[MemberType<br/>会員種別]
        MS[MemberStatus<br/>会員状態]
    end
    
    subgraph "蔵書ドメイン"
        I[Item<br/>蔵書]
        IS[ItemStatus<br/>蔵書状態]
        E[Entry<br/>資料]
        ET[EntryType<br/>資料種別]
    end
    
    L --> M
    L --> I
    L --> DD
    LR --> LO
    LO --> RES
    
    R --> M
    R --> E
    R --> WO
    RR --> RA
```

### 3.4 コンポーネント説明

| 名前 | 説明 |
|------|------|
| Controller | HTTPリクエストを受け付けてレスポンスを返すWebコントローラー |
| Scenario | 複数のサービスを組み合わせてユースケースを実現するクラス |
| Service | 単一の責務を持つビジネスロジックを提供するクラス |
| Domain Model | ビジネスルールとデータを保持するエンティティ・値オブジェクト |
| Repository | データベースアクセスを抽象化するインターフェース |
| Restriction | 貸出制限ルール（延滞制限、冊数制限など） |
| Availability | 予約・取置の可否判定ロジック |

## 4. 物理構成図

### 4.1 物理構成図

```mermaid
graph TB
    subgraph "クライアント環境"
        BR[Webブラウザ<br/>Chrome/Firefox/Safari]
    end
    
    subgraph "アプリケーションサーバー"
        subgraph "JVM"
            APP[Spring Boot App<br/>Java 17]
            EMB[組み込みTomcat<br/>Webサーバー]
            HCP[HikariCP<br/>コネクションプール]
        end
    end
    
    subgraph "データベースサーバー"
        subgraph "開発環境"
            H2[H2 Database<br/>インメモリDB]
        end
        subgraph "本番環境"
            PG[PostgreSQL<br/>RDBMS]
        end
    end
    
    subgraph "外部サービス"
        SC[SonarCloud<br/>品質分析]
        GH[GitHub Actions<br/>CI/CD]
    end
    
    subgraph "監視基盤"
        ACT[Spring Actuator<br/>ヘルスチェック]
        OT[OpenTelemetry<br/>トレーシング]
        ZIP[Zipkin<br/>トレース可視化]
    end
    
    BR -->|HTTP/8080| EMB
    EMB --> APP
    APP --> HCP
    HCP -->|JDBC| H2
    HCP -->|JDBC| PG
    
    APP --> ACT
    APP --> OT
    OT --> ZIP
    
    APP -.->|分析| SC
    APP -.->|ビルド・デプロイ| GH
```

### 4.2 物理構成説明

| 名前 | 説明 |
|------|------|
| Webブラウザ | エンドユーザーが使用するWebクライアント |
| Spring Boot App | Javaで実装されたWebアプリケーション本体 |
| 組み込みTomcat | Spring Bootに内蔵されたWebサーバー |
| HikariCP | 高性能なデータベースコネクションプール |
| H2 Database | 開発・テスト用のインメモリデータベース |
| PostgreSQL | 本番環境用のリレーショナルデータベース |
| SonarCloud | コード品質を継続的に分析するクラウドサービス |
| GitHub Actions | CI/CDパイプラインを実行する自動化サービス |
| Spring Actuator | アプリケーションの健全性を監視する組み込み機能 |
| OpenTelemetry | 分散トレーシングデータを収集するエージェント |
| Zipkin | トレースデータを可視化する監視ツール |

## 5. レイヤー構成

### 5.1 アプリケーション全体のレイヤー構成

```mermaid
graph TB
    subgraph "プレゼンテーション層"
        CTL[Controller]
        VIEW[View<br/>Thymeleaf]
        API[REST API<br/>JSON]
    end
    
    subgraph "アプリケーション層"
        SCN[Scenario]
        SVC[Service]
        DTO[DTO]
    end
    
    subgraph "ドメイン層"
        ENT[Entity]
        VO[Value Object]
        AGG[Aggregate]
        RULE[Business Rule]
        REPO_IF[Repository Interface]
    end
    
    subgraph "インフラストラクチャ層"
        REPO_IMPL[Repository実装]
        MAPPER[MyBatis Mapper]
        DS[DataSource]
    end
    
    CTL --> SCN
    CTL --> VIEW
    API --> SVC
    
    SCN --> SVC
    SVC --> ENT
    SVC --> REPO_IF
    
    ENT --> VO
    AGG --> ENT
    RULE --> ENT
    
    REPO_IF <-.->|実装| REPO_IMPL
    REPO_IMPL --> MAPPER
    MAPPER --> DS
```

### 5.2 貸出機能のレイヤー構成

```mermaid
graph TB
    subgraph "プレゼンテーション層"
        LRC[LoanRegisterController]
        LFV[loan/form.html]
        LCV[loan/completed.html]
    end
    
    subgraph "アプリケーション層"
        LSC[LoanScenario]
        LQS[LoanQueryService]
        LRS[LoanRecordService]
        MQS[MemberQueryService]
    end
    
    subgraph "ドメイン層"
        LN[Loan]
        LNR[LoanRequest]
        LNA[Loanability]
        LNB[LoanRepository]
        MEM[Member]
        ITM[Item]
    end
    
    subgraph "インフラストラクチャ層"
        LDS[LoanDataSource]
        LMP[LoanMapper]
        LMX[LoanMapper.xml]
    end
    
    LRC -->|表示| LFV
    LRC -->|実行| LSC
    LRC -->|完了| LCV
    
    LSC --> LQS
    LSC --> LRS
    LSC --> MQS
    
    LRS --> LN
    LRS --> LNR
    LNR --> LNA
    LRS --> LNB
    
    LN --> MEM
    LN --> ITM
    
    LNB <-.->|実装| LDS
    LDS --> LMP
    LMP --> LMX
```

### 5.3 レイヤー説明

| 名前 | 説明 |
|------|------|
| プレゼンテーション層 | ユーザーインターフェースとHTTPリクエスト/レスポンスを処理 |
| アプリケーション層 | ユースケースの実現とトランザクション境界の管理 |
| ドメイン層 | ビジネスルールとエンティティを含む中核的なロジック |
| インフラストラクチャ層 | データベースアクセスや外部システムとの連携を実装 |
| Controller | HTTPリクエストを受け付けて適切なサービスに委譲 |
| Scenario | 複数のサービスを組み合わせて1つのユースケースを実現 |
| Service | 単一の責務を持つビジネスロジックを提供 |
| Entity | ビジネス上の概念を表現し、振る舞いを持つオブジェクト |
| Value Object | 不変で等価性によって識別される値 |
| Repository | データアクセスを抽象化するインターフェース |
| Mapper | SQLとJavaオブジェクトのマッピングを定義 |
| DataSource | データベース接続を管理するコンポーネント |

## チェックリスト更新

- [x] C4 システムコンテキスト図作成
- [x] C4 コンテナ図作成
- [x] C4 コンポーネント図作成
- [x] 物理構成図作成
- [x] レイヤー構成の文書化