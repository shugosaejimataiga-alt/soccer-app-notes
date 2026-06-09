

# 追加Day5 完全まとめ
（新規登録API① チーム作成）

━━━━━━━━━━━━━━━━━━━
現在のロードマップ位置
━━━━━━━━━━━━━━━━━━━

完了済み

追加Day1
登録設計・role設計

追加Day2
invite_code追加

追加Day3
role追加

追加Day4
profile_image追加

追加Day5
新規登録API① チーム作成 ← 完了

次回

追加Day6
新規登録API② チーム参加

━━━━━━━━━━━━━━━━━━━
Day5の目的
━━━━━━━━━━━━━━━━━━━

POST /register/team

を作成し、

新しいチームを作りながら
ownerユーザーを登録できるようにする。

最終的に

・Team作成
・User作成
・role=owner
・team_id紐付け
・token発行

を行う。

━━━━━━━━━━━━━━━━━━━
今日作成したController
━━━━━━━━━━━━━━━━━━━

作成コマンド

php artisan make:controller RegisterController

作成ファイル

app/Http/Controllers/RegisterController.php

━━━━━━━━━━━━━━━━━━━
完成したregisterTeam()
━━━━━━━━━━━━━━━━━━━

処理の流れ

① 入力チェック

↓

② invite_code生成

↓

③ Team作成

↓

④ User作成

↓

⑤ Token発行

↓

⑥ JSON返却

━━━━━━━━━━━━━━━━━━━
入力チェック
━━━━━━━━━━━━━━━━━━━

実装

$request->validate([
    'team_name' => 'required|string|max:255',
    'name' => 'required|string|max:255',
    'email' => 'required|email|unique:users,email',
    'password' => 'required|min:8',
]);

学んだこと

required
↓
必須

string
↓
文字列

max:255
↓
255文字まで

unique:users,email
↓
usersテーブルのemail重複禁止

min:8
↓
8文字以上

━━━━━━━━━━━━━━━━━━━
invite_code生成
━━━━━━━━━━━━━━━━━━━

実装

$inviteCode = Str::random(8);

学んだこと

Str
↓
文字列操作クラス

random(8)
↓
8文字ランダム生成

例

kRN2GqvX

━━━━━━━━━━━━━━━━━━━
Team作成
━━━━━━━━━━━━━━━━━━━

実装

$team = Team::create([
    'name' => $request->team_name,
    'invite_code' => $inviteCode,
]);

学んだこと

Team::create()

↓

teamsテーブルへ保存

保存後

$team

の中には

id
name
invite_code

を持ったTeamオブジェクトが入る

例

$team->id

↓

3

━━━━━━━━━━━━━━━━━━━
User作成
━━━━━━━━━━━━━━━━━━━

実装

$user = User::create([
    'name' => $request->name,
    'email' => $request->email,
    'password' => Hash::make($request->password),
    'team_id' => $team->id,
    'role' => 'owner',
]);

学んだこと

Hash::make()

↓

パスワードをハッシュ化

team_id

↓

所属チーム

role

↓

owner権限

━━━━━━━━━━━━━━━━━━━
Token発行
━━━━━━━━━━━━━━━━━━━

実装

$token = $user->createToken('auth_token')->plainTextToken;

学んだこと

createToken()

↓

Sanctumトークン作成

auth_token

↓

トークンの名前

認証には使われない

人間が管理しやすくするラベル

plainTextToken

↓

実際のトークン文字列取得

例

1|abcXYZ123...

━━━━━━━━━━━━━━━━━━━
JSON返却
━━━━━━━━━━━━━━━━━━━

実装

return response()->json([
    'token' => $token,
]);

学んだこと

Laravel

↓

token生成

↓

JSON返却

React

↓

token受取

↓

localStorage保存

↓

ログイン状態

━━━━━━━━━━━━━━━━━━━
localStorage理解
━━━━━━━━━━━━━━━━━━━

重要

localStorage保存は

Laravel

ではなく

React

の仕事

Laravel

↓

token返却

React

↓

localStorage保存

━━━━━━━━━━━━━━━━━━━
ルート追加
━━━━━━━━━━━━━━━━━━━

追加

use App\Http\Controllers\RegisterController;

Route::post(
    '/register/team',
    [RegisterController::class, 'registerTeam']
);

意味

POST /register/team

↓

registerTeam()実行

━━━━━━━━━━━━━━━━━━━
本日最大の学び
━━━━━━━━━━━━━━━━━━━

fillable

━━━━━━━━━━━━━━━━━━━
最初に発生したエラー
━━━━━━━━━━━━━━━━━━━

invite_code保存エラー

原因

Team.php

protected $fillable = [
    'name'
];

だった

━━━━━━━━━━━━━━━━━━━
理解した本質
━━━━━━━━━━━━━━━━━━━

fillable

↓

保存処理ではない

↓

保存許可リスト

例

protected $fillable = [
    'name',
    'invite_code',
];

意味

Team::create()

で

name
invite_code

を保存して良い

━━━━━━━━━━━━━━━━━━━
さらに発生したエラー
━━━━━━━━━━━━━━━━━━━

team_id保存エラー

原因

User.php

protected $fillable = [
    'name',
    'email',
    'password',
];

だった

━━━━━━━━━━━━━━━━━━━
修正
━━━━━━━━━━━━━━━━━━━

protected $fillable = [
    'name',
    'email',
    'password',
    'team_id',
    'role',
    'profile_image',
];

━━━━━━━━━━━━━━━━━━━
fillableの本質
━━━━━━━━━━━━━━━━━━━

Migration

↓

DBにカラム作成

Controller

↓

保存したい値を書く

fillable

↓

保存許可

DB

↓

実際に保存

━━━━━━━━━━━━━━━━━━━
本日理解したテーブル構造
━━━━━━━━━━━━━━━━━━━

teams

id
name
invite_code

━━━━━━━━━━━━━━━━━━━

users

id
name
email
password
team_id
role
profile_image

━━━━━━━━━━━━━━━━━━━

personal_access_tokens

id
tokenable_id
name
token

━━━━━━━━━━━━━━━━━━━
本日理解した関連
━━━━━━━━━━━━━━━━━━━

teams.id

↓

users.team_id

━━━━━━━━━━━━━━━━━━━

users.id

↓

personal_access_tokens.tokenable_id

━━━━━━━━━━━━━━━━━━━
図で表すと
━━━━━━━━━━━━━━━━━━━

Team

↓

User

↓

Token

━━━━━━━━━━━━━━━━━━━
動作確認結果
━━━━━━━━━━━━━━━━━━━

Team

id = 3

name = FC Test

invite_code = kRN2GqvX

作成成功

━━━━━━━━━━━━━━━━━━━

User

id = 2

name = Shugo

email = shugo@example.com

team_id = 3

role = owner

profile_image = null

作成成功

━━━━━━━━━━━━━━━━━━━

Token

発行成功

JSON返却成功

━━━━━━━━━━━━━━━━━━━
本日の到達状態
━━━━━━━━━━━━━━━━━━━

POST /register/team

↓

Team作成

↓

invite_code生成

↓

User作成

↓

team_id紐付け

↓

role=owner

↓

Token発行

↓

JSON返却

まで完成

追加Day5 完了