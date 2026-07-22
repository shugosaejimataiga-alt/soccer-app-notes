

# 追加Day12：Sidebar整理 完全まとめ

## 今日の目的

追加Day12では、ログイン後のSidebarを整理した。

やったことは主に3つ。

1. SidebarにLogoutを追加
2. ownerだけTeamリンクを表示
3. ログアウト後に未ログインHome「/」へ移動

---

# 今回変更したファイル

## 変更したファイル

```txt
src/Sidebar.tsx
src/context/AuthContext.tsx
src/api/auth.ts
```

---

# 追加Day12で使った前提

追加Day11で、AuthContextを作成済み。

AuthContextには以下が入っている。

```ts
user
token
setToken
```

意味は以下。

```txt
user
↓
現在ログインしているユーザー情報

token
↓
ログインしている証明

setToken
↓
React側のtoken状態を変更する関数
```

---

# Contextとは何か

本来、別のコンポーネントで値を使う場合は、propsで渡す。

例：

```tsx
<Sidebar user={user} setToken={setToken} />
```

しかし、アプリ全体で毎回propsを渡すのは面倒。

そこでContextを使う。

Contextは、

```txt
Providerで囲まれた範囲のコンポーネントが、
必要な値を直接取り出せる仕組み
```

今回の場合、`main.tsx` で以下のように囲んでいる。

```tsx
<AuthProvider>
  <App />
</AuthProvider>
```

なので、`App` の中で表示される `Sidebar` や `Login` や `Register` などで、AuthContextを使える。

---

# useContext(AuthContext)とは何か

```tsx
const { user, token, setToken } = useContext(AuthContext)
```

これは、

```txt
AuthContextという認証情報の箱から、
user / token / setToken を取り出す
```

という意味。

今回Sidebarで必要だった理由は以下。

```txt
user
↓
ownerかmemberか確認するため

token
↓
logout APIに送るため

setToken
↓
ログアウト時にReact側のtokenをnullにするため
```

---

# userとは何か

`user` はログイン中のユーザー情報。

GET /user によってLaravelから返ってくる情報。

今回のUser型は以下。

```ts
type User = {
  id: number
  name: string
  email: string
  teamId: number
  teamName: string
  role: "owner" | "member"
  profileImage: string | null
}
```

この中の `role` を使って、ownerかmemberかを判断する。

---

# roleはどこにあるのか

`role` は `users` テーブルに追加した。

```txt
users table
- id
- name
- email
- password
- team_id
- role
- profile_image
```

つまり、

```tsx
user.role
```

を見ることで、そのログインユーザーがownerなのかmemberなのか判断できる。

---

# tokenとは何か

tokenはログイン中であることを証明する文字列。

フロント側では主に2か所で持っている。

```txt
localStorage
↓
ブラウザに保存しているtoken

React State
↓
今Reactが画面表示で使っているtoken
```

Day11で以下のように管理していた。

```tsx
const [token, setToken] = useState<string | null>(
  localStorage.getItem("token")
)
```

意味は以下。

```txt
アプリ起動時
↓
localStorageからtokenを取得
↓
React Stateの初期値にする
```

---

# localStorage.removeItem("token") の意味

```tsx
localStorage.removeItem("token")
```

これは、ブラウザに保存しているtokenを削除する処理。

意味は以下。

```txt
localStorageのtokenを消す
↓
次回アプリ起動時にtokenが復元されない
↓
ログインからやり直しになる
```

---

# setToken(null) の意味

```tsx
setToken(null)
```

これは、React側のtoken状態を空にする処理。

意味は以下。

```txt
React Stateのtokenをnullにする
↓
React上では未ログイン状態になる
```

`localStorage.removeItem("token")` だけでは、React Stateにはtokenが残っている可能性がある。

だからログアウト時は両方必要。

```tsx
localStorage.removeItem("token")
setToken(null)
```

意味は以下。

```txt
localStorage.removeItem("token")
↓
次回起動時のために保存tokenを消す

setToken(null)
↓
今の画面上のログイン状態も消す
```

---

# Authorization: Bearer token とは何か

logout APIでは、以下を送る。

```ts
"Authorization": `Bearer ${token}`
```

意味は以下。

```txt
Authorization
↓
認証情報を送るためのHTTPヘッダー

Bearer
↓
このtokenを持っている人として認証してください、という意味

${token}
↓
実際のログインtoken
```

つまりLaravelには以下のように送られる。

```txt
Authorization: Bearer 実際のtoken
```

Laravel側はこれを見て、

```txt
このtokenはpersonal_access_tokensに存在するか？
↓
存在するならログイン中ユーザーとして扱う
↓
存在しないなら401 Unauthorized
```

と判断する。

認証が必要なAPIでは、このAuthorizationヘッダーが必要。

---

# logout APIの意味

logout APIでは、Laravel側に現在のtokenを送る。

```ts
logout(token)
```

Laravel側では、そのtokenをもとに、

```txt
personal_access_tokens テーブルから現在のtokenを削除
```

する。

つまりバックエンド側では、

```txt
このtokenはもう使えません
```

という状態にする。

---

# auth.ts に追加した logout 関数

`src/api/auth.ts` に以下を追加した。

```ts
export const logout = async (token: string) => {
  return fetch("http://localhost:8000/api/logout", {
    method: "POST",
    headers: {
      "Accept": "application/json",
      "Authorization": `Bearer ${token}`,
    },
  })
}
```

意味は以下。

```txt
tokenを受け取る
↓
Authorization: Bearer token としてLaravelへ送る
↓
POST /api/logout を呼ぶ
↓
Laravel側でtokenを無効化する
```

---

# Sidebar.tsx の完成コード

```tsx
import { useContext } from "react"
import { Link, useNavigate } from "react-router-dom"
import { AuthContext } from "./context/AuthContext"
import { logout } from "./api/auth"

export default function Sidebar() {

  const { user, token, setToken } = useContext(AuthContext)
  const navigate = useNavigate()

  const handleLogout = async () => {
    if (!token) {
      return
    }

    const response = await logout(token)

    if (response.ok) {
      localStorage.removeItem("token")
      setToken(null)
      navigate("/")
    }
  }

  return (
    <div className="p-4 bg-gray-200 min-h-screen w-64">
      <ul className="flex flex-col gap-2">
        <li><Link to="/home">Home</Link></li>
        <li><Link to="/matches">Matches</Link></li>
        <li><Link to="/players">Players</Link></li>
        <li><Link to="/conditions">Conditions</Link></li>

        {user?.role === "owner" && (
          <li><Link to="/team">Team</Link></li>
        )}

        <li>
          <button onClick={handleLogout}>
            Logout
          </button>
        </li>

      </ul>
    </div>
  )
}
```

---

# Sidebar.tsx のimport部分

```tsx
import { useContext } from "react"
import { Link, useNavigate } from "react-router-dom"
import { AuthContext } from "./context/AuthContext"
import { logout } from "./api/auth"
```

意味は以下。

```txt
useContext
↓
AuthContextから値を取り出すため

Link
↓
画面遷移リンクを作るため

useNavigate
↓
処理の中で "/" へ移動するため

AuthContext
↓
user / token / setToken を使うため

logout
↓
Laravelのlogout APIを呼ぶため
```

---

# Sidebar内でAuthContextを使う

```tsx
const { user, token, setToken } = useContext(AuthContext)
```

意味は以下。

```txt
AuthContextから
user / token / setToken を取り出す
```

使い道は以下。

```txt
user
↓
ownerかどうか判定する

token
↓
logout APIに送る

setToken
↓
ログアウト後にReact側のtokenをnullにする
```

---

# useNavigateを使う

```tsx
const navigate = useNavigate()
```

意味は以下。

```txt
処理の中で画面遷移できるようにする
```

今回はログアウト成功後に以下を実行する。

```tsx
navigate("/")
```

意味は以下。

```txt
未ログインHome画面へ移動する
```

---

# ownerだけTeamを表示する処理

```tsx
{user?.role === "owner" && (
  <li><Link to="/team">Team</Link></li>
)}
```

意味は以下。

```txt
user が存在する
+
user.role が owner
↓
Teamリンクを表示
```

memberの場合は表示されない。

---

# user?.role の ?. とは何か

```tsx
user?.role
```

これは、userがnullのときにエラーにならないための書き方。

通常、

```tsx
user.role
```

と書くと、userがnullの時にエラーになる。

しかし、

```tsx
user?.role
```

なら、

```txt
userがある
↓
roleを見る

userがnull
↓
undefinedとして扱い、エラーにしない
```

となる。

---

# handleLogout の流れ

```tsx
const handleLogout = async () => {
  if (!token) {
    return
  }

  const response = await logout(token)

  if (response.ok) {
    localStorage.removeItem("token")
    setToken(null)
    navigate("/")
  }
}
```

流れは以下。

```txt
Logoutボタンを押す
↓
handleLogout が実行される
↓
token があるか確認
↓
token がなければ終了
↓
token があれば logout(token) を実行
↓
Laravel側でtoken削除
↓
response.ok なら成功
↓
localStorageのtoken削除
↓
React Stateのtokenをnull
↓
"/"へ移動
```

---

# if (!token) return の意味

```tsx
if (!token) {
  return
}
```

意味は以下。

```txt
tokenがない
↓
ログインしていない
↓
logout APIを呼ぶ必要がない
↓
処理終了
```

最初に間違えて、

```tsx
if (token) {
  return
}
```

と書いてしまった。

これは、

```txt
tokenがあるなら処理終了
```

という意味になってしまう。

ログアウトしたいのはtokenがある時なので、正しくは以下。

```tsx
if (!token) {
  return
}
```

---

# response.ok の意味

```tsx
if (response.ok) {
```

意味は以下。

```txt
logout APIが成功した場合だけ
↓
フロント側のログアウト処理をする
```

今回、`try...finally` は使わなかった。

理由は、バックエンド側のログアウトが失敗したのに、フロントだけログアウトすると、

```txt
画面上はログアウトした
でもDB側のtokenは残っている
```

というズレが起きるから。

これは以前のPlayersの時と同じ考え方。

```txt
画面だけ変える
でもDBは変わっていない
```

という状態を避けた。

---

# Logoutボタン

```tsx
<li>
  <button onClick={handleLogout}>
    Logout
  </button>
</li>
```

意味は以下。

```txt
Logoutボタンをクリック
↓
handleLogoutを実行
```

`<li>` は `<ul>` の中に入れる。

最初は `<ul>` の外に書いていたが、正しくは以下。

```tsx
<ul>
  ...
  <li>
    <button onClick={handleLogout}>
      Logout
    </button>
  </li>
</ul>
```

---

# AuthContext.tsx で追加修正したこと

memberでログインした時、一瞬だけTeamが表示された。

原因は、

```txt
前のownerのuser情報が一瞬残っていた
↓
memberのGET /userが完了する前に
user.role === "owner" と判定された
↓
一瞬Teamが表示された
↓
その後member情報に更新されてTeamが消えた
```

これを防ぐため、AuthContext.tsxのuseEffect内を修正した。

---

# AuthContext.tsx 修正部分

修正前は以下。

```tsx
useEffect(() => {
  if (!token) {
    return
  }

  const fetchUser = async () => {
    ...
  }

  fetchUser()
}, [token])
```

修正後は以下。

```tsx
useEffect(() => {
  if (!token) {
    setUser(null)
    return
  }

  const fetchUser = async () => {
    ...
  }

  fetchUser()
}, [token])
```

追加したのはこれ。

```tsx
setUser(null)
```

意味は以下。

```txt
tokenがない
↓
ログインしていない
↓
userもnullにする
↓
前のログインユーザー情報を残さない
```

---

# AuthContext.tsx 現在コード

```tsx
import { createContext, useEffect, useState  } from "react"
import { getUser } from "../api/auth"
import { useNavigate } from "react-router-dom"

type User = {
  id: number
  name: string
  email: string
  teamId: number
  teamName: string
  role: "owner" | "member"
  profileImage: string | null
}

type AuthContextType = {
  user: User | null
  token: string | null
  setToken: React.Dispatch<React.SetStateAction<string | null>>
}

export const AuthContext = createContext<AuthContextType>({
  user: null,
  token: null,
  setToken: () => {},
})

export function AuthProvider({ children }: { children: React.ReactNode }) {

  const [token, setToken] = useState<string | null>(
    localStorage.getItem("token")
  )

  const [user, setUser] = useState<User | null>(null)

  const navigate = useNavigate()

  useEffect(() => {
    if (!token) {
      setUser(null)
      return
    }

    const fetchUser = async () => {
      const response = await getUser(token)

      if (response.status === 401) {
        localStorage.removeItem("token")
        setToken(null)
        setUser(null)
        navigate("/login")
        return
      }

      const data = await response.json()

      setUser(data)
    }

    fetchUser()
  }, [token])

  return (
    <AuthContext.Provider value={{ user, token, setToken }}>
      {children}
    </AuthContext.Provider>
  )
}
```

---

# Tinkerで確認したこと

LaravelのDBを確認するためにTinkerを使った。

Dockerを起動。

```bash
docker compose up -d
```

Laravelコンテナに入った。

```bash
docker exec -it コンテナ名 bash
```

コンテナ内で以下のような表示になった。

```bash
root@1cdf4c222cb5:/var/www#
```

意味は以下。

```txt
root
↓
コンテナ内で一番強い権限を持つユーザー

1cdf4c222cb5
↓
Dockerコンテナを識別するID

/var/www
↓
現在いるフォルダ

#
↓
rootユーザーで操作している印
```

---

# ls コマンド

```bash
ls
```

意味は以下。

```txt
今いるフォルダの中身を一覧表示する
```

表示されたもの。

```txt
Dockerfile
composer.phar
docker
docker-compose.yml
laravel
```

これは `/var/www` の中にこれらがあるという意味。

Laravel本体は `laravel` フォルダにあった。

---

# Laravelフォルダへ移動

```bash
cd laravel
```

意味は以下。

```txt
/var/www
↓
/var/www/laravel
```

へ移動する。

---

# Tinker起動

```bash
php artisan tinker
```

意味は以下。

```txt
Laravelの中身をコマンドで確認・操作できるモードに入る
```

---

# User一覧確認

Tinkerで以下を実行した。

```php
App\Models\User::all();
```

意味は以下。

```txt
Userモデルを使って
usersテーブルの全データを取得する
```

分解すると以下。

```txt
App\Models\User
↓
Userモデル

::all()
↓
全部取得する
```

Laravelでは、Userモデルは基本的にusersテーブルにつながっている。

つまり、

```php
App\Models\User::all();
```

はSQLでいうとだいたい以下に近い。

```sql
SELECT * FROM users;
```

---

# \ について

Laravel / PHP の名前空間では、バックスラッシュを使う。

正しい書き方。

```php
App\Models\User::all();
```

これは、

```txt
App
↓
Models
↓
User
```

という名前空間を `\` で区切っている。

重要。

```txt
\ ＝ バックスラッシュ
/ ＝ スラッシュ
```

「バックスラッシュ」と文字で表示されるわけではなく、記号として `\` と表示される。

---

# パスワード再設定

登録済みユーザーのパスワードを忘れていたので、Tinkerで再設定した。

確認用ユーザーは以下。

```txt
shugo@example.com
↓
owner

join@example.com
↓
member
```

Laravelのパスワードはハッシュ化されて保存されているため、今のパスワードを見ることはできない。

そのため、新しいパスワードに上書きした。

---

# owner側パスワード再設定

```php
$user = App\Models\User::where('email', 'shugo@example.com')->first();
```

意味は以下。

```txt
usersテーブルから
email が shugo@example.com のユーザーを1人取得する
```

次にパスワードを変更。

```php
$user->password = Hash::make('password');
```

意味は以下。

```txt
'password'
↓
実際にログイン時に入力するパスワード

Hash::make('password')
↓
Laravel用にハッシュ化する

$user->password = ...
↓
そのハッシュ化された値をpasswordに代入する
```

最後に保存。

```php
$user->save();
```

意味は以下。

```txt
変更内容をDBに保存する
```

これで、

```txt
email: shugo@example.com
password: password
```

でログインできるようになった。

---

# member側パスワード再設定

```php
$user = App\Models\User::where('email', 'join@example.com')->first();
```

意味は以下。

```txt
usersテーブルから
email が join@example.com のユーザーを1人取得する
```

パスワード変更。

```php
$user->password = Hash::make('password');
```

DB保存。

```php
$user->save();
```

これで、

```txt
email: join@example.com
password: password
```

でログインできるようになった。

---

# Hash::make を使う理由

Laravelでは、パスワードをそのまま保存しない。

ダメな例。

```php
$user->password = 'password';
```

これは平文保存になるのでダメ。

正しい例。

```php
$user->password = Hash::make('password');
```

理由は、Laravelのログイン認証は、入力されたパスワードをハッシュ化して、DB内のハッシュと照合するから。

---

# React起動確認

LaravelとDBはDockerで起動している。

Reactは別ターミナルで起動してよい。

流れは以下。

```txt
Docker側
↓
Laravel / DB 起動中

別ターミナル
↓
React開発サーバー起動
```

React側のフォルダに移動して実行。

```bash
npm run dev
```

ブラウザで表示されたlocalhostを開く。

例。

```txt
http://localhost:5173
```

---

# ブラウザのパスワード警告について

ログイン時にブラウザから、

```txt
パスワードを変更してください
```

のような警告が出た。

理由は、確認用パスワードが以下だったから。

```txt
password
```

これは弱いパスワードなので、Chromeなどが警告する。

ただし、今回はローカル開発用の確認なので、そのまま進めてOK。

本番では絶対に使わない。

---

# ブラウザで確認したこと

## owner確認

ログイン情報。

```txt
email: shugo@example.com
password: password
```

期待するSidebar。

```txt
Home
Matches
Players
Conditions
Team
Logout
```

確認結果。

```txt
Team と Logout が表示された
```

ownerなのでTeamが表示されてOK。

---

## member確認

ログイン情報。

```txt
email: join@example.com
password: password
```

期待するSidebar。

```txt
Home
Matches
Players
Conditions
Logout
```

確認結果。

```txt
一瞬Teamが表示されたが、その後消えた
最終的にLogoutだけになった
```

最終的にmemberでTeamが表示されなければOK。

一瞬表示された原因は、前のownerのuser情報が一瞬残った可能性。

その対策として、AuthContextで以下を追加した。

```tsx
if (!token) {
  setUser(null)
  return
}
```

---

# Teamページについて

Day12では、Teamページ本体はまだ作らない。

今回やったのは、

```txt
SidebarにTeamリンクを表示する
```

ところまで。

ロードマップ上では以下。

```txt
追加Day12
SidebarにTeamリンクを出す

追加Day13
GET /team APIを作る

追加Day14
/team のTeam管理画面を作る
```

つまり今は、

```txt
Teamページへの入口だけ作った状態
```

Teamをクリックしてページが表示されない、またはエラーになるのは、Day12時点では問題ない。

---

# 確認作業とテスト作業の違い

今回やったのは、主に手動確認。

```txt
手動確認
↓
ブラウザやPostmanで人間が実際に触って確認する

テスト
↓
コードで自動的に確認する
```

今回のDay12では、React側のSidebar整理なので、基本はブラウザ確認でOK。

実務でも、実装直後には自分で手動確認する。

実務の流れは以下。

```txt
1. 実装する
2. 自分でブラウザやPostmanで手動確認する
3. 必要ならテストコードを書く
4. Pull Requestを出す
5. レビュー
6. CIで自動テスト
7. ステージング環境で確認
8. 本番反映
```

今のロードマップでは、

```txt
手動確認
↓
今やる

自動テスト
↓
元ロードマップDay19以降でやる
```

という扱い。

---

# Postman確認について

Day12はReact側のSidebar整理なので、基本はブラウザ確認。

ただし、logout APIだけはPostmanで確認してもよい。

APIは以下。

```txt
POST http://localhost:8000/api/logout
```

Headers。

```txt
Accept: application/json
Authorization: Bearer ログイン時に取得したtoken
```

成功すれば、HTTPステータスが200や204になればOK。

ただし、今回ブラウザでLogoutが成功していれば、Day12の確認としては十分。

---

# 今日の重要理解まとめ

## 1. Contextは共有箱

```txt
AuthContext
↓
認証情報を入れている箱
```

```txt
AuthProvider
↓
その箱を使える範囲
```

```txt
useContext(AuthContext)
↓
箱の中身を取り出す
```

---

## 2. user / token / setToken の違い

```txt
user
↓
誰がログインしているか

token
↓
ログインしている証明

setToken
↓
token状態を変える関数
```

---

## 3. owner/member判定はuser.role

```tsx
user?.role === "owner"
```

意味。

```txt
ログイン中ユーザーが存在する
+
そのユーザーのroleがowner
↓
Teamを表示
```

---

## 4. ログアウトは3段階

```txt
1. Laravel側でtokenを無効化
2. localStorageのtokenを削除
3. React Stateのtokenをnullにする
```

コードでは以下。

```tsx
const response = await logout(token)

if (response.ok) {
  localStorage.removeItem("token")
  setToken(null)
  navigate("/")
}
```

---

## 5. localStorageとReact Stateの違い

```txt
localStorage
↓
ブラウザに保存しておくtoken

React State
↓
今Reactが使っているtoken
```

ログアウト時は両方消す。

---

## 6. Authorization: Bearer token の意味

```ts
"Authorization": `Bearer ${token}`
```

意味。

```txt
このtokenを持っているユーザーとして認証してください
```

Laravelはこのtokenを見て、ログイン中ユーザーかどうか判断する。

---

## 7. TinkerはDB確認・操作に使える

```bash
php artisan tinker
```

例。

```php
App\Models\User::all();
```

意味。

```txt
usersテーブルの全ユーザーを確認する
```

---

## 8. パスワードは見られないが再設定できる

Laravelのパスワードはハッシュ化される。

今のパスワードを見ることはできない。

確認用に変更する場合は以下。

```php
$user->password = Hash::make('password');
$user->save();
```

---

# 追加Day12 完了内容

```txt
追加Day12：Sidebar整理

完了内容
・SidebarにLogoutを追加
・AuthContextからuser/token/setTokenを取得
・ownerだけTeamリンク表示
・logout APIをauth.tsに追加
・logout API成功後にlocalStorageのtokenを削除
・setToken(null)でReact側tokenを削除
・navigate("/")で未ログインHomeへ移動
・AuthContextでtokenがない時にsetUser(null)を追加
・owner/memberでSidebar表示確認完了
・Logout動作確認完了
```

---

# 現在の到達状態

```txt
ownerでログイン
↓
SidebarにTeamが表示される

memberでログイン
↓
SidebarにTeamが表示されない

Logoutを押す
↓
Laravel側logout APIを呼ぶ
↓
成功したらlocalStorageとReact Stateのtokenを消す
↓
"/" に移動する
```

これで、追加Day12は完了。

---

# 次回やること

次回は追加Day13。

```txt
追加Day13：Team情報取得API
```

やること。

```txt
GET /team
team名
invite_code
所属ユーザー一覧返却
ownerのみ許可
```

到達状態。

```txt
チーム管理情報を取得できる
```

Day12ではTeamページへの入口を作った。

Day13では、そのTeamページで使うためのバックエンドAPIを作る。
