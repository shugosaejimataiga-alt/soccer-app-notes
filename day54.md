

# 追加Day9 完全まとめ（追加Day10へ引き継ぎ）

━━━━━━━━━━━━━━━━━━━
現在のロードマップ位置
━━━━━━━━━━━━━━━━━━━

完了

・追加Day1 登録設計
・追加Day2 invite_code追加
・追加Day3 role追加
・追加Day4 profile_image追加
・追加Day5 チーム作成登録API
・追加Day6 チーム参加登録API
・追加Day7 GET /user強化
・追加Day8 Home画面
・追加Day9 ログイン画面 ← 完了

次回

追加Day10
新規登録画面

━━━━━━━━━━━━━━━━━━━
今回の目的
━━━━━━━━━━━━━━━━━━━

ログイン画面を作成し、

画面

↓

POST /login

↓

token保存

↓

/playersへ遷移

まで完成させる。

また、実務向けの構成としてAPI通信をapiフォルダへ分離した。

━━━━━━━━━━━━━━━━━━━
今回作成・変更したファイル
━━━━━━━━━━━━━━━━━━━

新規作成

src/pages/Login.tsx

src/api/auth.ts

変更

src/App.tsx

src/pages/Home.tsx

━━━━━━━━━━━━━━━━━━━
Home画面
━━━━━━━━━━━━━━━━━━━

useNavigateを使用。

ログインボタン

↓

/loginへ遷移。

━━━━━━━━━━━━━━━━━━━
App.tsx
━━━━━━━━━━━━━━━━━━━

追加Route

<Route
  path="/login"
  element={<Login />}
/>

追加

const isLoginPage =
location.pathname === "/login"

const shouldShowSidebar =
!isHomePage && !isLoginPage

表示

{shouldShowSidebar && <Sidebar />}

結果

Home

↓

Sidebarなし

Login

↓

Sidebarなし

Players

↓

Sidebarあり

━━━━━━━━━━━━━━━━━━━
Login.tsx
━━━━━━━━━━━━━━━━━━━

状態管理

const [email, setEmail] = useState("")
const [password, setPassword] = useState("")

メール入力

value={email}

onChange={(e)=>
setEmail(e.target.value)
}

パスワード入力

value={password}

onChange={(e)=>
setPassword(e.target.value)
}

ログインボタン

onClick={handleLogin}

━━━━━━━━━━━━━━━━━━━
handleLogin
━━━━━━━━━━━━━━━━━━━

現在

const handleLogin = async () => {

  const response =
    await login(email, password)

  const data =
    await response.json()

  localStorage.setItem(
    "token",
    data.token
  )

  navigate("/players")
}

処理の流れ

ログインボタン

↓

login()

↓

Laravel通信

↓

response受取

↓

JSON変換

↓

token保存

↓

Players画面へ遷移

━━━━━━━━━━━━━━━━━━━
auth.ts
━━━━━━━━━━━━━━━━━━━

現在

export async function login(
email: string,
password: string
) {

  const response = await fetch(
    "http://localhost:8000/api/login",
    {
      method: "POST",
      headers: {
        "Content-Type":
        "application/json",
      },
      body: JSON.stringify({
        email,
        password,
      }),
    }
  )

  return response
}

役割

認証API通信だけ担当。

━━━━━━━━━━━━━━━━━━━
責務分離
━━━━━━━━━━━━━━━━━━━

変更前

Login.tsx

↓

fetch

↓

Laravel

画面

+

API通信

全部担当

変更後

Login.tsx

↓

login()

↓

auth.ts

↓

fetch

↓

Laravel

画面担当

↓

Login.tsx

API通信担当

↓

auth.ts

━━━━━━━━━━━━━━━━━━━
今後のapi構成方針
━━━━━━━━━━━━━━━━━━━

src/api

auth.ts

ログイン
ログアウト
GET /user

register.ts

チーム作成登録
チーム参加登録

players.ts

選手取得
追加
更新
削除

team.ts

チーム取得
参加コード再生成
owner譲渡
メンバー除外
チーム削除

すべてAPI通信を書く場所。

画面(tsx)にはfetchを書かない。

━━━━━━━━━━━━━━━━━━━
fetch通信
━━━━━━━━━━━━━━━━━━━

await fetch(...)

意味

Laravelへ非同期通信。

await

↓

通信終了まで待つ。

method

POST

↓

Route::post()

headers

Content-Type:
application/json

↓

JSONで送ります。

body

JSON.stringify({
email,
password
})

↓

Laravelへ送るデータ。

━━━━━━━━━━━━━━━━━━━
Routeの決まり方
━━━━━━━━━━━━━━━━━━━

React

URL

/api/login

+

method

POST

↓

Laravel

Route::post(
'/login'
)

↓

AuthController@login()

URLだけではなく、

HTTPメソッドも一致する必要がある。

━━━━━━━━━━━━━━━━━━━
response
━━━━━━━━━━━━━━━━━━━

const response =
await login(...)

意味

Laravelから返ってきた返事。

response.json()

↓

JavaScriptオブジェクトへ変換。

例

Laravel

{
  "token":"abc123"
}

↓

React

data.token

━━━━━━━━━━━━━━━━━━━
localStorage
━━━━━━━━━━━━━━━━━━━

保存

localStorage.setItem(
"token",
data.token
)

左

"token"

↓

箱の名前(Key)

右

data.token

↓

保存する値(Value)

取り出し

const token =
localStorage.getItem("token")

━━━━━━━━━━━━━━━━━━━
token
━━━━━━━━━━━━━━━━━━━

ログイン済み証明書。

認証が必要なAPIでは

const token =
localStorage.getItem("token")

↓

headers: {
Authorization:
`Bearer ${token}`
}

を付けて送る。

Laravel

↓

auth:sanctum

↓

token確認

↓

ログイン済みなら処理許可。

━━━━━━━━━━━━━━━━━━━
重要理解
━━━━━━━━━━━━━━━━━━━

useState

↓

画面内だけ保持。

リロードすると消える。

localStorage

↓

ブラウザへ保存。

リロードしても残る。

auth.ts

↓

API通信だけ担当。

Login.tsx

↓

画面制御だけ担当。

API通信はapiフォルダへまとめる。

これを今後

register.ts

players.ts

team.ts

にも統一していく。