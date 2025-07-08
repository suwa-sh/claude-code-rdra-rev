# 環境構築分析書

## 1. ローカル環境セットアップ手順

### 1.1 前提条件

| 要件 | バージョン | 備考 |
|------|-----------|------|
| Java | 17以上 | OpenJDK推奨 |
| Gradle | 7.x以上 | Gradle Wrapper使用のため任意 |
| Git | 2.x以上 | ソースコードクローン用 |
| Graphviz | 最新版 | JIG設計ドキュメント生成用（オプション） |

### 1.2 セットアップ手順

#### Step 1: リポジトリクローン
```bash
git clone [リポジトリURL]
cd library
```

#### Step 2: 依存関係の確認
```bash
./gradlew dependencies
```

#### Step 3: アプリケーションビルド
```bash
./gradlew build
```

#### Step 4: データベース初期化
- H2インメモリデータベースが自動で初期化される
- 初期データは `src/main/resources/data.sql` から自動投入

#### Step 5: アプリケーション起動
```bash
./gradlew bootRun
```

#### Step 6: 動作確認
- アプリケーション: http://localhost:8080
- H2 Console: http://localhost:8080/h2-console
  - JDBC URL: `jdbc:h2:mem:testdb;MODE=PostgreSQL`
  - User: `sa`
  - Password: (空文字)

#### Step 7: 設計ドキュメント生成（オプション）
```bash
# Graphvizのインストールが必要
./gradlew jigReports
```
- 生成されたドキュメント: `build/jig/`

### 1.3 設定ファイル

#### application.yaml
```yaml
spring:
  sql:
    init:
      encoding: UTF-8
      mode: always
  datasource:
    url: jdbc:h2:mem:testdb;MODE=PostgreSQL
  h2:
    console:
      enabled: true
  thymeleaf:
    cache: false

management:
  tracing:
    enabled: false
    sampling:
      probability: 1.0

mybatis:
  config-location: classpath:mybatis.xml
```

### 1.4 主要な依存関係

| 依存関係 | バージョン | 用途 |
|----------|-----------|------|
| Spring Boot | 3.1.1 | Webアプリケーションフレームワーク |
| MyBatis Spring Boot Starter | 3.0.2 | O/Rマッパー |
| Thymeleaf | Spring Boot同梱 | テンプレートエンジン |
| H2 Database | Spring Boot同梱 | インメモリデータベース |
| PostgreSQL Driver | Spring Boot同梱 | 本番DB用ドライバー |
| Spring Boot Actuator | Spring Boot同梱 | 監視機能 |
| OpenTelemetry | Spring Boot同梱 | 分散トレーシング |

## 2. テスト環境セットアップ手順

### 2.1 テスト実行

#### 全テスト実行
```bash
./gradlew test
```

#### 特定のテストクラス実行
```bash
./gradlew test --tests "library.ScenarioTest"
```

#### カバレッジレポート生成
```bash
./gradlew jacocoTestReport
```
- レポート出力先: `build/reports/jacoco/test/html/index.html`

### 2.2 テスト用設定

#### application-test.properties
```properties
# テスト専用の設定ファイル
spring.datasource.url=jdbc:h2:mem:testdb;MODE=PostgreSQL;DB_CLOSE_DELAY=-1
spring.sql.init.mode=always
logging.level.org.springframework.jdbc=DEBUG
```

#### application-scenario-test.properties
```properties
# シナリオテスト専用の設定
spring.datasource.url=jdbc:h2:mem:scenario;MODE=PostgreSQL;DB_CLOSE_DELAY=-1
spring.sql.init.mode=always
```

### 2.3 テスト環境の特徴

| 項目 | 設定 | 説明 |
|------|------|------|
| データベース | H2インメモリ | テスト実行時に自動作成・削除 |
| トランザクション | 自動ロールバック | 各テスト後に自動的にデータリセット |
| ログレベル | DEBUG | SQL文などの詳細ログを出力 |
| キャッシュ | 無効 | テンプレートキャッシュを無効化 |

## 3. 本番環境セットアップ手順

### 3.1 本番環境用設定

#### application-production.yaml
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/library_production
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    driver-class-name: org.postgresql.Driver
  sql:
    init:
      mode: never
  h2:
    console:
      enabled: false
  thymeleaf:
    cache: true

management:
  tracing:
    enabled: true
    sampling:
      probability: 0.1
  endpoints:
    web:
      exposure:
        include: health,info,metrics
  endpoint:
    health:
      show-details: when-authorized

logging:
  level:
    library: INFO
    org.springframework.web: WARN
  file:
    name: /var/log/library/application.log
```

### 3.2 PostgreSQL セットアップ

#### データベース作成
```sql
CREATE DATABASE library_production;
CREATE USER library_user WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE library_production TO library_user;
```

#### スキーマ作成
```bash
# production環境では手動でスキーマ作成
psql -h localhost -U library_user -d library_production -f schema.sql
```

### 3.3 本番デプロイ手順

#### Step 1: アプリケーションビルド
```bash
./gradlew bootJar
```

#### Step 2: 環境変数設定
```bash
export SPRING_PROFILES_ACTIVE=production
export DB_USERNAME=library_user
export DB_PASSWORD=secure_password
export JAVA_OPTS="-Xms512m -Xmx1024m"
```

#### Step 3: アプリケーション起動
```bash
java $JAVA_OPTS -jar build/libs/library-*.jar
```

#### Step 4: サービス化（systemd）
```bash
# /etc/systemd/system/library.service
[Unit]
Description=Library Management System
After=network.target

[Service]
Type=simple
User=library
ExecStart=/usr/bin/java -jar /opt/library/library.jar
Restart=always
Environment=SPRING_PROFILES_ACTIVE=production
Environment=DB_USERNAME=library_user
Environment=DB_PASSWORD=secure_password

[Install]
WantedBy=multi-user.target
```

### 3.4 本番環境の監視

#### ヘルスチェック
```bash
curl http://localhost:8080/actuator/health
```

#### メトリクス取得
```bash
curl http://localhost:8080/actuator/metrics
```

#### トレーシング（Zipkin）
```yaml
management:
  tracing:
    enabled: true
    sampling:
      probability: 0.1
  zipkin:
    tracing:
      endpoint: http://zipkin-server:9411/api/v2/spans
```

## 4. 開発環境の追加ツール

### 4.1 JIG設計ドキュメント生成

#### Graphvizインストール
```bash
# macOS
brew install graphviz

# Ubuntu
sudo apt-get install graphviz

# Windows
# https://graphviz.org/download/ からダウンロード
```

#### JIGレポート生成
```bash
./gradlew jigReports
```

#### 生成されるドキュメント
- パッケージ関係図
- クラス関係図
- メソッド呼び出し図
- ビジネスルール一覧
- データベーススキーマ図

### 4.2 コード品質管理

#### SonarQube解析
```bash
./gradlew sonar
```

#### Jacoco カバレッジ
```bash
./gradlew test jacocoTestReport
```

### 4.3 IDE設定

#### IntelliJ IDEA
- Project SDK: Java 17
- Build Tool: Gradle
- Spring Boot DevTools: 有効化推奨

#### Eclipse
- Java Build Path: JRE 17
- Gradle プラグイン必須
- Spring Tool Suite推奨

## 5. トラブルシューティング

### 5.1 よくある問題

| 問題 | 原因 | 解決方法 |
|------|------|----------|
| アプリケーション起動失敗 | Java 17未インストール | Java 17をインストール |
| H2 Console接続失敗 | JDBC URL誤り | `jdbc:h2:mem:testdb;MODE=PostgreSQL`を使用 |
| JIG生成失敗 | Graphviz未インストール | Graphvizをインストール |
| テスト失敗 | データベース初期化エラー | `./gradlew clean test`で再実行 |

### 5.2 ログレベル調整

#### 開発時のデバッグ
```yaml
logging:
  level:
    library: DEBUG
    org.springframework.web: DEBUG
    org.mybatis: DEBUG
```

#### 本番環境のパフォーマンス
```yaml
logging:
  level:
    library: INFO
    org.springframework.web: WARN
    org.mybatis: WARN
```

## チェックリスト更新

- [x] ローカル環境セットアップ手順作成
- [x] テスト環境セットアップ手順作成
- [x] 本番環境セットアップ手順作成