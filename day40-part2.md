

📘 Players強化 Day5 完全版まとめ（Sanctum認証導入 + ログインAPI完成）

━━━━━━━━━━━━━━━━━━━
■ Day5 の目的
━━━━━━━━━━━━━━━━━━━

Laravel Sanctum を導入し、

・ログインAPI作成
・token認証導入
・Postmanでログイン確認

を行う。

最終目標：

POST /api/login

で、

ログイン成功
↓
token返却

を実現する。

━━━━━━━━━━━━━━━━━━━
■ 今日完成した状態
━━━━━━━━━━━━━━━━━━━

Sanctum導入完了
↓
personal_access_tokens テーブル作成完了
↓
UserモデルへHasApiTokens追加完了
↓
AuthController作成完了
↓
POST /api/login 作成完了
↓
Postmanログイン成功
↓
token返却成功
↓
DB保存確認完了

━━━━━━━━━━━━━━━━━━━
■ Laravel構造 理解
━━━━━━━━━━━━━━━━━━━

現在の構造：

appコンテナ
├ PHP
├ Composer
└ /var/www/laravel
    ├ app
    ├ config
    ├ database
    ├ routes
    ├ vendor

■ PHP

コード実行エンジン

■ Composer

PHPライブラリ管理ツール

使用：

composer require laravel/sanctum

■ vendor

Composerがインストールした
外部ライブラリ置き場

例：

vendor/laravel/framework
vendor/laravel/sanctum

■ framework

場所：

vendor/laravel/framework

意味：

Laravel本体エンジン

■ Sanctum

場所：

vendor/laravel/sanctum

意味：

Laravel公式token認証ライブラリ

━━━━━━━━━━━━━━━━━━━
■ migration と table の違い（超重要）
━━━━━━━━━━━━━━━━━━━

■ migration

DB構造の設計書

例：

Schema::create(...)

を書く。

■ php artisan migrate

migration設計書を実行し、
実際のDBテーブルを作る

■ table

例：

users
players
teams
personal_access_tokens

意味：

実データ保存場所

■ 全体流れ

migration
↓
php artisan migrate
↓
table作成
↓
実データ保存

━━━━━━━━━━━━━━━━━━━
■ vendor:publish 理解
━━━━━━━━━━━━━━━━━━━

実行：

php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

■ 本当の意味

これは：

Sanctum本体コピー

ではない。

正しくは：

Sanctumが持つ
migration/configテンプレートを
アプリ側へコピー

している。

■ コピー元

vendor/laravel/sanctum/database/migrations
vendor/laravel/sanctum/config

■ コピー先

database/migrations
config

■ なぜコピーする？

理由：

vendor
= 外部ライブラリ領域

だから。

つまり：

composer update

で変更される可能性がある。

そのため：

アプリ側で安全に管理するため

コピーする。

━━━━━━━━━━━━━━━━━━━
■ Sanctum token認証 理解
━━━━━━━━━━━━━━━━━━━

全体流れ：

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
■ Model 理解（超重要）
━━━━━━━━━━━━━━━━━━━

■ usersテーブル

実データ保存場所

■ Userモデル

usersテーブル操作クラス

■ 超重要理解

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

DBデータ取得。

$user->createToken(...)

Model機能実行。

━━━━━━━━━━━━━━━━━━━
■ 今日実行したコマンド
━━━━━━━━━━━━━━━━━━━

■ Sanctum導入

composer require laravel/sanctum

■ Sanctum設定コピー

php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

■ migration確認

ls database/migrations

■ migration実行

php artisan migrate

■ AuthController作成

php artisan make:controller AuthController

■ Tinker起動

php artisan tinker

━━━━━━━━━━━━━━━━━━━
■ Docker理解（超重要）
━━━━━━━━━━━━━━━━━━━

最初に間違えた：

docker compose exec app php artisan tinker

を、

コンテナ内部

で実行した。

■ なぜエラー？

理由：

dockerコマンドは
Windows側（ホスト側）のコマンド

だから。

■ 現在地

root@xxxx:/var/www/laravel#

これは：

appコンテナ内部
+
Laravelプロジェクト内部

という意味。

■ だから可能

コンテナ内部では：

php artisan tinker

だけでよい。

━━━━━━━━━━━━━━━━━━━
■ User.php 現在状態
━━━━━━━━━━━━━━━━━━━

場所：

app/Models/User.php

重要部分：

use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;

    public function team()
    {
        return $this->belongsTo(Team::class);
    }
}

■ HasApiTokens の意味

Sanctum token機能追加

■ 追加された機能

$user->createToken(...)

━━━━━━━━━━━━━━━━━━━
■ AuthController.php 現在完成コード
━━━━━━━━━━━━━━━━━━━

場所：

app/Http/Controllers/AuthController.php

コード：

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
}

━━━━━━━━━━━━━━━━━━━
■ login処理 flow
━━━━━━━━━━━━━━━━━━━

① email/password バリデーション
↓
② usersテーブル検索
↓
③ Hash::check でpassword照合
↓
④ 失敗時401返却
↓
⑤ createToken 実行
↓
⑥ personal_access_tokensへ保存
↓
⑦ plainTextToken取得
↓
⑧ JSON返却

━━━━━━━━━━━━━━━━━━━
■ createToken の内部理解
━━━━━━━━━━━━━━━━━━━

$user->createToken('login-token')

内部：

① ランダムtoken生成
② DB保存用ハッシュ化
③ personal_access_tokens保存
④ 生token生成

■ plainTextToken

API通信で使う生token

━━━━━━━━━━━━━━━━━━━
■ routes/api.php 現在状態
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

━━━━━━━━━━━━━━━━━━━
■ Postmanログイン確認
━━━━━━━━━━━━━━━━━━━

送信：

POST http://localhost:8000/api/login

Body：

{
  "email": "test@example.com",
  "password": "password"
}

■ 成功レスポンス

{
  "message": "ログイン成功",
  "token": "1|xxxxxxxxxxxx"
}

━━━━━━━━━━━━━━━━━━━
■ personal_access_tokens 確認
━━━━━━━━━━━━━━━━━━━

実行：

DB::table('personal_access_tokens')->get();

確認できた内容：

tokenable_type
= App\Models\User

tokenable_id
= 1

name
= login-token

■ 超重要理解

Postmanへ返した：

1|xxxxxxxx

は：

生token

DB保存されるのは：

ハッシュtoken

■ なぜ？

DB漏洩時でも
tokenそのものを盗まれないため

━━━━━━━━━━━━━━━━━━━
■ 現在のDB状態
━━━━━━━━━━━━━━━━━━━

存在するテーブル：

users
teams
players
personal_access_tokens

■ personal_access_tokens の役割

ログインtoken保存

━━━━━━━━━━━━━━━━━━━
■ Day5 完了時点の到達状態
━━━━━━━━━━━━━━━━━━━

Laravel Sanctum導入完了
↓
ログインAPI完成
↓
token発行成功
↓
token DB保存成功
↓
Postmanログイン成功

━━━━━━━━━━━━━━━━━━━
■ 次回 Day6 でやること
━━━━━━━━━━━━━━━━━━━

・ログアウトAPI
・ログイン中ユーザー取得API
・auth:sanctum middleware理解