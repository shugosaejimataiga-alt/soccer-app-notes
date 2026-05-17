# Players強化 Day5 前半 完全まとめ（修正版）

# 今日の目的

Laravel Sanctum を導入し、
ログインAPIを作成する

━━━━━━━━━━━━━━━━━━━
1. Laravel / Composer / vendor / framework 構造理解
━━━━━━━━━━━━━━━━━━━

■ 正しい構造

appコンテナ
├ PHP
├ Composer
└ /var/www/laravel ← Laravelプロジェクト
    ├ app
    ├ config
    ├ database
    ├ routes
    ├ composer.json
    └ vendor
        └ laravel
            ├ framework
            └ sanctum

■ PHP

PHP
= コード実行エンジン

■ Composer

Composer
= PHPライブラリ管理ツール

■ vendor

vendor
= Composerが入れた外部ライブラリ置き場

■ Laravel framework

vendor/laravel/framework

Laravel本体エンジン

■ Sanctum

vendor/laravel/sanctum

Laravel公式認証ライブラリ

━━━━━━━━━━━━━━━━━━━
2. Sanctum導入
━━━━━━━━━━━━━━━━━━━

実行：

composer require laravel/sanctum

意味：

Composerが
vendor/laravel/sanctum
を追加

━━━━━━━━━━━━━━━━━━━
3. vendor:publish 理解
━━━━━━━━━━━━━━━━━━━

実行：

php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

■ 重要理解

これは：

Sanctum本体コピー

ではない。

正しくは：

Sanctumが持つ
migration/configテンプレートを
アプリ側へコピー

である。

■ コピー元

vendor/laravel/sanctum/database/migrations
vendor/laravel/sanctum/config

■ コピー先

database/migrations
config

■ なぜコピーする？

vendor
= 外部ライブラリ領域

なので、

composer update

などで変更される可能性がある。

だから：

アプリ側で管理するため

コピーする。

━━━━━━━━━━━━━━━━━━━
4. migration と table の違い（超重要）
━━━━━━━━━━━━━━━━━━━

■ migration

DB構造の設計書

例：

Schema::create(...)

などを書く。

■ php artisan migrate

migration設計書を実行し、
実際のDBテーブルを作成

する。

■ table

users
players
personal_access_tokens

など。

役割：

実データ保存場所

■ 流れ

migration
↓ 実行
table作成
↓
実データ保存

━━━━━━━━━━━━━━━━━━━
5. Sanctum migration実行
━━━━━━━━━━━━━━━━━━━

確認：

ls database/migrations

存在確認：

create_personal_access_tokens_table.php

実行：

php artisan migrate

結果：

personal_access_tokens
テーブル作成

━━━━━━━━━━━━━━━━━━━
6. personal_access_tokens テーブル役割
━━━━━━━━━━━━━━━━━━━

役割：

ログインtoken保存

保存内容：

どのtokenが
どのUserのものか

━━━━━━━━━━━━━━━━━━━
7. Userモデルへ Sanctum追加
━━━━━━━━━━━━━━━━━━━

追加：

use Laravel\Sanctum\HasApiTokens;

use HasApiTokens, HasFactory, Notifiable;

■ HasApiTokens役割

Sanctum token機能

を Userモデルへ追加。

■ 追加された機能

$user->createToken(...)

━━━━━━━━━━━━━━━━━━━
8. Model理解（超重要）
━━━━━━━━━━━━━━━━━━━

■ usersテーブル

実データ保存場所

■ Userモデル

usersテーブル操作用クラス

■ 重要理解

User::where(...)

は、

Userモデルを使って
usersテーブル検索

している。

■ $user の本質

$user = User::where(...)->first();

の $user は：

usersテーブル1行データ
+
Userモデル機能

である。

■ だから可能

$user->password

DBデータ取得可能。

$user->createToken()

Model機能実行可能。

━━━━━━━━━━━━━━━━━━━
9. AuthController作成
━━━━━━━━━━━━━━━━━━━

実行：

php artisan make:controller AuthController

作成：

app/Http/Controllers/AuthController.php

━━━━━━━━━━━━━━━━━━━
10. loginメソッド作成
━━━━━━━━━━━━━━━━━━━

public function login(Request $request)
{
}

━━━━━━━━━━━━━━━━━━━
11. Request / User / Hash 読み込み
━━━━━━━━━━━━━━━━━━━

use Illuminate\Http\Request;
use App\Models\User;
use Illuminate\Support\Facades\Hash;

■ Hash役割

入力password
と
DBハッシュpassword
比較

━━━━━━━━━━━━━━━━━━━
12. loginバリデーション
━━━━━━━━━━━━━━━━━━━

$data = $request->validate([
    'email' => ['required', 'email'],
    'password' => ['required', 'string'],
]);

━━━━━━━━━━━━━━━━━━━
13. User検索
━━━━━━━━━━━━━━━━━━━

$user = User::where('email', $data['email'])->first();

■ 意味

usersテーブルから
email一致Userを1件取得

■ first() の意味

検索結果1件目取得

━━━━━━━━━━━━━━━━━━━
14. password確認
━━━━━━━━━━━━━━━━━━━

if (!$user || !Hash::check($data['password'], $user->password)) {

■ passwordを検索しない理由

passwordは：

ハッシュ化保存

されているから。

■ Hash::check役割

入力password
↓
同方式ハッシュ化
↓
DBハッシュと比較

━━━━━━━━━━━━━━━━━━━
15. ログイン失敗レスポンス
━━━━━━━━━━━━━━━━━━━

return response()->json([
    'message' => 'ログイン失敗'
], 401);

■ 401意味

認証失敗
Unauthorized

━━━━━━━━━━━━━━━━━━━
16. token作成
━━━━━━━━━━━━━━━━━━━

$token = $user->createToken('login-token')->plainTextToken;

■ 内部で起きること

① ランダムtoken生成
② DB保存用にハッシュ化
③ personal_access_tokensへ保存
④ 生token生成・保持

■ plainTextToken意味

API通信で使う
生token取得

━━━━━━━━━━━━━━━━━━━
17. ログイン成功レスポンス
━━━━━━━━━━━━━━━━━━━

return response()->json([
    'message' => 'ログイン成功',
    'token' => $token,
]);

■ フロントへ返るもの

{
  "message": "ログイン成功",
  "token": "3|xxxxxxxx"
}

━━━━━━━━━━━━━━━━━━━
18. token認証全体像
━━━━━━━━━━━━━━━━━━━

ログイン成功
↓
token生成
↓
DBへハッシュtoken保存
↓
フロントへ生token返却
↓
次回API通信時にBearer token送信
↓
Laravelがtoken照合
↓
ログイン済み判定

━━━━━━━━━━━━━━━━━━━
19. routes/api.php
━━━━━━━━━━━━━━━━━━━

追加：

use App\Http\Controllers\AuthController;

Route::post('/login', [AuthController::class, 'login']);

■ 意味

POST /api/login
↓
AuthController@login 実行

━━━━━━━━━━━━━━━━━━━
20. 現在完成している状態
━━━━━━━━━━━━━━━━━━━

Sanctum導入完了
↓
personal_access_tokens テーブル作成完了
↓
UserへHasApiTokens追加完了
↓
AuthController完成
↓
login処理完成
↓
POST /api/login ルート接続完了

━━━━━━━━━━━━━━━━━━━
21. 次回やること（Day5後半）
━━━━━━━━━━━━━━━━━━━

Postmanで
POST /api/login
実行確認