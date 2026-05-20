

# 📘 Players強化 Day7 完全版まとめ
（Players API 認証必須化 + auth:sanctum middleware 理解）

━━━━━━━━━━━━━━━━━━━
■ Day7 の目的
━━━━━━━━━━━━━━━━━━━

Players API を：

ログイン必須API

へ変更する。

つまり：

未ログインでは
Players API を使えない

状態を作る。

使用技術：

auth:sanctum middleware

━━━━━━━━━━━━━━━━━━━
■ Day7 完了時点の到達状態
━━━━━━━━━━━━━━━━━━━

Players API 全体に：

auth:sanctum

適用完了。

確認済み：

未ログイン
↓
401 Unauthenticated.

ログイン済み：

Bearer Token付き
↓
Players API 使用可能

━━━━━━━━━━━━━━━━━━━
■ 今回理解した本質
━━━━━━━━━━━━━━━━━━━

今回最重要なのは：

Controller に入る前に
middleware が認証確認している

という構造。

流れ：

Request
↓
Route
↓
middleware(auth:sanctum)
↓
token確認
↓
本人確認成功
↓
Controller実行

失敗時：

Request
↓
middleware(auth:sanctum)
↓
token無し
↓
401 Unauthenticated.
↓
Controller実行されない

━━━━━━━━━━━━━━━━━━━
■ middleware の本質
━━━━━━━━━━━━━━━━━━━

middleware は：

APIへ入る前の検問所

役割。

つまり：

この条件を通った人だけ
Controllerへ進める

という仕組み。

今回の：

auth:sanctum

は：

ログイン済みか確認するmiddleware

━━━━━━━━━━━━━━━━━━━
■ なぜ認証が必要なのか
━━━━━━━━━━━━━━━━━━━

認証が無いと：

誰でも
選手一覧取得
作成
更新
削除

できてしまう。

つまり：

セキュリティ的に危険

実務では：

認証無しCRUD
=
危険API

扱い。

━━━━━━━━━━━━━━━━━━━
■ 今回変更した routes/api.php
━━━━━━━━━━━━━━━━━━━

変更前：

Route::get('/players', [PlayerController::class, 'index']);
Route::post('/players', [PlayerController::class, 'store']);
Route::put('/players/{id}', [PlayerController::class, 'update']);
Route::delete('/players/{id}', [PlayerController::class, 'destroy']);
Route::get('/players/{id}', [PlayerController::class, 'show']);

━━━━━━━━━━━━━━━━━━━

変更後：

Route::middleware('auth:sanctum')->group(function () {

    Route::get('/players', [PlayerController::class, 'index']);
    Route::post('/players', [PlayerController::class, 'store']);
    Route::put('/players/{id}', [PlayerController::class, 'update']);
    Route::delete('/players/{id}', [PlayerController::class, 'destroy']);
    Route::get('/players/{id}', [PlayerController::class, 'show']);

});

━━━━━━━━━━━━━━━━━━━
■ Route::middleware(...)->group の意味
━━━━━━━━━━━━━━━━━━━

意味：

この group 内は
全部 auth:sanctum 適用

もし group を使わない場合：

Route::middleware('auth:sanctum')->get(...);
Route::middleware('auth:sanctum')->post(...);
Route::middleware('auth:sanctum')->put(...);

毎回書く必要がある。

group を使うと：

共通middlewareをまとめられる

━━━━━━━━━━━━━━━━━━━
■ auth:sanctum の内部 flow
━━━━━━━━━━━━━━━━━━━

Bearer Token受信
↓
personal_access_tokens検索
↓
token所有者特定
↓
ログインUser特定
↓
$request->user() へ保存
↓
Controller許可

━━━━━━━━━━━━━━━━━━━
■ 今回確認した未ログイン挙動
━━━━━━━━━━━━━━━━━━━

Postman：

GET /api/players

送信。

ただし：

Authorization無し

結果：

{
  "message": "Unauthenticated."
}

━━━━━━━━━━━━━━━━━━━
■ なぜ JSON エラーが返ったのか
━━━━━━━━━━━━━━━━━━━

Headers に：

Accept: application/json

を付けたため。

意味：

APIとしてJSON形式で返してください

Laravel内部では：

通常Web画面なら
ログインページへ戻そうとする

しかし：

APIなのでJSONエラーを返した

━━━━━━━━━━━━━━━━━━━
■ 今回確認したログイン済み挙動
━━━━━━━━━━━━━━━━━━━

まず：

POST /api/login

成功。

返却：

{
  "message": "ログイン成功",
  "token": "4|xxxxxxxx"
}

━━━━━━━━━━━━━━━━━━━

次に：

GET /api/players

へ：

Authorization: Bearer 4|xxxxxxxx

付きでアクセス。

結果：

Players一覧取得成功

━━━━━━━━━━━━━━━━━━━
■ Bearer Token の本質
━━━━━━━━━━━━━━━━━━━

Bearer Token は：

私はログイン済みです

を証明する鍵。

つまり：

tokenを持つ人
=
ログイン済みとして扱う

━━━━━━━━━━━━━━━━━━━
■ 今回の重要理解
━━━━━━━━━━━━━━━━━━━

今回かなり重要なのは：

認証は Controller 内ではなく
middleware で止めている

という設計。

つまり：

Controller は
「認証済み前提」
で処理を書ける

実務Laravelで非常に重要。

━━━━━━━━━━━━━━━━━━━
■ 現在保護されているAPI
━━━━━━━━━━━━━━━━━━━

現在：

GET    /api/players
POST   /api/players
PUT    /api/players/{id}
DELETE /api/players/{id}
GET    /api/players/{id}

すべて：

ログイン必須

になっている。

━━━━━━━━━━━━━━━━━━━
■ 現在の routes/api.php 完成状態
━━━━━━━━━━━━━━━━━━━

<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\PlayerController;
use App\Http\Controllers\AuthController;

Route::middleware('auth:sanctum')->group(function () {

    Route::get('/players', [PlayerController::class, 'index']);
    Route::post('/players', [PlayerController::class, 'store']);
    Route::put('/players/{id}', [PlayerController::class, 'update']);
    Route::delete('/players/{id}', [PlayerController::class, 'destroy']);
    Route::get('/players/{id}', [PlayerController::class, 'show']);

});

Route::post('/login', [AuthController::class, 'login']);

Route::middleware('auth:sanctum')->get('/user', [AuthController::class, 'user']);

Route::middleware('auth:sanctum')->post('/logout', [AuthController::class, 'logout']);

━━━━━━━━━━━━━━━━━━━
■ Day7 で身についた技術
━━━━━━━━━━━━━━━━━━━

・auth:sanctum middleware
・Route group
・認証必須API化
・401 Unauthenticated
・Bearer Token認証
・middleware flow
・API保護
・Postman認証確認
・Accept: application/json

━━━━━━━━━━━━━━━━━━━
■ Day7 の本質まとめ
━━━━━━━━━━━━━━━━━━━

今回の本質は：

APIは公開するだけでは危険

だから：

middlewareで入口制御

する。

Laravelでは：

auth:sanctum

がその入口制御を担当している。