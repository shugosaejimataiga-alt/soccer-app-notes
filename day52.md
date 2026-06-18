

# 追加Day7 完全まとめ

（GET /user 強化）

━━━━━━━━━━━━━━━━━━━
現在のロードマップ位置
━━━━━━━━━━━━━━━━━━━

完了済み

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
GET /user 強化 ← 完了

次回

追加Day8
未ログインホーム画面

━━━━━━━━━━━━━━━━━━━
Day7の目的
━━━━━━━━━━━━━━━━━━━

フロントがログイン中ユーザーの状態を取得できるようにする。

今までは

```json
{
    "user": {
        ...
    }
}
```

のようにUserモデルをそのまま返していた。

Day7では

```text
teamId
teamName
role
profileImage
```

を含めて返せるようにした。

━━━━━━━━━━━━━━━━━━━
変更前のコード
━━━━━━━━━━━━━━━━━━━

AuthController

```php
public function user(Request $request)
{
    return response()->json([
        'user' => $request->user(),
    ]);
}
```

意味

```text
認証済みUserをそのまま返す
```

━━━━━━━━━━━━━━━━━━━
$request->user() の理解
━━━━━━━━━━━━━━━━━━━

```php
$request->user()
```

は

```text
auth:sanctum
↓
token確認
↓
personal_access_tokens検索
↓
users特定
↓
User取得
```

した結果のUserオブジェクトを返す。

例えば

```text
Join User
```

でログインしている場合

```php
$request->user()
```

の中身は

```text
id = 3
name = Join User
email = join@example.com
team_id = 3
role = member
profile_image = null
```

になる。

━━━━━━━━━━━━━━━━━━━
Userモデルの確認
━━━━━━━━━━━━━━━━━━━

確認ファイル

```text
app/Models/User.php
```

確認したコード

```php
public function team()
{
    return $this->belongsTo(Team::class);
}
```

意味

```text
User は Team に所属する
```

━━━━━━━━━━━━━━━━━━━
belongsTo の理解
━━━━━━━━━━━━━━━━━━━

```php
$user->team
```

を書くと

```text
users.team_id
↓
teams.id
↓
Team取得
```

できる。

例

```php
$user->team->name
```

↓

```text
FC Test
```

取得可能

━━━━━━━━━━━━━━━━━━━
変更後のコード
━━━━━━━━━━━━━━━━━━━

AuthController

```php
public function user(Request $request)
{
    $user = $request->user();

    return response()->json([
        'id' => $user->id,
        'name' => $user->name,
        'email' => $user->email,
        'teamId' => $user->team_id,
        'teamName' => $user->team->name,
        'role' => $user->role,
        'profileImage' => $user->profile_image,
    ]);
}
```

━━━━━━━━━━━━━━━━━━━
新しく学んだこと
━━━━━━━━━━━━━━━━━━━

$user = $request->user();

意味

```text
ログイン中のUserを取得して
$user変数へ保存
```

━━━━━━━━━━━━━━━━━━━

$user->team->name

意味

```text
User
↓
team()リレーション
↓
Team取得
↓
name取得
```

━━━━━━━━━━━━━━━━━━━

response()->json()

意味

```text
フロントへJSON返却
```

━━━━━━━━━━━━━━━━━━━
動作確認
━━━━━━━━━━━━━━━━━━━

URL

```text
http://localhost:8000/api/user
```

Method

```text
GET
```

Headers

```text
Authorization
Bearer トークン
```

━━━━━━━━━━━━━━━━━━━
実際の返却結果
━━━━━━━━━━━━━━━━━━━

```json
{
    "id": 3,
    "name": "Join User",
    "email": "join@example.com",
    "teamId": 3,
    "teamName": "FC Test",
    "role": "member",
    "profileImage": null
}
```

確認できたこと

```text
認証成功

ログインユーザー取得成功

team取得成功

teamName取得成功

role取得成功

profileImage取得成功

JSON整形成功
```

━━━━━━━━━━━━━━━━━━━
今回の本質
━━━━━━━━━━━━━━━━━━━

今回やったことは

```text
DBのデータ
↓
フロントが使いやすい形へ変換
```

である。

例

DB

```text
team_id
profile_image
```

↓

フロント向け

```json
teamId
profileImage
```

━━━━━━━━━━━━━━━━━━━
PlayerResourceとの違い
━━━━━━━━━━━━━━━━━━━

Player

```text
項目が多い
今後さらに増える
複数箇所で返す
```

↓

```php
PlayerResource
```

で整形

━━━━━━━━━━━━━━━━━━━

今回のUser

```text
返却項目が少ない
GET /userだけ
```

↓

Controller内で整形

━━━━━━━━━━━━━━━━━━━

理解した結論

```text
小規模
↓
Controllerで整形でもOK

大規模
複数箇所で使う
↓
Resourceが向いている
```

━━━━━━━━━━━━━━━━━━━
AuthController と RegisterController の違い
━━━━━━━━━━━━━━━━━━━

AuthController

担当

```text
認証
```

現在のメソッド

```php
login()

logout()

user()
```

役割

```text
ログイン

ログアウト

現在のユーザー取得
```

━━━━━━━━━━━━━━━━━━━

RegisterController

担当

```text
新規登録
```

現在のメソッド

```php
registerTeam()

joinTeam()
```

役割

```text
チーム作成登録

チーム参加登録
```

━━━━━━━━━━━━━━━━━━━
現在の認証関連API
━━━━━━━━━━━━━━━━━━━

```http
POST /login
```

```http
POST /logout
```

```http
GET /user
```

```http
POST /register/team
```

```http
POST /register/join
```

━━━━━━━━━━━━━━━━━━━
Day7終了時点の到達状態
━━━━━━━━━━━━━━━━━━━

フロントは

```http
GET /user
```

を呼ぶだけで

```text
ユーザー名

メールアドレス

チームID

チーム名

role

プロフィール画像
```

を取得できる。

これにより、

```text
ログイン状態管理

権限管理

チーム表示
```

の土台が完成した。
