# Day11 完全まとめ（AuthContext・認証状態管理）

━━━━━━━━━━━━━━━━━━━
■ Day11の目的
━━━━━━━━━━━━━━━━━━━

今までは

ログイン
↓
token取得
↓
localStorage保存
↓
Players画面

だけだった。

つまりブラウザ(localStorage)にしかtokenが存在せず、
React側は

・ログインしているか
・誰がログインしているか

を管理していなかった。

そこでDay11では

AuthContext

を作成し、

・token
・user

をReact全体で共有できるようにした。

━━━━━━━━━━━━━━━━━━━
■ Day11で実現したこと
━━━━━━━━━━━━━━━━━━━

AuthContext作成
↓
token管理
↓
GET /user
↓
user取得
↓
React全体へ共有
↓
401時自動ログアウト

━━━━━━━━━━━━━━━━━━━
■ 変更したファイル
━━━━━━━━━━━━━━━━━━━

src/context/AuthContext.tsx
（新規作成）

src/api/auth.ts
（getUser追加）

src/pages/Login.tsx
（setToken追加）

src/pages/Register.tsx
（setToken追加）

src/main.tsx
（AuthProvider追加）

━━━━━━━━━━━━━━━━━━━
■ AuthContextとは
━━━━━━━━━━━━━━━━━━━

React全体で

・token
・user
・setToken

を共有する場所。

つまり

「認証状態管理センター」

━━━━━━━━━━━━━━━━━━━
■ User型
━━━━━━━━━━━━━━━━━━━

type User = {
  id
  name
  email
  teamId
  teamName
  role
  profileImage
}

意味

GET /user

↓

Laravelから返るユーザー情報

━━━━━━━━━━━━━━━━━━━
■ AuthContextType
━━━━━━━━━━━━━━━━━━━

type AuthContextType = {
  user
  token
  setToken
}

意味

Contextが共有する設計図。

user
→ 誰がログインしているか

token
→ ログインしているか

setToken
→ tokenを書き換える関数

━━━━━━━━━━━━━━━━━━━
■ createContext
━━━━━━━━━━━━━━━━━━━

createContext()

↓

React全体で共有するContext作成

初期値

user = null

token = null

setToken = 空関数

空関数を書く理由

createContextは
AuthContextTypeと同じ形の初期値が必要だから。

実際には

Provider

で本物へ置き換わる。

━━━━━━━━━━━━━━━━━━━
■ AuthProvider
━━━━━━━━━━━━━━━━━━━

役割

token

user

setToken

を

App全体へ配る。

━━━━━━━━━━━━━━━━━━━
■ token State
━━━━━━━━━━━━━━━━━━━

const [token, setToken] =
useState(
localStorage.getItem("token")
)

意味

アプリ起動時

localStorage
↓

保存済みtoken取得
↓

React State(token)

へコピーする。

重要

localStorage.getItem()

は

初期値

として1回だけ使う。

その後は

React State(token)

を使う。

━━━━━━━━━━━━━━━━━━━
■ user State
━━━━━━━━━━━━━━━━━━━

const [user, setUser]
=
useState(null)

意味

現在ログイン中のユーザー保持。

初期値

null

━━━━━━━━━━━━━━━━━━━
■ useEffect
━━━━━━━━━━━━━━━━━━━

[token]

なので

・初回表示

・token変更時

に実行される。

━━━━━━━━━━━━━━━━━━━
■ useEffectの流れ
━━━━━━━━━━━━━━━━━━━

① tokenある？

if(!token)

↓

return

② tokenある

↓

getUser(token)

↓

Authorization Bearer

↓

Laravel

↓

そのtoken誰？

↓

ユーザー情報返却

③

response.json()

↓

JSON取得

④

setUser(data)

↓

user更新

↓

AuthProvider再レンダリング

↓

Provider更新

↓

App全体へ共有

━━━━━━━━━━━━━━━━━━━
■ Provider
━━━━━━━━━━━━━━━━━━━

<AuthContext.Provider

value={{
user,
token,
setToken
}}

>

意味

AuthProviderが持つ

user

token

setToken

を

App全体へ配る。

━━━━━━━━━━━━━━━━━━━
■ useContext
━━━━━━━━━━━━━━━━━━━

必要な情報だけ取得する。

ログインしているか

↓

const { token }

誰がログインしているか

↓

const { user }

token変更したい

↓

const { setToken }

━━━━━━━━━━━━━━━━━━━
■ Contextを使うタイミング
━━━━━━━━━━━━━━━━━━━

必要な画面だけ取得する。

例

Login

↓

setToken

Register

↓

setToken

Sidebar

↓

user

Players

↓

userを使いたくなった時だけ取得

全部取得する必要はない。

━━━━━━━━━━━━━━━━━━━
■ Login変更
━━━━━━━━━━━━━━━━━━━

追加

const { setToken }
=
useContext(AuthContext)

ログイン成功時

localStorage.setItem(
"token",
data.token
)

setToken(data.token)

━━━━━━━━━━━━━━━━━━━
■ Register変更
━━━━━━━━━━━━━━━━━━━

Team作成

localStorage.setItem()

↓

setToken(data.token)

Team参加

localStorage.setItem()

↓

setToken(data.token)

Loginと同じ認証フローへ統一。

━━━━━━━━━━━━━━━━━━━
■ localStorageとReact State
━━━━━━━━━━━━━━━━━━━

localStorage

役割

ブラウザへ長期間保存

React State

役割

Reactが現在使うtoken

ログイン時

Laravel

↓

token取得

↓

① localStorage保存

↓

② React State更新

同じtokenを

保存用

+

React利用用

として管理する。

━━━━━━━━━━━━━━━━━━━
■ なぜReact Stateが必要か
━━━━━━━━━━━━━━━━━━━

毎回

localStorage.getItem()

を書かなくて済む。

API通信では

React State(token)

を使う。

━━━━━━━━━━━━━━━━━━━
■ 初期値にlocalStorage.getItemを書く理由
━━━━━━━━━━━━━━━━━━━

アプリ起動時

↓

以前保存したtoken取得

↓

Reactへ復元

ログイン時には使わない。

ログイン時は

setToken()

で更新する。

━━━━━━━━━━━━━━━━━━━
■ 401対応
━━━━━━━━━━━━━━━━━━━

GET /user

↓

401

↓

localStorage.removeItem

↓

setToken(null)

↓

setUser(null)

↓

navigate("/login")

↓

return

意味

認証切れ

↓

token削除

↓

user削除

↓

未ログイン状態へ戻す

━━━━━━━━━━━━━━━━━━━
■ tokenとuserの関係
━━━━━━━━━━━━━━━━━━━

token

↓

認証

↓

user取得

tokenが無効なのに

userだけ残る

↓

矛盾

だから

token = null

user = null

へ戻す。

━━━━━━━━━━━━━━━━━━━
■ Day11で理解した重要概念
━━━━━━━━━━━━━━━━━━━

AuthContext
→ 認証情報共有

Provider
→ App全体へ配る

useContext
→ 必要な情報だけ取得

localStorage
→ 長期保存

React State
→ 現在利用する値

useEffect
→ token変更検知

↓

GET /user

setUser(data)

↓

user更新

↓

AuthProvider再レンダリング

↓

Provider更新

↓

useContext利用画面更新

━━━━━━━━━━━━━━━━━━━
■ Day11最終フロー
━━━━━━━━━━━━━━━━━━━

【アプリ起動】

localStorage.getItem("token")

↓

React State(token)

↓

useEffect

↓

GET /user

↓

setUser(data)

↓

Provider更新

↓

App全体でuser共有

━━━━━━━━━━━━━━━━━━━

【ログイン】

Laravel

↓

token取得

↓

localStorage保存

↓

setToken(data.token)

↓

token変更

↓

useEffect発火

↓

GET /user

↓

setUser(data)

↓

Provider更新

↓

App全体更新

━━━━━━━━━━━━━━━━━━━

【401】

GET /user

↓

401

↓

localStorage.removeItem

↓

setToken(null)

↓

setUser(null)

↓

navigate("/login")

↓

未ログイン状態へ戻す