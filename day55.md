

# 追加Day10 完全まとめ（新規登録画面）

━━━━━━━━━━━━━━━━━━━

# Day10の目的

━━━━━━━━━━━━━━━━━━━

画面から新規登録できるようにする。

完成目標

```
/register

↓

チームを作成して登録

または

既存チームに参加

↓

登録API

↓

token保存

↓

/playersへ遷移
```

Day9ではログイン画面を作成した。

Day10では新規登録画面を作成した。

━━━━━━━━━━━━━━━━━━━

# 今回作成したファイル

━━━━━━━━━━━━━━━━━━━

```
src/pages/Register.tsx

src/api/register.ts
```

━━━━━━━━━━━━━━━━━━━

# App.tsxで行ったこと

━━━━━━━━━━━━━━━━━━━

追加

```
import Register from "./pages/Register"
```

Route追加

```
<Route
  path="/register"
  element={<Register />}
/>
```

Sidebar非表示条件追加

```
const isRegisterPage =
location.pathname === "/register"

const shouldShowSidebar =
!isHomePage &&
!isLoginPage &&
!isRegisterPage
```

結果

```
Home

↓

Sidebarなし

Login

↓

Sidebarなし

Register

↓

Sidebarなし

Players

↓

Sidebarあり
```

━━━━━━━━━━━━━━━━━━━

# Home画面変更

━━━━━━━━━━━━━━━━━━━

変更前

```
新規登録

↓

navigate("/")
```

変更後

```
新規登録

↓

navigate("/register")
```

つまり

```
Home

↓

新規登録ボタン

↓

Register画面
```

━━━━━━━━━━━━━━━━━━━

# register.ts

━━━━━━━━━━━━━━━━━━━

API責務分離を維持。

画面にfetchを書かない。

```
registerTeam()

↓

POST /register/team
```

```
registerJoin()

↓

POST /register/join
```

Register.tsxは

API関数を呼ぶだけ。

━━━━━━━━━━━━━━━━━━━

# Register画面構成

━━━━━━━━━━━━━━━━━━━

```
新規登録

[チーム作成]
[チーム参加]

↓

フォーム

↓

ホームへ戻る
```

1画面で

表示だけ切り替える。

━━━━━━━━━━━━━━━━━━━

# タブ切り替え

━━━━━━━━━━━━━━━━━━━

state

```
const [tab, setTab]
=
useState<"team"|"join">("team")
```

初期状態

```
team
```

つまり

画面を開くと

チーム作成フォームが表示される。

━━━━━━━━━━━━━━━━━━━

# タブの切り替え

━━━━━━━━━━━━━━━━━━━

ボタン

```
setTab("team")
```

または

```
setTab("join")
```

を実行する。

━━━━━━━━━━━━━━━━━━━

# Reactで画面が切り替わる仕組み

━━━━━━━━━━━━━━━━━━━

```
ボタン押す

↓

setTab()

↓

tabが変更

↓

React再描画

↓

条件判定

↓

表示切り替え
```

重要なのは

ボタンが表示を切り替えているのではない。

stateが変わることで

Reactが再描画している。

━━━━━━━━━━━━━━━━━━━

# 条件付きレンダリング

━━━━━━━━━━━━━━━━━━━

```
{tab === "team" && (...)}

{tab === "join" && (...)}
```

意味

```
tab=="team"

↓

teamフォーム表示
```

```
tab=="join"

↓

joinフォーム表示
```

Reactでは

状態によって

表示を切り替える。

━━━━━━━━━━━━━━━━━━━

# teamフォーム

━━━━━━━━━━━━━━━━━━━

入力項目

```
名前

メール

パスワード

チーム名
```

登録ボタン

```
チームを作成して登録
```

━━━━━━━━━━━━━━━━━━━

# joinフォーム

━━━━━━━━━━━━━━━━━━━

入力項目

```
名前

メール

パスワード

参加コード
```

登録ボタン

```
既存チームに参加
```

━━━━━━━━━━━━━━━━━━━

# useState

━━━━━━━━━━━━━━━━━━━

今回追加

```
name

email

password

teamName

inviteCode
```

役割

```
入力値を保存する箱
```

Reactでは

inputは

```
value

onChange
```

で

stateと同期する。

━━━━━━━━━━━━━━━━━━━

# handleRegisterTeam

━━━━━━━━━━━━━━━━━━━

処理

```
registerTeam()

↓

response

↓

response.json()

↓

token保存

↓

Players画面
```

コードの流れ

```
ボタン

↓

handleRegisterTeam()

↓

registerTeam()

↓

fetch

↓

Laravel

↓

token取得

↓

localStorage

↓

Players
```

━━━━━━━━━━━━━━━━━━━

# handleRegisterJoin

━━━━━━━━━━━━━━━━━━━

流れは同じ。

違うのは

```
registerJoin()

↓

inviteCode送信
```

だけ。

━━━━━━━━━━━━━━━━━━━

# localStorage

━━━━━━━━━━━━━━━━━━━

保存

```
localStorage.setItem(
"token",
data.token
)
```

役割

```
登録直後も

ログイン済み状態
```

━━━━━━━━━━━━━━━━━━━

# API責務分離

━━━━━━━━━━━━━━━━━━━

今回も徹底した。

画面

```
handleRegisterTeam()

↓

registerTeam()

↓

fetch

↓

Laravel
```

画面に

fetchを書かない。

━━━━━━━━━━━━━━━━━━━

# React設計の理解

━━━━━━━━━━━━━━━━━━━

画面

↓

state

↓

再描画

↓

条件判定

↓

表示

Reactは

状態(state)中心で画面を描画する。

━━━━━━━━━━━━━━━━━━━

# 今回理解した重要ポイント

━━━━━━━━━━━━━━━━━━━

① stateが変わるとReactは再描画する。

② 条件付きレンダリングで表示を切り替える。

③ タブはstateで管理する。

④ 1画面の中でフォームを切り替えられる。

⑤ API通信はapiフォルダへ分離する。

⑥ handle○○は処理をまとめた関数。

⑦ onClickはその関数を呼び出す入口。

━━━━━━━━━━━━━━━━━━━

# 完成した登録フロー

━━━━━━━━━━━━━━━━━━━

チーム作成

```
Register画面

↓

入力

↓

handleRegisterTeam

↓

registerTeam()

↓

POST /register/team

↓

Laravel

↓

token

↓

localStorage

↓

Players
```

チーム参加

```
Register画面

↓

入力

↓

handleRegisterJoin

↓

registerJoin()

↓

POST /register/join

↓

Laravel

↓

token

↓

localStorage

↓

Players
```

━━━━━━━━━━━━━━━━━━━

# Day10到達状態

━━━━━━━━━━━━━━━━━━━

✅ Register画面作成

✅ Homeから遷移

✅ Route追加

✅ Sidebar非表示

✅ タブ切り替え

✅ state管理

✅ 条件付きレンダリング

✅ teamフォーム

✅ joinフォーム

✅ registerTeam接続

✅ registerJoin接続

✅ token保存

✅ 登録後Players画面へ遷移

Day10完了。
