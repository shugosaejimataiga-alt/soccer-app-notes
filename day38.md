
■ Players強化ロードマップ Day3 完全まとめ

━━━━━━━━━━━━━━━━━━━
■ Day3の目的
━━━━━━━━━━━━━━━━━━━

検索・絞り込み性能を意識した
DB構造へ進化させる。

目的：

・usersテーブル構造理解
・team_id追加準備理解
・index理解
・検索高速化理解
・migration積み重ね思想理解

━━━━━━━━━━━━━━━━━━━
■ Day3でやったこと
━━━━━━━━━━━━━━━━━━━

① usersテーブル確認
② Schema / Blueprint理解
③ migrationとModel役割整理
④ players.name index追加
⑤ players.position index追加
⑥ up/down/migrate/rollback理解

━━━━━━━━━━━━━━━━━━━
■ users migration確認
━━━━━━━━━━━━━━━━━━━

確認したファイル：

database/migrations/
xxxx_xx_xx_create_users_table.php

確認コード：

Schema::create('users', function (Blueprint $table)

意味：

usersテーブル作成。

━━━━━━━━━━━━━━━━━━━
■ usersテーブル現在構造
━━━━━━━━━━━━━━━━━━━

users
 ├ id
 ├ name
 ├ email
 ├ email_verified_at
 ├ password
 ├ remember_token
 ├ created_at
 └ updated_at

━━━━━━━━━━━━━━━━━━━
■ usersテーブルの役割
━━━━━━━━━━━━━━━━━━━

ログインユーザー管理。

主に：

・name
・email
・password

を保存。

━━━━━━━━━━━━━━━━━━━
■ 今後必要になるもの
━━━━━━━━━━━━━━━━━━━

users.team_id

理由：

ログイン中ユーザーが
どのteam所属か判断するため。

将来的に：

user.team_id
↓
自チームplayersだけ取得

を行う。

━━━━━━━━━━━━━━━━━━━
■ Schema理解
━━━━━━━━━━━━━━━━━━━

Schema =
LaravelのDB構造操作機能。

例：

Schema::create(...)
Schema::table(...)

役割：

・テーブル作成
・テーブル変更

━━━━━━━━━━━━━━━━━━━
■ :: 理解
━━━━━━━━━━━━━━━━━━━

:: =
クラス機能呼び出し。

例：

Schema::create()
Team::create()

意味：

クラスの機能を使用。

━━━━━━━━━━━━━━━━━━━
■ Blueprint $table 理解
━━━━━━━━━━━━━━━━━━━

function (Blueprint $table)

意味：

$tableを使って
テーブル中身を定義する。

$table =
テーブル設計用オブジェクト。

━━━━━━━━━━━━━━━━━━━
■ 例
━━━━━━━━━━━━━━━━━━━

$table->id();
↓
idカラム追加

$table->string('name');
↓
文字列nameカラム追加

━━━━━━━━━━━━━━━━━━━
■ Laravel内部で起きていること
━━━━━━━━━━━━━━━━━━━

migration(PHP)
↓
LaravelがSQLへ変換
↓
MySQLへ送信
↓
DB変更

━━━━━━━━━━━━━━━━━━━
■ migrationとModelの違い
━━━━━━━━━━━━━━━━━━━

migration
↓
DB構造変更

Model
↓
DBデータ操作

━━━━━━━━━━━━━━━━━━━
■ migration役割
━━━━━━━━━━━━━━━━━━━

・テーブル作成
・カラム追加
・外部キー追加
・index追加

つまり：

DB骨組み操作。

━━━━━━━━━━━━━━━━━━━
■ Model役割
━━━━━━━━━━━━━━━━━━━

DB中身操作。

例：

Team::create([...])

━━━━━━━━━━━━━━━━━━━
■ migrationは積み重ねる
━━━━━━━━━━━━━━━━━━━

重要：

過去migrationを基本編集しない。

新変更が必要なら：

新migration追加。

━━━━━━━━━━━━━━━━━━━
■ 理由
━━━━━━━━━━━━━━━━━━━

migration =
DB変更履歴。

だから：

create
↓
追加
↓
追加
↓
追加

という形で積み重なる。

━━━━━━━━━━━━━━━━━━━
■ players.name index追加
━━━━━━━━━━━━━━━━━━━

実行：

php artisan make:migration add_index_to_players_name --table=players

意味：

playersテーブルへ
name index追加migration作成。

━━━━━━━━━━━━━━━━━━━
■ 作成されたファイル
━━━━━━━━━━━━━━━━━━━

database/migrations/
2026_05_11_082144_add_index_to_players_name.php

━━━━━━━━━━━━━━━━━━━
■ 最終コード
━━━━━━━━━━━━━━━━━━━

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('players', function (Blueprint $table) {

            $table->index('name');

        });
    }

    public function down(): void
    {
        Schema::table('players', function (Blueprint $table) {

            $table->dropIndex(['name']);

        });
    }
};

━━━━━━━━━━━━━━━━━━━
■ index理解
━━━━━━━━━━━━━━━━━━━

index =
DB検索用目次。

目的：

検索高速化。

━━━━━━━━━━━━━━━━━━━
■ indexが無い場合
━━━━━━━━━━━━━━━━━━━

DBが：

1行ずつ全部確認。

これを：

フルスキャン

という。

━━━━━━━━━━━━━━━━━━━
■ indexがある場合
━━━━━━━━━━━━━━━━━━━

DB内部に：

検索目次

が作られる。

結果：

検索高速化。

━━━━━━━━━━━━━━━━━━━
■ name index目的
━━━━━━━━━━━━━━━━━━━

今後：

GET /players?name=田中

などを高速化するため。

━━━━━━━━━━━━━━━━━━━
■ players.position index追加
━━━━━━━━━━━━━━━━━━━

実行：

php artisan make:migration add_index_to_players_position --table=players

━━━━━━━━━━━━━━━━━━━
■ 作成されたファイル
━━━━━━━━━━━━━━━━━━━

database/migrations/
2026_05_11_225741_add_index_to_players_position.php

━━━━━━━━━━━━━━━━━━━
■ 最終コード
━━━━━━━━━━━━━━━━━━━

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('players', function (Blueprint $table) {

            $table->index('position');

        });
    }

    public function down(): void
    {
        Schema::table('players', function (Blueprint $table) {

            $table->dropIndex(['position']);

        });
    }
};

━━━━━━━━━━━━━━━━━━━
■ なぜpositionにもindex？
━━━━━━━━━━━━━━━━━━━

今後：

GET /players?position=GK

などの絞り込みを行うため。

━━━━━━━━━━━━━━━━━━━
■ ただし重要
━━━━━━━━━━━━━━━━━━━

positionは：

GK / DF / MF / FW

の4種類しかない。

つまり：

種類が少ないカラム。

こういうindexは
効果が弱い場合もある。

━━━━━━━━━━━━━━━━━━━
■ それでも追加した理由
━━━━━━━━━━━━━━━━━━━

今後：

team_id + position

で検索する可能性が高いため。

━━━━━━━━━━━━━━━━━━━
■ up() 理解
━━━━━━━━━━━━━━━━━━━

up() =
DB変更適用。

例：

$table->index('name');

━━━━━━━━━━━━━━━━━━━
■ down() 理解
━━━━━━━━━━━━━━━━━━━

down() =
rollback時取り消し。

例：

$table->dropIndex(['name']);

━━━━━━━━━━━━━━━━━━━
■ migrate理解
━━━━━━━━━━━━━━━━━━━

実行：

php artisan migrate

意味：

up()実行。

DBへ変更適用。

━━━━━━━━━━━━━━━━━━━
■ rollback理解
━━━━━━━━━━━━━━━━━━━

実行：

php artisan migrate:rollback

意味：

down()実行。

変更取り消し。

━━━━━━━━━━━━━━━━━━━
■ 超重要理解
━━━━━━━━━━━━━━━━━━━

php artisan migrate
↓
up()だけ実行

php artisan migrate:rollback
↓
down()だけ実行

━━━━━━━━━━━━━━━━━━━
■ dropIndex(['name']) 理解
━━━━━━━━━━━━━━━━━━━

Laravel内部で：

players_name_index

のようなindex名が自動生成される。

['name'] と書くことで：

nameカラム用index

をLaravelが自動解決。

━━━━━━━━━━━━━━━━━━━
■ 現在の重要migration構造
━━━━━━━━━━━━━━━━━━━

database/migrations
├ create_players_table.php
├ create_teams_table.php
├ add_foreign_key_to_players_team_id.php
├ add_index_to_players_name.php
└ add_index_to_players_position.php

━━━━━━━━━━━━━━━━━━━
■ 現在のplayersテーブル構造
━━━━━━━━━━━━━━━━━━━

players
 ├ id
 ├ team_id
 ├ name
 ├ position
 ├ photo_url
 ├ is_active
 ├ created_at
 └ updated_at

追加されたindex：

・team_id index
・name index
・position index

━━━━━━━━━━━━━━━━━━━
■ Day3 完了状態
━━━━━━━━━━━━━━━━━━━

・usersテーブル理解
・Schema理解
・Blueprint理解
・migrationとModel違い理解
・index理解
・検索高速化理解
・migration積み重ね思想理解
・up/down理解
・migrate/rollback理解
・players.name index追加完了
・players.position index追加完了

━━━━━━━━━━━━━━━━━━━
■ Day4でやること
━━━━━━━━━━━━━━━━━━━

目的：

UserとTeamを正式接続。

実施内容：

・users.team_id追加
・users → team リレーション作成
・team → users リレーション作成

目標：

ログインユーザーが
どのteam所属か判断できる状態へ進める。