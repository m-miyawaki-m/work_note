はい、**OpenAPI定義（openapi.yaml / json / xml）から Vue.js で使えるインターフェース（TypeScript型やAPI関数）を自動生成することは可能**です。
この目的には、主に以下のツールを使います。

---

## ✅ 1. OpenAPI Generator（TypeScript Axios/Fetch対応）

### 🔧 コマンド例（TypeScript Axios クライアント生成）

```bash
openapi-generator-cli generate \
  -i openapi.yaml \
  -g typescript-axios \
  -o ./generated-api
```

### 📁 出力される構成（例）

```
generated-api/
├── api.ts              // API呼び出し関数群
├── configuration.ts    // 接続設定
├── models.ts           // DTOに相当する型定義（TypeScript型）
├── ...
```

### 🎯 特徴

* 型安全なAxiosベースのクライアントが生成される
* Vue + Composition API + TypeScriptとの相性が良い

---

## ✅ 2. 他のクライアント生成オプション（`-g` で変更）

| ジェネレータ名            | 用途                     |
| ------------------ | ---------------------- |
| `typescript-axios` | Axiosベースのクライアント（Vue向き） |
| `typescript-fetch` | Fetch APIベース（軽量）       |
| `typescript`       | 型定義のみ生成（APIは自作）        |
| `nuxtjs`           | Nuxt向けクライアント（Nuxt 2）   |

---

## ✅ 3. 使用例（Vueコンポーネント側）

```ts
import { DefaultApi, Configuration } from '@/generated-api'

const api = new DefaultApi(
  new Configuration({
    basePath: 'http://localhost:8080',
  })
)

// 使用例（Composition API内）
const { data } = await api.getUsers()
```

型も自動でついてくるので補完が効き、開発効率が上がります。

---

## ✅ 4. 開発フローでの位置づけ

```text
[openapi.yaml] 
   ↓
[Java DTO + Controller] ← openapi-generator (spring)
[TypeScript クライアント] ← openapi-generator (typescript-axios)
   ↓
[Vue コンポーネントから呼び出し]
```

---

## ✅ 5. 便利な補足機能

### 🌟 `npm script` に統合

```json
"scripts": {
  "generate:client": "openapi-generator-cli generate -i openapi.yaml -g typescript-axios -o src/api"
}
```

### 🌟 CIで自動生成

* `GitHub Actions` や `npm prepare` で自動実行可能

---

## ✅ 6. 注意点

| 項目         | 内容                                                      |
| ---------- | ------------------------------------------------------- |
| ファイル肥大化    | API数が多いと `models.ts` や `api.ts` が非常に大きくなることがある          |
| Axiosバージョン | `typescript-axios` は Axios v1系と互換性あり                    |
| インターセプター等  | 認証ヘッダーなどは `configuration.ts` や `axios.create()` を拡張して設定 |

---

## ✅ まとめ

| やりたいこと          | 方法                                      |
| --------------- | --------------------------------------- |
| Java側DTO生成      | `openapi-generator -g spring`           |
| Vue側APIクライアント生成 | `openapi-generator -g typescript-axios` |
| 型安全で使いたい        | TypeScript + Axios構成がベスト                |

---

Vue 3 + TypeScript + Composition API の構成での具体的な利用コード（例）も出せますので、必要であればお知らせください！
