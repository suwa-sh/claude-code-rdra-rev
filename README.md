# claude-code-rdra-rev

既存のリポジトリから**RDRAモデルのたたき台**をリバースエンジニアリングする**claude code カスタムスラッシュコマンド**です。

- [RDRA 公式サイト](https://www.rdra.jp/)
- [claude code カスタムスラッシュコマンド](https://docs.anthropic.com/ja/docs/claude-code/slash-commands#%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%A0%E3%82%B9%E3%83%A9%E3%83%83%E3%82%B7%E3%83%A5%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89)

## インストール

```sh
curl -Lo \
  ~/.claude/commands/rdra-rev.md \
  https://raw.githubusercontent.com/suwa-sh/claude-code-rdra-rev/refs/heads/main/.claude/commands/rdra-rev.md
```

## 利用方法

- 前提
  - 分析対象のリポジトリがgit clone済み
- リポジトリの親ディレクトリで`claude code`を起動

  ```sh
  cd path/to/repo-parent
  claude
  ```

- カスタムスラッシュコマンドに、対象リポジトリのディレクトリを指定

  ```txt
  /rdra-rev ./repo
  ```

  - 複数リポジトリに分割している場合
    - `/rdra-rev ./frontend ./backend ./infra`
  - 実行イメージ
    - ![](https://share.cleanshot.com/39WKWHw3+)

### 中断からの再開

- `--continue`オプションを追加して実行してください

  ```txt
  /rdra-rev --continue ./repo
  ```

  - `docs/rdra-rev/rdra-progress-checklist.md`の進捗に合わせて、処理を再開します。
  - 実行イメージ
    - ![](https://share.cleanshot.com/lJXk59QK+)

### 出力内容

- あくまで**たたき台**です。コードにない部分はLLMが推測した結果です。
  - 各種RDRAモデルのたたき台
    - ![](https://share.cleanshot.com/B6R1sGPv+)
    - ![](https://share.cleanshot.com/dpLMHLgt+)
    - ![](https://share.cleanshot.com/yw2SYh17+)
  - RDRAシートのたたき台
    - ![](https://share.cleanshot.com/xyfd4Vxr+)
- ディレクトリ構成

  ```txt
  ./docs
  └── rdra-rev
      ├── 00-rdra-summary.md
      ├── consistency-check-report.md
      ├── directory-analysis
      ├── phase1-overview
      │   ├── 01-multi-directory-analysis.md
      │   ├── 02-system-context.md
      │   └── 03-stakeholders.md
      ├── phase2-business
      │   ├── 04-business-usecase.md
      │   ├── 05-business-flow.md
      │   └── 06-variations-conditions.md
      ├── phase3-system
      │   ├── 07-uc-complex.md
      │   ├── 08-information-model.md
      │   └── 09-state-model.md
      ├── rdra-progress-checklist.md
      └── rdra-sheets
          ├── actor-sheet.md
          ├── buc-sheet.md
          ├── condition-sheet.md
          ├── information-sheet.md
          ├── state-sheet.md
          ├── system-sheet.md
          └── variation-sheet.md
  ```

## サンプル

git submoduleに含めている「[増田さんのRDRA2.0 図書館システム実装サンプル: system-sekkei/library](https://github.com/system-sekkei/library)」を対象に分析した結果です。

- [分析サマリー: 00-rdra-summary.md](./docs/rdra-rev/00-rdra-summary.md)
- [結果を転記したRDRA定義分析シート](https://docs.google.com/spreadsheets/d/1ycV-aucZ7na0z9yNc7cnx0mm_LKeKxgvvwPuTjXtTgQ)

## 既知の課題

- mermaidの文法エラーになることがある
- RDRA定義分析シートで不整合が出る
