なるほど、整理すると以下のような要件ですね：

---

## ✅ 要件のまとめ

1. **複数のDTO型（User, Post, Commentなど）** を管理したい
2. **コンポーネントからはDTO単位で渡される**（API呼び出しはコンポーネント側）
3. **タブ切り替えごとに内容が異なるが、型（DTOの構造）は同じ**
   　→ 例：タブ1はユーザーAのコメント一覧、タブ2はユーザーBのコメント一覧
4. したがって、Vuexでは `タブ番号 × DTO種別` で保存したい
   　→ 例：`comments[tabId]`、`posts[tabId]` のような構造

---

## ✅ 解決方針：2次元Map（tabId × DTO種別）で登録・取得

```ts
state = {
  comments: {
    [tabId: number]: Comment[]
  },
  posts: {
    [tabId: number]: Post[]
  },
  ...
}
```

---

## ✅ 1. 型定義（例）

### `types/User.ts`

```ts
export interface User {
  id: number
  name: string
}
```

### `types/Post.ts`

```ts
export interface Post {
  id: number
  title: string
}
```

### `types/Comment.ts`

```ts
export interface Comment {
  id: number
  text: string
}
```

---

## ✅ 2. Vuexストア `store/modules/data.ts`

```ts
import { Module } from 'vuex'
import { RootState } from '../index'
import { User } from '@/types/User'
import { Post } from '@/types/Post'
import { Comment } from '@/types/Comment'

export interface DataState {
  users: { [tabId: number]: User[] }
  posts: { [tabId: number]: Post[] }
  comments: { [tabId: number]: Comment[] }
}

const dataModule: Module<DataState, RootState> = {
  namespaced: true,

  state: (): DataState => ({
    users: {},
    posts: {},
    comments: {},
  }),

  mutations: {
    setUsers(state, payload: { tabId: number, data: User[] }) {
      state.users[payload.tabId] = payload.data
    },
    setPosts(state, payload: { tabId: number, data: Post[] }) {
      state.posts[payload.tabId] = payload.data
    },
    setComments(state, payload: { tabId: number, data: Comment[] }) {
      state.comments[payload.tabId] = payload.data
    },
  },

  getters: {
    usersByTab: (state) => (tabId: number): User[] => {
      return state.users[tabId] ?? []
    },
    postsByTab: (state) => (tabId: number): Post[] => {
      return state.posts[tabId] ?? []
    },
    commentsByTab: (state) => (tabId: number): Comment[] => {
      return state.comments[tabId] ?? []
    },
  },
}

export default dataModule
```

---

## ✅ 3. 使用方法（コンポーネント側）

### 登録時（例：コメントをタブ2に登録）

```ts
store.commit('data/setComments', {
  tabId: 2,
  data: comments, // APIレスポンスから取得した配列
})
```

### 取得時（例：タブ2のコメント一覧を表示）

```ts
const comments = computed(() => store.getters )
```

※Composition APIでも使いやすいように、引数付き `getter` にしてあります。

---

## ✅ 4. 必要に応じて汎用化（optional）

もし DTOの種類が増えていくことが予想される場合、以下のように汎用 `mutation` にすることも可能です：

```ts
mutations: {
  setData<T>(state, payload: { type: 'users' | 'posts' | 'comments', tabId: number, data: T[] }) {
    (state[payload.type] as Record<number, T[]>)[payload.tabId] = payload.data
  }
}
```

呼び出し例：

```ts
store.commit('data/setData', { type: 'posts', tabId: 1, data: postList })
```

---

## ✅ まとめ

| 要件              | 実装方針                                                  |
| --------------- | ----------------------------------------------------- |
| タブごとに異なるデータを保持  | `Record<tabId, DTO[]>` の構造で state を定義                 |
| DTOの種類が複数       | 各 DTOごとに state / mutation / getter を定義                |
| タブ + DTO で管理したい | `setXxx({ tabId, data })` の形で登録、`xxxByTab(tabId)` で取得 |

---

ご希望があれば：

* タブ切り替え時に「未取得ならAPIコールする」ロジック
* 「現在表示中のタブ」state管理との連携方法
* 永続化（ローカルストレージなど）対応

などもご提案できます。必要であればお気軽にどうぞ！
