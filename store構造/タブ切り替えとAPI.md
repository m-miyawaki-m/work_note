承知しました！
以下のような前提に基づいて、**DefineComponent + Composition API + API呼び出しはコンポーネント側のみ**という条件で、**タブ切り替え時に未取得ならAPIを投げ、取得結果をVuexに登録する設計**を提示します。

---

## ✅ 前提条件まとめ

| 項目      | 内容                                          |
| ------- | ------------------------------------------- |
| Vue構成   | Vue 3 + Composition API + `defineComponent` |
| API呼び出し | 必ず各コンポーネントから行う（storeからは投げない）                |
| Store構成 | 1モジュールで複数DTO・タブ番号別に管理（例：`comments[tabId]`）  |
| 処理内容    | タブ切り替え時、未取得のときのみAPI呼び出し・storeへcommit        |

---

## ✅ 1. Vuex store（`modules/data.ts`）

```ts
import { Module } from 'vuex'
import { RootState } from '../index'
import { Comment } from '@/types/Comment'
import { Post } from '@/types/Post'
import { User } from '@/types/User'

export interface DataState {
  comments: { [tabId: number]: Comment[] }
  posts: { [tabId: number]: Post[] }
  users: { [tabId: number]: User[] }
}

const dataModule: Module<DataState, RootState> = {
  namespaced: true,

  state: (): DataState => ({
    comments: {},
    posts: {},
    users: {},
  }),

  mutations: {
    setComments(state, payload: { tabId: number, data: Comment[] }) {
      state.comments[payload.tabId] = payload.data
    },
    setPosts(state, payload: { tabId: number, data: Post[] }) {
      state.posts[payload.tabId] = payload.data
    },
    setUsers(state, payload: { tabId: number, data: User[] }) {
      state.users[payload.tabId] = payload.data
    },
  },

  getters: {
    commentsByTab: (state) => (tabId: number): Comment[] => {
      return state.comments[tabId] ?? []
    },
    hasComments: (state) => (tabId: number): boolean => {
      return !!state.comments[tabId]
    },
  },
}

export default dataModule
```

---

## ✅ 2. コンポーネント側（DefineComponent + Composition API）

```ts
<script lang="ts">
import { defineComponent, ref, computed, watch, onMounted } from 'vue'
import { useStore } from 'vuex'
import { RootState } from '@/store'
import { Comment } from '@/types/Comment'

export default defineComponent({
  name: 'CommentTabPanel',

  setup() {
    const store = useStore<RootState>()
    const currentTab = ref<number>(1)

    // getter 経由で現在のタブのコメントを取得
    const comments = computed<Comment[]>(() => store.getters['data/commentsByTab'](currentTab.value))

    // API fetch（storeからではなく、ここで呼び出す）
    const fetchComments = async (tabId: number) => {
      const res = await fetch(`/api/comments?tab=${tabId}`)
      const data: Comment[] = await res.json()
      store.commit('data/setComments', { tabId, data })
    }

    // 初期表示 or タブ切り替え時にデータがなければfetch
    const ensureComments = async () => {
      const has = store.getters['data/hasComments'](currentTab.value)
      if (!has) {
        await fetchComments(currentTab.value)
      }
    }

    onMounted(ensureComments)
    watch(currentTab, ensureComments)

    return {
      currentTab,
      comments,
    }
  },
})
</script>

<template>
  <v-tabs v-model="currentTab">
    <v-tab :value="1">ユーザーA</v-tab>
    <v-tab :value="2">ユーザーB</v-tab>
  </v-tabs>

  <v-list>
    <v-list-item v-for="comment in comments" :key="comment.id">
      {{ comment.text }}
    </v-list-item>
  </v-list>
</template>
```

---

## ✅ 補足と発展設計案

| 目的         | 方法                                                                  |
| ---------- | ------------------------------------------------------------------- |
| 初回のみ取得     | `store.getters['data/hasXxx'](tabId)` を使って判定                        |
| 共通ロジックにしたい | `useEnsureFetched(tabId, type, apiFn)` のような Composable 化            |
| 複数DTOに対応   | `setXxx` や `hasXxx` をDTOごとに定義して対応                                   |
| 更新/削除対応    | `updateComment(tabId, comment)`, `removeComment(tabId, id)` なども定義可能 |

---

## ✅ まとめ

| 項目                | 実装ポイント                                    |
| ----------------- | ----------------------------------------- |
| `defineComponent` | Composition APIで `setup()` を利用            |
| API呼び出し           | コンポーネント内から直接行い、storeに commit              |
| Vuex管理            | DTOごとに `tabId` をキーとする辞書形式で保持              |
| 初回判定              | getterで「すでに取得済みか」をチェック                    |
| 実行タイミング           | `onMounted` ＋ `watch(currentTab)` で切り替え対応 |

---

ご希望があれば：

* 複数DTOを共通ロジックで `useTabData<T>(...)` のように扱う composable 化
* API実装と連携した fetch のテスト構成
* 非同期取得中の loading 表示連動

なども対応できます！ご希望があればお知らせください。
