# Open NotebookLM ドキュメント

## 概要
このディレクトリには、Open NotebookLMプロジェクトのすべてのドキュメントが含まれています。

## ディレクトリ構造

```
docs/
├── rules/           # コーディングルール・ガイドライン
├── requirements/    # 要件・仕様書
└── logs/           # 作業ログ
```

## クイックリンク

### 📋 ルール・ガイドライン
- [ドキュメント管理ルール](./rules/documentation-management.md)
- [TypeScriptコーディングガイドライン](./rules/typescript-coding-guideline.md)
- [マイクロサービス設計ガイドライン](./rules/microservice-design-guideline.md)

### 📝 要件・仕様書
各Issueの詳細な要件定義書は以下のディレクトリに格納されています：

#### Phase 1: 基盤インフラストラクチャ
- [Issue #001](./requirements/001/) - Dockerベース開発環境の構築
- [Issue #002](./requirements/002/) - TypeScript + Node.jsプロジェクト初期化
- [Issue #003](./requirements/003/) - マイクロサービス間通信基盤
- [Issue #004](./requirements/004/) - CI/CDパイプライン構築
- [Issue #005](./requirements/005/) - コアマイクロサービス骨格作成
- [Issue #006](./requirements/006/) - Mastra AI統合基盤

#### Phase 2: NotebookLMコア機能
- [Issue #007](./requirements/007/) - ソース管理システム
- [Issue #008](./requirements/008/) - RAGシステム実装
- [Issue #009](./requirements/009/) - 精密な引用システム
- [Issue #010](./requirements/010/) - 基本的なチャットインターフェース

#### Phase 3: Audio Overview機能
- [Issue #011](./requirements/011/) - Audio Overview音声生成基盤
- [Issue #012](./requirements/012/) - Audio Overviewカスタマイズ
- [Issue #013](./requirements/013/) - インタラクティブAudio機能

#### Phase 6: 拡張データソース統合
- [Issue #014](./requirements/014/) - プロジェクト管理ツール連携

#### Phase 8: AI Agent機能
- [Issue #015](./requirements/015/) - スケジュールタスクエンジン
- [Issue #016](./requirements/016/) - プロアクティブエージェント

### 📅 作業ログ
- [2025-06-18](./logs/2025-06-18.md) - プロジェクト初期設定

## ドキュメント作成ガイド

### 新しいルールを追加する場合
1. `docs/rules/`ディレクトリに新しいMarkdownファイルを作成
2. ファイル名はkebab-case（例: `api-design-guideline.md`）
3. このREADMEに追加

### 新しい要件書を作成する場合
1. `docs/requirements/{ISSUE_NUMBER}/`ディレクトリを作成
2. Issue番号は3桁ゼロパディング（例: `001`, `002`）
3. 最低限`overview.md`を作成
4. 必要に応じて`technical-spec.md`、`api-spec.md`などを追加

### 作業ログの記録
1. 毎日の作業終了時に`docs/logs/{yyyy-mm-dd}.md`を作成
2. テンプレートは[ドキュメント管理ルール](./rules/documentation-management.md)を参照

## 貢献ガイドライン
1. ドキュメントの変更はPull Requestで行う
2. コミットメッセージは`docs:`プレフィックスを使用
3. Markdownリンターでフォーマットを確認

## 関連リンク
- [GitHubリポジトリ](https://github.com/umaidashi/open-notebooklm)
- [GitHub Project](https://github.com/umaidashi/open-notebooklm/projects)
- [Issues](https://github.com/umaidashi/open-notebooklm/issues)
