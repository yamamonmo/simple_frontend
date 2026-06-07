# React ToDo アプリ

React と TypeScript を使ったシンプルなToDoアプリです。
タスクの追加・完了トグル（クリックで完了/未完了を切り替え）ができます。

---

## 動かし方

```bash
# 依存パッケージをインストール（初回のみ）
npm install

# 開発サーバーを起動
npm run dev

# コードの問題をチェック（ESLint）
npm run lint

# 本番用ファイルをビルド（dist/ フォルダに出力）
npm run build
```

`npm run dev` 後、ブラウザで `http://localhost:5173` を開くとアプリが表示されます。

---

## ビルド後に動かすには

`npm run build` で `dist/` フォルダにファイルを出力しても、`dist/index.html` をブラウザで直接開くだけでは動きません。

### なぜ直接開けないのか

ブラウザでローカルファイルを開くと、URLが `http://...` ではなく `file:///...` になります。

```
# HTTPサーバー経由（動く）
http://localhost:4173/index.html

# ファイルを直接開いた場合（動かない）
file:///Users/yourname/react_todo/dist/index.html
```

`npm run build` が出力するHTMLは、JavaScriptを `<script type="module">` という形式で読み込みます。

```html
<script type="module" src="/assets/index-abc123.js"></script>
```

ブラウザはセキュリティ上の理由から、`file://` でアクセスしたページでは `type="module"` のスクリプトを実行しません（CORS制約）。また `/assets/...` のようなルート相対パスも `file://` では解決できません。

→ **ファイルをHTTP経由で配信するサーバーが必要です。**

### 確認方法

**方法1（一番簡単）: `npm run preview`**

Viteに組み込まれているプレビューサーバーを使います。`npm run build` の後に実行します。

```bash
npm run build    # dist/ にビルド
npm run preview  # dist/ をHTTPサーバーで配信
```

ブラウザで `http://localhost:4173` を開くと動作確認できます。`npm run dev` との違いは次のとおりです：

| コマンド | 何を動かすか | 用途 |
|---|---|---|
| `npm run dev` | ソースコード（`src/`）をその場で変換 | 開発中の編集・確認 |
| `npm run preview` | ビルド済みファイル（`dist/`）をそのまま配信 | 本番ビルドの最終確認 |

**方法2: `npx serve`**

Node.jsが入っていれば `serve` パッケージで簡易サーバーを立てることもできます：

```bash
npx serve dist
```

---

## ファイル構成

```
react_todo/
├── index.html                  # ブラウザが最初に読み込むHTMLファイル
├── package.json                # プロジェクト設定・依存パッケージの一覧
├── vite.config.ts              # ビルドツール Vite の設定
├── eslint.config.js            # コード品質チェックツール ESLint の設定
├── tsconfig.json               # TypeScript の設定
└── src/
    ├── main.tsx                # アプリの起動ポイント（エントリーポイント）
    ├── App.tsx                 # アプリ全体のルートコンポーネント
    ├── App.css                 # アプリのスタイル
    ├── index.css               # グローバルスタイル
    ├── types.ts                # TypeScriptの型定義
    └── components/
        ├── TodoInput.tsx       # タスク入力欄コンポーネント
        └── TodoItem.tsx        # タスク1件分の表示コンポーネント
```

---

## 各ファイルの役割

### `index.html` — HTMLの土台

ブラウザが最初に読み込むファイルです。中身はほぼ空で、`<div id="root">` という「Reactの描画先」だけを用意しています。

```html
<div id="root"></div>
```

このdivの中に、Reactが JavaScript でコンテンツを動的に書き込みます。

---

### `vite.config.ts` — Vite の設定ファイル

[Vite（ヴィート）](https://vitejs.dev/) は、開発サーバーの起動やビルドを行うツールです。

```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
})
```

- `plugins: [react()]` : ReactのJSX（`<App />` のような記法）をブラウザが解釈できるJavaScriptに変換するプラグインを有効にしています

**変換後のJSファイルはどこに保存されるか**

モードによって異なります。

| モード | 変換後JSの保存先 |
|---|---|
| `npm run dev`（開発中） | **保存なし（メモリ上のみ）** ブラウザのリクエストに応じてその都度変換・送信される |
| `npm run build`（本番ビルド） | `dist/assets/` フォルダに出力される |

本番ビルド後の `dist/` フォルダの中身はこのようになります：

```
dist/
├── index.html
└── assets/
    ├── index-abc123.js   ← 全 .tsx/.ts をまとめた変換済みJS
    └── index-def456.css  ← まとめられたCSS
```

複数の `.tsx` ファイルは1つの `.js` ファイルに**まとめて（バンドルして）**出力されます。ファイル名のハッシュ（`abc123` の部分）はブラウザのキャッシュ対策です。

**Viteを使う主なメリット**

| 機能 | 説明 |
|---|---|
| 開発サーバー | `npm run dev` で即座に起動し、ファイルを保存するたびにブラウザが自動更新される（HMR: Hot Module Replacement） |
| ビルド | `npm run build` で本番用に最適化された静的ファイルを `dist/` フォルダに出力する |
| TypeScript変換 | `.tsx` / `.ts` ファイルをブラウザが動くJavaScriptに自動変換する |

---

### `eslint.config.js` — ESLint の設定ファイル

[ESLint](https://eslint.org/) は、コードのミスや問題のある書き方を自動で検出するツールです。

```js
export default defineConfig([
  globalIgnores(['dist']),          // ビルド後の dist フォルダは検査しない
  {
    files: ['**/*.{js,jsx}'],       // 対象ファイルの拡張子
    extends: [
      js.configs.recommended,       // JavaScript の基本ルール
      reactHooks.configs.flat.recommended,  // React Hooks の正しい使い方を検査
      reactRefresh.configs.vite,    // 開発中のHMRが正しく動くか検査
    ],
    ...
  },
])
```

- `eslint-plugin-react-hooks` : `useState` などの Hooks を正しい順序・場所で使っているか検査します（Hooks は条件分岐の中で使ってはいけないなどのルールがあります）
- `eslint-plugin-react-refresh` : 開発中のホットリロードが正しく機能するかチェックします

`npm run lint` で手動チェックできます。エディタ（VS Code など）にESLint拡張を入れると、ファイルを編集中にリアルタイムで警告が表示されます。

---

### `src/main.tsx` — アプリの起動ポイント

Reactアプリを起動し、HTMLの `<div id="root">` に取り付ける（マウントする）ファイルです。

```tsx
createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>
)
```

- `createRoot` : HTMLの要素に React を取り付ける関数
- `StrictMode` : 開発中に潜在的な問題を検出して警告してくれるモード（本番では無効）
- `<App />` : アプリ本体のコンポーネント

**開発中と本番の切り分け**

`StrictMode` が有効かどうかは、実行するコマンドで自動的に決まります。

| コマンド | モード | StrictMode |
|---|---|---|
| `npm run dev` | `development` | 有効 |
| `npm run build` | `production` | 無効 |

Vite はコマンドに応じて `development` / `production` を自動でセットし、ビルド時に StrictMode 関連のコードを含めるかどうかを切り替えます。`production` ビルドでは StrictMode のコードがバンドルから取り除かれるため、警告は表示されず、ファイルサイズも小さくなります。

---

### `src/types.ts` — 型定義

アプリ全体で使う TypeScript の型（データの形）を定義しています。

```ts
export interface Todo {
  id: number;        // タスクを一意に識別する番号
  text: string;      // タスクの文章
  completed: boolean; // 完了したかどうか
}
```

`interface` は「このオブジェクトはこういうプロパティを持つ」という設計図です。型が合わないとコンパイルエラーになるため、バグを早期に発見できます。

---

### `src/App.tsx` — ルートコンポーネント

アプリ全体を束ねる親コンポーネントです。タスクの一覧データ (`todos`) をここで管理し、子コンポーネントに渡します。

```
App.tsx（データを管理）
├── <TodoInput />  ← タスクの追加関数 (handleAdd) を渡す
└── <TodoItem />   ← タスクデータ (todo) と完了トグル関数 (handleToggle) を渡す
```

**データの流れ（一方向）**

```
App (todos の状態を管理)
  │
  ├─── handleAdd ────→ TodoInput（入力してEnter/ボタンで呼ぶ）
  │
  └─── todo, handleToggle ──→ TodoItem（クリックで完了トグル）
```

Reactでは「データは上（親）で管理して、下（子）に渡す」のが基本です。

---

### `src/components/TodoInput.tsx` — 入力欄コンポーネント

テキスト入力欄と「追加」ボタンを担当するコンポーネントです。

```tsx
function TodoInput({ onAdd }: Props) {
  const [inputText, setInputText] = useState<string>("");
  ...
}
```

- `useState` : 入力欄の文字列をReactの「状態（state）」として管理します
- `value={inputText}` + `onChange={...}` の組み合わせで「制御されたコンポーネント」になり、JavaScriptが入力欄の値を常に把握している状態を作ります
- `onAdd` は親 (App.tsx) から渡された「タスクを追加する関数」です

---

### `src/components/TodoItem.tsx` — タスク1件のコンポーネント

タスク1件分の表示を担当するコンポーネントです。

```tsx
function TodoItem({ todo, onToggle }: Props) {
  return (
    <li
      className={todo.completed ? "completed" : ""}
      onClick={() => onToggle(todo.id)}
    >
      {todo.text}
    </li>
  );
}
```

- `todo.completed ? "completed" : ""` : 完了状態なら CSS クラス `completed` を付与（打ち消し線スタイルが適用される）
- クリックすると `onToggle(todo.id)` が呼ばれ、親 (App.tsx) の完了状態が更新される

---

### `src/App.css` — スタイルシート

アプリの見た目を定義しています。基本的なCSSの書き方はHTMLの場合と同じです。

Reactでは JSX の中で `class` の代わりに `className` を使う点だけ異なります。

```tsx
// ✗ HTMLの書き方
<li class="completed">

// ✓ JSXの書き方
<li className="completed">
```

---

## お作法（React / TypeScript のコーディング規約）

### `export default` とは

各コンポーネントファイルの末尾にある `export default` は、「このファイルの主役をほかのファイルから使えるようにする」宣言です。

```ts
// TodoItem.tsx の末尾
export default TodoItem;
```

対応する `import` はこうなります：

```ts
// App.tsx での読み込み
import TodoItem from "./components/TodoItem";
//     ↑ 名前は自由につけられる（慣習的に同じ名前にする）
```

**`export default`（デフォルトエクスポート）と名前付きエクスポートの違い**

| 種類 | 書き方（export側） | 書き方（import側） | 特徴 |
|---|---|---|---|
| デフォルト | `export default TodoItem` | `import TodoItem from "./TodoItem"` | 1ファイルに1つだけ。import時に名前を自由に変えられる |
| 名前付き | `export interface Todo` | `import { Todo } from "./types"` | 1ファイルに複数OK。import時は `{}` で囲み、同じ名前を使う |

このプロジェクトでは、コンポーネントは `export default`、型定義（`interface`）は名前付きエクスポートを使っています。

---

### 拡張子 `.js` `.jsx` `.ts` `.tsx` の違い

4種類の拡張子は、**TypeScriptを使うか** と **JSXを書くか** の組み合わせで決まります。

| 拡張子 | TypeScript | JSX | 用途 |
|---|---|---|---|
| `.js` | なし | なし | 通常のJavaScript |
| `.jsx` | なし | あり | JSXを使うJavaScript |
| `.ts` | あり | なし | JSXを使わないTypeScript |
| `.tsx` | あり | あり | JSXを使うTypeScript |

**JSX（ジェイエスエックス）** とは、JavaScriptの中にHTMLのような記法を書ける構文です。

```tsx
// JSXあり（.tsx / .jsx）
function App() {
  return <div className="container">Hello</div>;
//        ↑ HTMLのような記法がJSX
}

// JSXなし（.ts / .js）
// types.ts や eslint.config.js はJSXを使わないのでこちら
export interface Todo {
  id: number;
  text: string;
}
```

このプロジェクトでの使い分けは次のとおりです：

| ファイル | 拡張子 | 理由 |
|---|---|---|
| `App.tsx` `TodoItem.tsx` `TodoInput.tsx` `main.tsx` | `.tsx` | TypeScript + JSXを使うコンポーネント |
| `types.ts` | `.ts` | 型定義のみ。JSXは不要 |
| `eslint.config.js` | `.js` | 設定ファイル。TypeScriptもJSXも不要 |
| `vite.config.ts` | `.ts` | 設定ファイル。TypeScriptを使うがJSXは不要 |

---

### 命名規則

**コンポーネント名・ファイル名は大文字始まり（PascalCase）**

```
✓ App.tsx / TodoItem.tsx / TodoInput.tsx
✗ app.tsx / todoItem.tsx / todoInput.tsx
```

Reactはコンポーネントを `<TodoItem />` のように大文字始まりで区別しています。小文字始まりだと `<todoItem />` は HTML タグと見なされ、コンポーネントとして認識されません。

**イベントハンドラの命名**

| 役割 | 命名パターン | 例 |
|---|---|---|
| コンポーネント内の処理 | `handle〇〇` | `handleAdd`, `handleToggle`, `handleKeyDown` |
| 親から渡されるprops | `on〇〇` | `onAdd`, `onToggle` |

```tsx
// 親（App.tsx）: on〇〇 という名前で渡す
<TodoInput onAdd={handleAdd} />

// 子（TodoInput.tsx）: on〇〇 という名前で受け取る
function TodoInput({ onAdd }: Props) { ... }
```

この慣習はHTML標準（`onClick`, `onChange` など）に合わせたものです。

---

### ファイルの書き順

Reactコンポーネントファイルは、慣習的にこの順番で書きます。

```tsx
// 1. import（外部ライブラリ → 内部ファイルの順）
import { useState } from "react";
import { Todo } from "./types";

// 2. 型定義（interface）
interface Props { ... }

// 3. コンポーネント本体
function TodoItem({ todo, onToggle }: Props) {
  // 3a. state の宣言
  const [inputText, setInputText] = useState("");

  // 3b. イベントハンドラ
  const handleAdd = () => { ... };

  // 3c. return（JSX）
  return ( ... );
}

// 4. export（ファイルの末尾）
export default TodoItem;
```

---

### JSX内のコメントの書き方

JSXの中（`return` の `()` の中）では、通常の `//` コメントが使えません。`{/* */}` で囲む必要があります。

```tsx
return (
  <div>
    {/* ✓ JSX内のコメントはこう書く */}
    // ✗ これはコメントにならず文字列として表示されてしまう
    <p>テキスト</p>
  </div>
);
```

---

### `return` の中は1つの要素で包む

JSXの `return` では、複数の要素をそのまま並べることはできません。1つの要素で包む必要があります。

```tsx
// ✗ NG: 複数の要素がそのまま並んでいる
return (
  <h1>タイトル</h1>
  <p>本文</p>
);

// ✓ OK: div や Fragment（<>）で包む
return (
  <div>
    <h1>タイトル</h1>
    <p>本文</p>
  </div>
);

// ✓ OK: Fragment（余分なHTMLタグを増やしたくないとき）
return (
  <>
    <h1>タイトル</h1>
    <p>本文</p>
  </>
);
```

---

### スプレッド構文（`...`）

`...` はJavaScriptの**スプレッド構文（spread syntax）**です。配列やオブジェクトの要素を「展開する」記法で、このプロジェクトでは App.tsx の2か所で使われています。

**配列のスプレッド — `[...todos, newTodo]`**

```ts
// todos が [{id:1, text:"買い物"}, {id:2, text:"掃除"}] のとき
setTodos([...todos, newTodo]);
// ↓ 展開すると
// [{id:1, text:"買い物"}, {id:2, text:"掃除"}, newTodo]
```

`push()` を使わずスプレッドを使う理由はReactのルールにあります：

```ts
// ✗ NG: 元の配列を直接変更してしまう
todos.push(newTodo);
setTodos(todos);

// ✓ OK: 元の配列をコピーした新しい配列を渡す
setTodos([...todos, newTodo]);
```

Reactは「新しい配列かどうか」で再描画が必要かを判断します。元の配列を直接変更（ミューテート）すると、Reactが変化を検知できず画面が更新されません。

**オブジェクトのスプレッド — `{ ...todo, completed: !todo.completed }`**

```ts
todos.map((todo) =>
  todo.id === id ? { ...todo, completed: !todo.completed } : todo
)
```

`{ ...todo, completed: !todo.completed }` は「`todo` の全プロパティをコピーした上で `completed` だけ上書きした新しいオブジェクト」を作ります：

```ts
// todo が { id:1, text:"買い物", completed: false } のとき
{ ...todo, completed: true }
// → { id:1, text:"買い物", completed: true }
//            ↑ コピー               ↑ 上書き
```

配列と同じ理由で、オブジェクトも直接変更せず新しいオブジェクトを作って渡します。

---

## 学べる主な概念

### React

| 概念 | どのファイルで学べるか |
|---|---|
| コンポーネントの分割 | `TodoItem.tsx`, `TodoInput.tsx` |
| props（親から子へのデータ渡し） | `TodoItem.tsx`, `TodoInput.tsx` |
| useState（状態管理） | `App.tsx`, `TodoInput.tsx` |
| リストの描画と `key` | `App.tsx` |
| 制御されたコンポーネント | `TodoInput.tsx` |

### TypeScript

| 概念 | どのファイルで学べるか |
|---|---|
| `interface` でオブジェクトの型を定義 | `types.ts` |
| propsに型をつける | `TodoItem.tsx`, `TodoInput.tsx` |
| イベントの型（`React.KeyboardEvent` など） | `TodoInput.tsx` |
| `useState` にジェネリクスで型を渡す | `App.tsx` |

### JavaScript

| 概念 | どのファイルで学べるか |
|---|---|
| スプレッド構文（`...`）で配列をコピー・追加 | `App.tsx` |
| スプレッド構文（`...`）でオブジェクトをコピー・上書き | `App.tsx` |

---

## 使用技術

- [React 19](https://react.dev/) — UIを構築するJavaScriptライブラリ
- [TypeScript](https://www.typescriptlang.org/) — 型があるJavaScript
- [Vite](https://vitejs.dev/) — 高速なフロントエンドビルドツール
