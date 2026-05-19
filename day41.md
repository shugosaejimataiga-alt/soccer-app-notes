

# 📘 Players強化 Day6 完全まとめ
（ログアウトAPI + ログイン中ユーザー取得 + auth:sanctum理解）

━━━━━━━━━━━━━━━━━━━
# ■ Day6 の目的
━━━━━━━━━━━━━━━━━━━

Laravel Sanctum を使い、

・ログイン中ユーザー取得
・ログアウト
・Bearer Token認証

を理解する。

最終目標：

GET /api/user
POST /api/logout

を完成させる。

━━━━━━━━━━━━━━━━━━━
# ■ Day6 完了時点の到達状態
━━━━━━━━━━━━━━━━━━━

・auth:sanctum middleware理解
・Bearer Token理解
・GET /api/user 完成
・POST /api/logout 完成
・token削除確認
・Unauthenticated確認
・Accept: application/json 理解

━━━━━━━━━━━━━━━━━━━
# ■ 今回完成したAPI
━━━━━━━━━━━━━━━━━━━

POST /api/login
GET  /api/user
POST /api/logout

━━━━━━━━━━━━━━━━━━━
# ■ 認証全体 flow（超重要）
━━━━━━━━━━━━━━━━━━━

ログイン
↓
token発行
↓
personal_access_tokens保存
↓
フロントへtoken返却
↓
Bearer Tokenとして送信
↓
auth:sanctum が本人確認
↓
$request->user() 取得可能
↓
logoutでtoken削除
↓
token無効化

━━━━━━━━━━━━━━━━━━━
# ■ usersテーブル と personal_access_tokens の違い
━━━━━━━━━━━━━━━━━━━

■ usersテーブル

役割：

ユーザー本体保存

保存内容：

id
name
email
password（ハッシュ化）
team_id

ログインしていようがいまいが、
全ユーザー保存。

━━━━━━━━━━━━━━━━━━━

■ personal_access_tokensテーブル

役割：

ログインtoken保存

ログイン成功後に作られる。

保存内容：

このtokenは誰のものか

━━━━━━━━━━━━━━━━━━━
# ■ ログイン flow 理解
━━━━━━━━━━━━━━━━━━━

email / password送信
↓
usersテーブルからemail検索
↓
Hash::checkでpassword確認
↓
成功
↓
$user->createToken()
↓
token生成
↓
personal_access_tokens保存
↓
生tokenをフロントへ返す

━━━━━━━━━━━━━━━━━━━
# ■ Bearer Token 理解
━━━━━━━━━━━━━━━━━━━

ログイン成功時：

{
  "message": "ログイン成功",
  "token": "3|xxxxxxxx"
}

返ってきた：

3|xxxxxxxx

これが token。

次回API通信時：

Authorization: Bearer 3|xxxxxxxx

として送る。

意味：

私はこのtokenを持っています

━━━━━━━━━━━━━━━━━━━
# ■ auth:sanctum の本質
━━━━━━━━━━━━━━━━━━━

ログインユーザー特定係

である。

流れ：

Bearer Token受け取り
↓
personal_access_tokens検索
↓
誰のtokenか判定
↓
ログインUser特定
↓
$request->user() へセット
↓
Controller実行

━━━━━━━━━━━━━━━━━━━
# ■ $request->user() の本質
━━━━━━━━━━━━━━━━━━━

意味：

現在ログイン中のUser

取得。

内部的には：

usersテーブル1行データ
+
Userモデル機能

を持つ。

━━━━━━━━━━━━━━━━━━━
# ■ Model 理解（超重要）
━━━━━━━━━━━━━━━━━━━

Modelは2役ある。

■ ① テーブル操作機能

例：

User::where(...)
User::find(...)
$user->createToken()
$user->save()

━━━━━━━━━━━━━━━━━━━

■ ② DB1行データの入れ物

$user = User::find(1);

の $user は：

usersテーブル1行データ
+
Userモデル機能

を持つ。

━━━━━━━━━━━━━━━━━━━
# ■ JSON返却 理解（超重要）
━━━━━━━━━━━━━━━━━━━

Laravel内部：

$user
=
データ + Model機能

しかし：

return response()->json([
    'user' => $user
]);

すると：

JSON化

される。

JSONは：

文字列
数値
配列
オブジェクトデータ

しか扱えない。

そのため：

createToken()
save()
team()

などの関数は返らない。

返るのは：

id
name
email
team_id

などのデータ部分のみ。

━━━━━━━━━━━━━━━━━━━
# ■ ポリモーフィックリレーション理解
━━━━━━━━━━━━━━━━━━━

Sanctumは普通の外部キーではない。

使用：

tokenable_type
tokenable_id

例：

tokenable_type = App\Models\User
tokenable_id = 1

意味：

Userモデルの id=1

つまり：

どのModelの何番か

を保存している。

━━━━━━━━━━━━━━━━━━━
# ■ なぜ外部キーではない？
━━━━━━━━━━━━━━━━━━━

Sanctumは：

User
Admin
Company

など複数Modelへtoken対応可能。

だから：

user_id 固定

ではなく、

tokenable_type
+
tokenable_id

で管理。

━━━━━━━━━━━━━━━━━━━
# ■ GET /api/user 作成
━━━━━━━━━━━━━━━━━━━

作成：

public function user(Request $request)
{
    return response()->json([
        'user' => $request->user(),
    ]);
}

意味：

ログイン中UserをJSON返却

━━━━━━━━━━━━━━━━━━━
# ■ logout() 作成
━━━━━━━━━━━━━━━━━━━

作成：

public function logout(Request $request)
{
    $request->user()->currentAccessToken()->delete();

    return response()->json([
        'message' => 'ログアウト成功',
    ]);
}

━━━━━━━━━━━━━━━━━━━
# ■ logout flow
━━━━━━━━━━━━━━━━━━━

$request->user()
↓
ログイン中User取得
↓
currentAccessToken()
↓
今回使われたtoken取得
↓
delete()
↓
personal_access_tokensから削除
↓
token無効化

━━━━━━━━━━━━━━━━━━━
# ■ routes/api.php 現在状態
━━━━━━━━━━━━━━━━━━━

<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\PlayerController;
use App\Http\Controllers\AuthController;

Route::get('/players', [PlayerController::class, 'index']);
Route::post('/players', [PlayerController::class, 'store']);
Route::put('/players/{id}', [PlayerController::class, 'update']);
Route::delete('/players/{id}', [PlayerController::class, 'destroy']);
Route::get('/players/{id}', [PlayerController::class, 'show']);

Route::post('/login', [AuthController::class, 'login']);

Route::middleware('auth:sanctum')->get('/user', [AuthController::class, 'user']);

Route::middleware('auth:sanctum')->post('/logout', [AuthController::class, 'logout']);

━━━━━━━━━━━━━━━━━━━
# ■ AuthController.php 現在状態
━━━━━━━━━━━━━━━━━━━

<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\User;
use Illuminate\Support\Facades\Hash;

class AuthController extends Controller
{
    public function login(Request $request)
    {
        $data = $request->validate([
        'email' => ['required', 'email'],
        'password' => ['required', 'string'],
        ]);

        $user = User::where('email', $data['email'])->first();

        if (!$user || !Hash::check($data['password'], $user->password)) {

        return response()->json([
            'message' => 'ログイン失敗'
        ], 401);

        }

        $token = $user->createToken('login-token')->plainTextToken;

        return response()->json([
        'message' => 'ログイン成功',
        'token' => $token,
        ]);
    }

    public function user(Request $request)
    {
        return response()->json([
            'user' => $request->user(),
        ]);
    }

    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'message' => 'ログアウト成功',
        ]);
    }
}

━━━━━━━━━━━━━━━━━━━
# ■ Accept: application/json 理解
━━━━━━━━━━━━━━━━━━━

意味：

JSON形式で返してください

未認証時：

Laravel本来：

ログイン画面へ戻そうとする

しかしAPIでは：

JSONエラーが欲しい

そのため：

Accept: application/json

を送る。

すると：

{
  "message": "Unauthenticated."
}

を返す。

━━━━━━━━━━━━━━━━━━━
# ■ 今回確認した重要挙動
━━━━━━━━━━━━━━━━━━━

ログイン成功
↓
token取得
↓
GET /user 成功
↓
ログアウト
↓
token削除
↓
同じtokenで再アクセス
↓
Unauthenticated.

つまり：

token無効化成功

まで確認できた。