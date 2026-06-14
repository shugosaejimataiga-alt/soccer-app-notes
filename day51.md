

# 追加Day6：新規登録API② チーム参加 完全まとめ

━━━━━━━━━━━━━━━━━━━
ロードマップ位置
━━━━━━━━━━━━━━━━━━━

完了済み

* 追加Day1 登録設計・role設計
* 追加Day2 invite_code追加
* 追加Day3 users.role追加
* 追加Day4 users.profile_image追加
* 追加Day5 新規登録API① チーム作成
* 追加Day6 新規登録API② チーム参加 ← 本日完了

次回

* 追加Day7 GET /user 強化

━━━━━━━━━━━━━━━━━━━
Day6の目的
━━━━━━━━━━━━━━━━━━━

既存チームへ参加できるようにする。

実現したかったこと

* invite_codeでチームを探す
* 新しいユーザーを作る
* role=memberで登録する
* team_idを紐付ける
* Sanctumトークンを発行する
* tokenを返却する

━━━━━━━━━━━━━━━━━━━
追加したルート
━━━━━━━━━━━━━━━━━━━

routes/api.php

```php
Route::post(
    '/register/join',
    [RegisterController::class, 'joinTeam']
);
```

意味

```text
POST /api/register/join

↓

RegisterController

↓

joinTeam()
```

Routeの役割

```text
URLと実行するControllerメソッドを結び付ける
```

━━━━━━━━━━━━━━━━━━━
完成したjoinTeam()
━━━━━━━━━━━━━━━━━━━

app/Http/Controllers/RegisterController.php

```php
public function joinTeam(Request $request)
{
    $request->validate([
        'invite_code' => 'required|string',
        'name' => 'required|string|max:255',
        'email' => 'required|email|unique:users,email',
        'password' => 'required|min:8',
    ]);

    $team = Team::where(
        'invite_code',
        $request->invite_code
    )->first();

    if (!$team) {
        return response()->json([
            'message' => 'Invalid invite code',
        ], 404);
    }

    $user = User::create([
        'name' => $request->name,
        'email' => $request->email,
        'password' => Hash::make($request->password),
        'team_id' => $team->id,
        'role' => 'member',
    ]);

    $token = $user->createToken(
        'auth_token'
    )->plainTextToken;

    return response()->json([
        'token' => $token,
    ]);
}
```

━━━━━━━━━━━━━━━━━━━
処理の流れ
━━━━━━━━━━━━━━━━━━━

```text
POST /register/join

↓

入力チェック

↓

invite_code検索

↓

Team取得

↓

Teamが存在しなければ404

↓

User作成

↓

role=member

↓

team_id紐付け

↓

Sanctumトークン発行

↓

token返却
```

━━━━━━━━━━━━━━━━━━━
学んだこと①
$request->invite_code
━━━━━━━━━━━━━━━━━━━

```php
$request->invite_code
```

意味

```text
リクエストで送られてきた
invite_codeの値
```

例

送信

```json
{
    "invite_code": "kRN2GqvX"
}
```

↓

```php
$request->invite_code
```

↓

```php
"kRN2GqvX"
```

━━━━━━━━━━━━━━━━━━━
学んだこと②
where()
━━━━━━━━━━━━━━━━━━━

```php
Team::where(
    'invite_code',
    $request->invite_code
)
```

意味

```text
teamsテーブルから

invite_code が一致する行を探す
```

イメージ

```sql
SELECT *
FROM teams
WHERE invite_code = 'kRN2GqvX'
```

━━━━━━━━━━━━━━━━━━━
学んだこと③
first()
━━━━━━━━━━━━━━━━━━━

```php
->first();
```

意味

```text
検索結果の先頭1件を取得する
```

つまり

```php
$team = Team::where(...)->first();
```

は

```text
条件に一致するTeamを1件取得して
$teamへ入れる
```

━━━━━━━━━━━━━━━━━━━
学んだこと④
404エラー処理
━━━━━━━━━━━━━━━━━━━

```php
if (!$team) {
    return response()->json([
        'message' => 'Invalid invite code',
    ], 404);
}
```

意味

```text
チームが見つからなかったら

処理を終了して

404エラーを返す
```

━━━━━━━━━━━━━━━━━━━
学んだこと⑤
User作成
━━━━━━━━━━━━━━━━━━━

```php
$user = User::create([
    ...
]);
```

意味

```text
usersテーブルへ
新しいユーザーを保存する
```

今回重要だった部分

```php
'team_id' => $team->id,
'role' => 'member',
```

意味

```text
見つけたチームへ所属

かつ

権限はmember
```

━━━━━━━━━━━━━━━━━━━
学んだこと⑥
createToken()
━━━━━━━━━━━━━━━━━━━

```php
$user->createToken('auth_token')
```

意味

```text
このユーザー用の
認証トークンを発行する
```

auth_token

```php
'auth_token'
```

は

```text
トークンの名前

人間が分かりやすくするためのラベル
```

━━━━━━━━━━━━━━━━━━━
学んだこと⑦
plainTextToken
━━━━━━━━━━━━━━━━━━━

```php
->plainTextToken
```

意味

```text
実際のトークン文字列を取り出す
```

イメージ

```php
createToken()
```

↓

```text
トークンオブジェクト
```

↓

```php
->plainTextToken
```

↓

```text
7|xxxxxxxxxxxxxxxxxxxx
```

↓

```php
$token
```

へ保存

━━━━━━━━━━━━━━━━━━━
学んだこと⑧
JSON返却
━━━━━━━━━━━━━━━━━━━

```php
return response()->json([
    'token' => $token,
]);
```

意味

```json
{
    "token": "7|xxxxxxxxxxxx"
}
```

として返す。

左

```php
'token'
```

↓

JSONのキー

右

```php
$token
```

↓

値

━━━━━━━━━━━━━━━━━━━
学んだこと⑨
tokenの使われ方
━━━━━━━━━━━━━━━━━━━

登録

↓

```text
token発行
```

↓

フロント

↓

```text
localStorageへ保存
```

↓

次回API呼び出し

↓

```text
Authorization: Bearer token
```

↓

Laravel

↓

```text
auth:sanctum
```

↓

User特定

↓

```php
auth()->user()
```

利用可能

━━━━━━━━━━━━━━━━━━━
学んだこと⑩
middleware(auth:sanctum)
━━━━━━━━━━━━━━━━━━━

理解した重要ポイント

middlewareは単なる

```text
ログインしているか確認
```

だけではない。

実際には

```text
token取得

↓

personal_access_tokens検索

↓

User特定

↓

auth()->user()へセット
```

まで行っている。

流れ

```text
リクエスト

↓

middleware(auth:sanctum)

↓

token確認

↓

personal_access_tokens

↓

users

↓

auth()->user()

↓

Controller実行
```

━━━━━━━━━━━━━━━━━━━
動作確認結果
━━━━━━━━━━━━━━━━━━━

Postman

```http
POST /api/register/join
```

送信成功

返却

```json
{
    "token": "7|..."
}
```

成功確認

* token返却
* User作成
* team_id保存
* role保存

━━━━━━━━━━━━━━━━━━━
Tinker確認結果
━━━━━━━━━━━━━━━━━━━

```php
User::latest()->first();
```

結果

```text
id: 3

name: Join User

email: join@example.com

team_id: 3

role: member

profile_image: null
```

確認できたこと

```text
User作成成功

team_id保存成功

role=member保存成功

Hash化成功
```

━━━━━━━━━━━━━━━━━━━
現在のDB状態
━━━━━━━━━━━━━━━━━━━

Team

```text
id: 3

name: FC Test

invite_code: kRN2GqvX
```

Owner

```text
id: 2

team_id: 3

role: owner
```

Member

```text
id: 3

team_id: 3

role: member
```

構造

```text
FC Test

├─ Shugo      owner
└─ Join User  member
```

━━━━━━━━━━━━━━━━━━━
Day6完了時点で出来ること
━━━━━━━━━━━━━━━━━━━

```text
チーム作成

↓

owner登録

↓

invite_code発行

↓

別ユーザーがinvite_code入力

↓

既存チームへ参加

↓

member作成

↓

同じteam_idへ所属
```

━━━━━━━━━━━━━━━━━━━
追加Day6 到達状態
━━━━━━━━━━━━━━━━━━━

```text
既存チームへ参加できる

POST /register/join 完成

動作確認完了
```
