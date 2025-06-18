# マイクロサービス設計ガイドライン

## 概要
Open NotebookLMプロジェクトにおけるマイクロサービスアーキテクチャの設計原則とガイドラインを定義します。

## 設計原則

### 1. 単一責任の原則
各マイクロサービスは単一の業務機能に責任を持つ
- ✅ `source-service`: ソース管理のみ
- ✅ `auth-service`: 認証・認可のみ
- ❌ `user-and-document-service`: 複数の責任

### 2. 疎結合
サービス間の依存を最小限に抑える
- 非同期通信の活用
- イベント駆動アーキテクチャ
- サービス間の直接的なデータベース共有を避ける

### 3. 高凝集
関連する機能を同一サービス内に配置
- ドメイン駆動設計（DDD）の境界づけられたコンテキスト
- データとビジネスロジックの一体化

## サービス構成

### コアサービス
```
services/
├── gateway-service/        # APIゲートウェイ
├── auth-service/          # 認証・認可
├── source-service/        # ソース管理
├── embedding-service/     # ベクトル化処理
├── chat-service/          # 会話管理
├── audio-service/         # 音声処理
└── agent-service/         # AIエージェント
```

### サービステンプレート
```
{service-name}/
├── src/
│   ├── api/              # APIエンドポイント定義
│   ├── domain/           # ドメインモデル・ビジネスロジック
│   ├── infrastructure/   # 外部サービス連携
│   ├── repositories/     # データアクセス層
│   └── index.ts         # エントリーポイント
├── tests/
├── Dockerfile
├── package.json
└── tsconfig.json
```

## 通信パターン

### 1. 同期通信（gRPC）
内部サービス間の高速通信に使用

```typescript
// proto/services/source.proto
syntax = "proto3";

service SourceService {
  rpc GetSource(GetSourceRequest) returns (Source);
  rpc ListSources(ListSourcesRequest) returns (ListSourcesResponse);
}
```

### 2. 非同期通信（イベントバス）
疎結合な連携に使用

```typescript
// イベント定義
interface SourceUploadedEvent {
  eventId: string;
  timestamp: Date;
  payload: {
    sourceId: string;
    userId: string;
    notebookId: string;
  };
}

// パブリッシュ
await eventBus.publish('source.uploaded', event);

// サブスクライブ
eventBus.subscribe('source.uploaded', async (event) => {
  // 処理
});
```

### 3. REST API（外部向け）
クライアントアプリケーションとの通信

```typescript
// OpenAPI仕様に準拠
/**
 * @openapi
 * /api/v1/sources:
 *   get:
 *     summary: List sources
 *     parameters:
 *       - in: query
 *         name: notebookId
 *         required: true
 *         schema:
 *           type: string
 */
```

## データ管理

### 1. データベース分離
各サービスは独自のデータベースを持つ

```yaml
# docker-compose.yml
services:
  source-db:
    image: postgres:15
    environment:
      POSTGRES_DB: source_db
  
  auth-db:
    image: postgres:15
    environment:
      POSTGRES_DB: auth_db
```

### 2. データ整合性
- **結果整合性**を許容する設計
- **Sagaパターン**による分散トランザクション
- **イベントソーシング**による状態管理

### 3. キャッシング戦略
```typescript
// Redis を使用したキャッシング
class CacheService {
  async get<T>(key: string): Promise<T | null> {
    const cached = await redis.get(key);
    return cached ? JSON.parse(cached) : null;
  }
  
  async set<T>(key: string, value: T, ttl?: number): Promise<void> {
    await redis.set(key, JSON.stringify(value), 'EX', ttl || 3600);
  }
}
```

## エラーハンドリング

### 1. エラー伝播
```typescript
// gRPCエラーコード
import { status } from '@grpc/grpc-js';

throw new GrpcError(
  status.NOT_FOUND,
  'Source not found',
  { sourceId }
);
```

### 2. サーキットブレーカー
```typescript
class CircuitBreaker {
  private failures = 0;
  private readonly threshold = 5;
  private readonly timeout = 60000; // 1分
  
  async call<T>(fn: () => Promise<T>): Promise<T> {
    if (this.isOpen()) {
      throw new Error('Circuit breaker is open');
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
}
```

### 3. リトライ戦略
```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  options: RetryOptions = {}
): Promise<T> {
  const { maxAttempts = 3, delay = 1000, backoff = 2 } = options;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxAttempts) throw error;
      
      await sleep(delay * Math.pow(backoff, attempt - 1));
    }
  }
  
  throw new Error('Retry failed');
}
```

## 監視とロギング

### 1. 構造化ログ
```typescript
import { logger } from '@/utils/logger';

logger.info('Source uploaded', {
  sourceId: source.id,
  userId: user.id,
  size: file.size,
  duration: uploadDuration,
});
```

### 2. 分散トレーシング
```typescript
// OpenTelemetry を使用
import { trace } from '@opentelemetry/api';

const tracer = trace.getTracer('source-service');

const span = tracer.startSpan('uploadSource');
try {
  // 処理
} finally {
  span.end();
}
```

### 3. メトリクス
```typescript
// Prometheus メトリクス
const uploadCounter = new Counter({
  name: 'source_uploads_total',
  help: 'Total number of source uploads',
  labelNames: ['status', 'type'],
});

uploadCounter.inc({ status: 'success', type: 'pdf' });
```

## セキュリティ

### 1. サービス間認証
```typescript
// mTLS による相互認証
const server = new grpc.Server({
  'grpc.ssl_server_certificate': serverCert,
  'grpc.ssl_server_private_key': serverKey,
  'grpc.ssl_client_root_certs': clientCACert,
});
```

### 2. APIレート制限
```typescript
const rateLimiter = new RateLimiter({
  windowMs: 15 * 60 * 1000, // 15分
  max: 100, // リクエスト数
  keyGenerator: (req) => req.user?.id || req.ip,
});
```

## デプロイメント

### 1. コンテナ化
```dockerfile
# マルチステージビルド
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### 2. ヘルスチェック
```typescript
app.get('/health', (req, res) => {
  res.json({
    status: 'healthy',
    version: process.env.VERSION,
    uptime: process.uptime(),
  });
});

app.get('/ready', async (req, res) => {
  const checks = await Promise.all([
    checkDatabase(),
    checkRedis(),
    checkDependencies(),
  ]);
  
  const isReady = checks.every(check => check.healthy);
  res.status(isReady ? 200 : 503).json({ checks });
});
```

## ベストプラクティス

1. **イミュータブルインフラ**: コンテナイメージでバージョン管理
2. **12ファクターアプリ**: 設定の外部化、ステートレス設計
3. **防御的プログラミング**: 入力検証、タイムアウト設定
4. **段階的ロールアウト**: カナリアデプロイ、ブルーグリーンデプロイ
5. **障害注入テスト**: Chaos Engineeringの実践

## 関連ドキュメント
- [API設計ガイドライン](./api-design-guideline.md)
- [gRPC設計ガイドライン](./grpc-design-guideline.md)
- [監視・ロギングガイドライン](./monitoring-logging-guideline.md)
