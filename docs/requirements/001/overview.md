# [Issue #001] Dockerベース開発環境の構築

## 概要
マイクロサービスアーキテクチャで動作するAI Knowledge Assistantの基盤となるDocker開発環境を構築します。すべての開発者が統一された環境で開発できるようにし、本番環境との差異を最小限に抑えます。

## 背景
- 複数のマイクロサービスを効率的に開発・テストする必要がある
- 開発環境の差異による問題を防ぐ必要がある
- 新規参加者のオンボーディングを迅速に行う必要がある

## 要件

### 機能要件

#### 1. 統合開発環境
- すべてのマイクロサービスを1コマンドで起動できる
- サービス間の依存関係が自動的に解決される
- ホットリロード機能により、コード変更が即座に反映される

#### 2. データベース環境
- 各サービス用の独立したPostgreSQLインスタンス
- Redisキャッシュサーバー
- ベクトルデータベース（開発用）

#### 3. 開発ツール
- pgAdminによるデータベース管理
- RedisInsightによるキャッシュ監視
- Jaegerによる分散トレーシング

### 非機能要件

#### 1. パフォーマンス
- 起動時間: 3分以内
- メモリ使用量: 8GB以下（全サービス起動時）
- CPU使用率: アイドル時10%以下

#### 2. 可用性
- サービスの個別再起動が可能
- 障害時の自動リスタート
- ヘルスチェック機能

#### 3. 開発効率
- ホットリロード対応
- ログの統合表示
- デバッグポートの公開

## 技術仕様

### ディレクトリ構造
```
.
├── docker/
│   ├── dev/                    # 開発環境用設定
│   │   ├── docker-compose.yml  # メインのcompose設定
│   │   ├── .env.example       # 環境変数テンプレート
│   │   └── volumes/           # 永続化データ
│   └── scripts/               # 便利スクリプト
│       ├── setup.sh          # 初期設定
│       ├── clean.sh          # クリーンアップ
│       └── logs.sh           # ログ表示
├── services/                  # 各マイクロサービス
└── Makefile                  # 開発コマンド
```

### docker-compose.yml設計
```yaml
version: '3.8'

x-common-variables: &common-variables
  NODE_ENV: development
  LOG_LEVEL: debug

services:
  # APIゲートウェイ
  gateway:
    build:
      context: ../../services/gateway-service
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
      - "9229:9229"  # デバッグポート
    environment:
      <<: *common-variables
      PORT: 3000
    volumes:
      - ../../services/gateway-service:/app
      - /app/node_modules
    depends_on:
      - auth
      - source
    networks:
      - notebooklm-network

  # 認証サービス
  auth:
    build:
      context: ../../services/auth-service
      dockerfile: Dockerfile.dev
    environment:
      <<: *common-variables
      DATABASE_URL: postgresql://auth:password@auth-db:5432/auth_db
    depends_on:
      auth-db:
        condition: service_healthy
    networks:
      - notebooklm-network

  # データベース群
  auth-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: auth
      POSTGRES_PASSWORD: password
      POSTGRES_DB: auth_db
    volumes:
      - auth-db-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U auth"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - notebooklm-network

  # Redis
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - notebooklm-network

  # 開発ツール
  pgadmin:
    image: dpage/pgadmin4:latest
    ports:
      - "5050:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@notebooklm.local
      PGADMIN_DEFAULT_PASSWORD: admin
    networks:
      - notebooklm-network

volumes:
  auth-db-data:
  source-db-data:
  redis-data:

networks:
  notebooklm-network:
    driver: bridge
```

### Makefile
```makefile
.PHONY: help up down logs clean setup

help: ## ヘルプを表示
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'

setup: ## 初期設定
	@echo "🚀 Setting up development environment..."
	@cp docker/dev/.env.example docker/dev/.env
	@docker compose -f docker/dev/docker-compose.yml build
	@echo "✅ Setup complete!"

up: ## 全サービス起動
	docker compose -f docker/dev/docker-compose.yml up -d
	@echo "✅ All services are running!"
	@echo "📱 Gateway: http://localhost:3000"
	@echo "🗄️  pgAdmin: http://localhost:5050"

down: ## 全サービス停止
	docker compose -f docker/dev/docker-compose.yml down

logs: ## ログ表示
	docker compose -f docker/dev/docker-compose.yml logs -f

clean: ## データ削除を含む完全クリーンアップ
	docker compose -f docker/dev/docker-compose.yml down -v
	@echo "⚠️  All data has been removed!"
```

### 環境変数設定（.env.example）
```env
# Common
NODE_ENV=development
LOG_LEVEL=debug

# Database
DB_HOST=localhost
DB_PORT=5432
DB_USER=notebooklm
DB_PASSWORD=changeme

# Redis
REDIS_URL=redis://localhost:6379

# Auth
JWT_SECRET=dev-secret-change-in-production
JWT_EXPIRY=7d

# AI Providers (開発用ダミー値)
OPENAI_API_KEY=sk-dev-dummy
ANTHROPIC_API_KEY=sk-ant-dev-dummy
GOOGLE_API_KEY=dev-dummy
```

## 実装計画

### Phase 1: 基本構造（2時間）
1. ディレクトリ構造の作成
2. 基本的なdocker-compose.yml
3. Makefile作成

### Phase 2: サービス設定（4時間）
1. 各マイクロサービスのDockerfile.dev作成
2. ホットリロード設定
3. デバッグポート設定

### Phase 3: データベース設定（1時間）
1. PostgreSQL設定
2. Redis設定
3. ヘルスチェック実装

### Phase 4: 開発ツール統合（1時間）
1. pgAdmin設定
2. RedisInsight設定
3. ログ集約設定

## テスト計画

### 1. 起動テスト
- `make setup`が正常に完了する
- `make up`で全サービスが起動する
- 各サービスのヘルスチェックが通る

### 2. ホットリロードテスト
- コード変更が自動的に反映される
- node_modulesの変更は反映されない

### 3. 通信テスト
- サービス間の通信が正常に動作する
- ネットワーク分離が機能する

## 制約事項
- Docker Desktop for Mac/Windowsでの動作を前提
- メモリ8GB以上のマシンが必要
- ポート3000, 5432, 6379, 5050が使用可能であること

## 関連ドキュメント
- [マイクロサービス設計ガイドライン](../../rules/microservice-design-guideline.md)
- [Issue #002 - TypeScript + Node.jsプロジェクト初期化](../002/overview.md)
- [Issue #005 - コアマイクロサービス骨格作成](../005/overview.md)
