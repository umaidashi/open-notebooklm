# TypeScriptコーディングガイドライン

## 概要
Open NotebookLMプロジェクトにおけるTypeScriptコーディング規約を定義します。

## 基本原則
1. **型安全性**: `any`型の使用を避け、適切な型定義を行う
2. **可読性**: コードは他の開発者が理解しやすいように書く
3. **一貫性**: プロジェクト全体で統一されたスタイルを維持
4. **テスタビリティ**: テストしやすい設計を心がける

## コーディング規約

### 1. 命名規則

#### 変数・関数
```typescript
// ✅ Good - camelCase
const userName = 'John';
const calculateTotal = (items: Item[]) => { /* ... */ };

// ❌ Bad
const user_name = 'John';
const CalculateTotal = (items: Item[]) => { /* ... */ };
```

#### インターフェース・型
```typescript
// ✅ Good - PascalCase
interface UserProfile {
  id: string;
  name: string;
}

type ResponseStatus = 'success' | 'error';

// ❌ Bad
interface userProfile { /* ... */ }
type response_status = 'success' | 'error';
```

#### 定数
```typescript
// ✅ Good - UPPER_SNAKE_CASE
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = 'https://api.example.com';

// ❌ Bad
const maxRetryCount = 3;
const ApiBaseUrl = 'https://api.example.com';
```

#### ファイル名
- kebab-case を使用: `user-service.ts`, `api-client.ts`
- テストファイル: `user-service.test.ts`
- 型定義ファイル: `user-service.types.ts`

### 2. 型定義

#### インターフェース vs 型エイリアス
```typescript
// ✅ Good - オブジェクトの形状にはインターフェースを使用
interface User {
  id: string;
  name: string;
}

// ✅ Good - ユニオン型や複雑な型には型エイリアスを使用
type Status = 'pending' | 'approved' | 'rejected';
type AsyncFunction<T> = () => Promise<T>;
```

#### 必須・オプショナルプロパティ
```typescript
interface Config {
  apiUrl: string;        // 必須
  timeout?: number;      // オプショナル
  retryCount?: number;   // オプショナル
}
```

### 3. 関数

#### 関数の型定義
```typescript
// ✅ Good - 明示的な戻り値の型
const add = (a: number, b: number): number => {
  return a + b;
};

// ✅ Good - 複雑な関数は型を分離
type CalculateFunction = (items: Item[]) => CalculationResult;

const calculate: CalculateFunction = (items) => {
  // 実装
};
```

#### デフォルト引数
```typescript
// ✅ Good
const createUser = (
  name: string,
  role: UserRole = UserRole.GUEST
): User => {
  // 実装
};
```

### 4. 非同期処理

#### async/await の使用
```typescript
// ✅ Good
const fetchUser = async (id: string): Promise<User> => {
  try {
    const response = await api.get(`/users/${id}`);
    return response.data;
  } catch (error) {
    logger.error('Failed to fetch user', { id, error });
    throw new UserNotFoundError(id);
  }
};

// ❌ Bad - Promiseチェーンは避ける
const fetchUser = (id: string): Promise<User> => {
  return api.get(`/users/${id}`)
    .then(response => response.data)
    .catch(error => {
      throw new UserNotFoundError(id);
    });
};
```

### 5. エラーハンドリング

#### カスタムエラークラス
```typescript
// ✅ Good
export class ValidationError extends Error {
  constructor(
    message: string,
    public readonly field: string,
    public readonly value: unknown
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}

// 使用例
throw new ValidationError('Invalid email format', 'email', email);
```

### 6. インポート・エクスポート

#### インポートの順序
```typescript
// 1. Node.js標準モジュール
import { readFile } from 'fs/promises';

// 2. 外部ライブラリ
import express from 'express';
import { z } from 'zod';

// 3. 内部モジュール（絶対パス）
import { UserService } from '@/services/user-service';
import { logger } from '@/utils/logger';

// 4. 内部モジュール（相対パス）
import { calculateTotal } from './helpers';
import type { Item } from './types';
```

### 7. クラス

#### クラスの構造
```typescript
export class UserService {
  // 1. プライベートプロパティ
  private readonly repository: UserRepository;
  
  // 2. コンストラクタ
  constructor(repository: UserRepository) {
    this.repository = repository;
  }
  
  // 3. パブリックメソッド
  async getUser(id: string): Promise<User> {
    return this.repository.findById(id);
  }
  
  // 4. プライベートメソッド
  private validateUser(user: User): void {
    // 実装
  }
}
```

### 8. コメント

#### JSDocコメント
```typescript
/**
 * ユーザー情報を取得します
 * @param id - ユーザーID
 * @returns ユーザー情報
 * @throws {UserNotFoundError} ユーザーが見つからない場合
 */
async function getUser(id: string): Promise<User> {
  // 実装
}
```

## ESLint設定

```json
{
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/recommended-requiring-type-checking",
    "prettier"
  ],
  "rules": {
    "@typescript-eslint/explicit-function-return-type": "error",
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
    "@typescript-eslint/naming-convention": [
      "error",
      {
        "selector": "interface",
        "format": ["PascalCase"]
      },
      {
        "selector": "typeAlias",
        "format": ["PascalCase"]
      },
      {
        "selector": "variable",
        "modifiers": ["const"],
        "format": ["camelCase", "UPPER_CASE"]
      }
    ]
  }
}
```

## 関連ドキュメント
- [マイクロサービス設計ガイドライン](./microservice-design-guideline.md)
- [API設計ガイドライン](./api-design-guideline.md)
- [テストガイドライン](./testing-guideline.md)
