

# 追加Day8：未ログインホーム画面 完全まとめ

━━━━━━━━━━━━━━━━━━━
現在のロードマップ位置
━━━━━━━━━━━━━━━━━━━

完了済み

```txt
追加Day1
登録設計・role設計

追加Day2
teams に invite_code追加

追加Day3
users に role追加

追加Day4
users に profile_image追加

追加Day5
新規登録API① チーム作成

追加Day6
新規登録API② チーム参加

追加Day7
GET /user 強化

追加Day8
未ログインホーム画面 ← 完了
```

次回

```txt
追加Day9
ログイン画面
```

━━━━━━━━━━━━━━━━━━━
Day8の目的
━━━━━━━━━━━━━━━━━━━

未ログインユーザー用の入口画面を作る。

完成イメージ

```txt
未ログイン

/
↓
Home画面
↓
ログイン
新規登録
```

```txt
ログイン済み

/
↓
自動で /players
```

━━━━━━━━━━━━━━━━━━━
今回作成したファイル
━━━━━━━━━━━━━━━━━━━

新規作成

```txt
src/pages/Home.tsx
```

内容

```tsx
function Home() {
  return (
    <div>
      <h1>Football Team Manager</h1>

      <p>
        サッカーチームの選手管理アプリです。
      </p>

      <button>
        ログイン
      </button>

      <button>
        新規登録
      </button>
    </div>
  )
}

export default Home
```

━━━━━━━━━━━━━━━━━━━
学んだこと①
Home.tsxはなぜpagesなのか
━━━━━━━━━━━━━━━━━━━

Homeは

```txt
URLで表示される画面
```

だから。

例

```txt
/
↓
Home.tsx
```

```txt
/players
↓
Players.tsx
```

画面は

```txt
pages
```

へ置く。

━━━━━━━━━━━━━━━━━━━
学んだこと②
Sidebarはなぜpagesではないのか
━━━━━━━━━━━━━━━━━━━

Sidebarは

```txt
URLで表示される画面
```

ではない。

共通部品。

例

```txt
App
│
├─ Sidebar
│
└─ Players
```

Sidebarは

```txt
どこへ移動するか
```

を表示する。

━━━━━━━━━━━━━━━━━━━
学んだこと③
Routeの役割
━━━━━━━━━━━━━━━━━━━

App.tsxのRouteは

```txt
URLと画面の対応表
```

例

```tsx
<Route
  path="/players"
  element={<Players />}
/>
```

意味

```txt
/players

↓

Players.tsx表示
```

━━━━━━━━━━━━━━━━━━━
学んだこと④
Sidebar → Route → Page
━━━━━━━━━━━━━━━━━━━

流れ

```txt
Sidebar

↓

/playersへ移動

↓

Route一致

↓

Players.tsx表示
```

役割

```txt
Sidebar
↓
どこへ移動するか

Route
↓
そのURLなら何を表示するか

Players.tsx
↓
実際の画面
```

━━━━━━━━━━━━━━━━━━━
App.tsx変更①
Home読み込み
━━━━━━━━━━━━━━━━━━━

追加

```tsx
import Home from "./pages/Home"
```

意味

```txt
Home.tsxを使えるようにする
```

━━━━━━━━━━━━━━━━━━━
App.tsx変更②
トップページ追加
━━━━━━━━━━━━━━━━━━━

変更前

```tsx
<Route path="/home" element={<div>Home</div>} />
```

変更後

```tsx
<Route path="/" element={<Home />} />
```

意味

```txt
/

↓

Home.tsx表示
```

━━━━━━━━━━━━━━━━━━━
学んだこと⑤
location.pathname
━━━━━━━━━━━━━━━━━━━

意味

```txt
現在のURL
```

例

```txt
http://localhost:5173/
```

↓

```tsx
location.pathname
```

↓

```txt
"/"
```

━━━━━━━━━━━━━━━━━━━

例

```txt
http://localhost:5173/players
```

↓

```tsx
location.pathname
```

↓

```txt
"/players"
```

━━━━━━━━━━━━━━━━━━━
学んだこと⑥
isHomePage
━━━━━━━━━━━━━━━━━━━

コード

```tsx
const isHomePage =
  location.pathname === "/"
```

意味

```txt
今いる場所がHomeか
```

結果

```txt
/
↓
true

/players
↓
false
```

━━━━━━━━━━━━━━━━━━━
学んだこと⑦
条件付き表示
━━━━━━━━━━━━━━━━━━━

コード

```tsx
{!isHomePage && <Sidebar />}
```

意味

```txt
Homeじゃないなら
Sidebar表示
```

━━━━━━━━━━━━━━━━━━━

例

```txt
/
```

↓

```txt
isHomePage = true
```

↓

```txt
!isHomePage = false
```

↓

```txt
Sidebar表示しない
```

━━━━━━━━━━━━━━━━━━━

例

```txt
/players
```

↓

```txt
isHomePage = false
```

↓

```txt
!isHomePage = true
```

↓

```txt
Sidebar表示
```

━━━━━━━━━━━━━━━━━━━
学んだこと⑧
useNavigate
━━━━━━━━━━━━━━━━━━━

追加

```tsx
const navigate = useNavigate()
```

意味

```txt
コードでページ移動する
```

例

```tsx
navigate("/players")
```

意味

```txt
/playersへ移動
```

━━━━━━━━━━━━━━━━━━━
学んだこと⑨
useEffect
━━━━━━━━━━━━━━━━━━━

基本形

```tsx
useEffect(() => {

}, [])
```

意味

```txt
画面表示時に1回実行
```

━━━━━━━━━━━━━━━━━━━

基本形

```tsx
useEffect(() => {

}, [A])
```

意味

```txt
画面表示時

+

Aが変わるたび実行
```

━━━━━━━━━━━━━━━━━━━
学んだこと⑩
Dependency Array
━━━━━━━━━━━━━━━━━━━

今回

```tsx
[location.pathname, navigate]
```

意味

```txt
location.pathnameが変わったら
useEffect再実行
```

例

```txt
/

↓

/players
```

↓

```txt
pathname変更
```

↓

```txt
useEffect再実行
```

━━━━━━━━━━━━━━━━━━━
ログイン済み自動遷移
━━━━━━━━━━━━━━━━━━━

追加コード

```tsx
useEffect(() => {
  const token = localStorage.getItem("token")

  if (token && location.pathname === "/") {
    navigate("/players")
  }
}, [location.pathname, navigate])
```

意味

```txt
token取得

↓

token存在

かつ

現在URLが /

↓

/playersへ移動
```

━━━━━━━━━━━━━━━━━━━
完成したApp.tsx重要部分
━━━━━━━━━━━━━━━━━━━

```tsx
import { useEffect } from "react"
import {
  Routes,
  Route,
  useLocation,
  useNavigate
} from "react-router-dom"

import Home from "./pages/Home"
```

━━━━━━━━━━━━━━━━━━━

```tsx
const location = useLocation()
const navigate = useNavigate()

const isHomePage =
  location.pathname === "/"
```

━━━━━━━━━━━━━━━━━━━

```tsx
useEffect(() => {
  const token = localStorage.getItem("token")

  if (token && location.pathname === "/") {
    navigate("/players")
  }
}, [location.pathname, navigate])
```

━━━━━━━━━━━━━━━━━━━

```tsx
{!isHomePage && <Sidebar />}
```

━━━━━━━━━━━━━━━━━━━

```tsx
<Route
  path="/"
  element={<Home />}
/>
```

━━━━━━━━━━━━━━━━━━━
Day8完了時の到達状態
━━━━━━━━━━━━━━━━━━━

```txt
Home.tsx作成済み

/ でHome表示

HomeではSidebar非表示

ログイン済みなら
/playersへ自動遷移

未ログインユーザー用入口完成
```

━━━━━━━━━━━━━━━━━━━
次回 Day9
━━━━━━━━━━━━━━━━━━━

```txt
ログイン画面

/login作成

ログインAPI接続

token保存

ログイン後
/playersへ遷移
```
