

■ Players強化ロードマップ Day2 完全まとめ
目的：
players.team_id を teams.id とDBレベルで正式接続し、
Player belongsTo Team を成立させる。

━━━━━━━━━━━━━━━━━━━
■ Day2でやったこと
━━━━━━━━━━━━━━━━━━━

・players.team_id に外部キー制約追加
・team_id に index追加
・teamsデータ作成
・Model理解
・fillable理解
・Mass Assignment理解
・tinker理解
・migration失敗原因理解

━━━━━━━━━━━━━━━━━━━
■ 現在のDB構造
━━━━━━━━━━━━━━━━━━━

teams
 ├ id
 ├ name
 ├ created_at
 └ updated_at

players
 ├ id
 ├ team_id
 ├ name
 ├ position
 ├ photo_url
 ├ is_active
 ├ created_at
 └ updated_at

━━━━━━━━━━━━━━━━━━━
■ Day2で理解した本質
━━━━━━━━━━━━━━━━━━━

以前：

players.team_id
↓
ただの数字

現在：

players.team_id
↓
teams.id を参照する正式な外部キー

つまり：

Player belongsTo Team

をDBレベルで保証する状態になった。

━━━━━━━━━━━━━━━━━━━
■ migration理解
━━━━━━━━━━━━━━━━━━━

migration =
DB構造変更設計書

Laravel内部：

migration PHPコード
↓
SQL生成
↓
MySQL送信

━━━━━━━━━━━━━━━━━━━
■ migrationは積み重なる
━━━━━━━━━━━━━━━━━━━

過去migrationは基本編集しない。

新しいDB変更が必要なら：

新migration追加

で管理する。

例：

① create_players_table
↓
players作成

② add_foreign_key_to_players_team_id
↓
外部キー追加

━━━━━━━━━━━━━━━━━━━
■ 実行したmigration作成コマンド
━━━━━━━━━━━━━━━━━━━

php artisan make:migration add_foreign_key_to_players_team_id --table=players

意味：

make:migration
↓
migration作成

add_foreign_key_to_players_team_id
↓
players.team_idへ外部キー追加migration名

--table=players
↓
既存playersテーブル変更migration

━━━━━━━━━━━━━━━━━━━
■ 作成されたmigration
━━━━━━━━━━━━━━━━━━━

database/migrations/
2026_05_08_120502_add_foreign_key_to_players_team_id.php

━━━━━━━━━━━━━━━━━━━
■ migrationコード最終形
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

            $table->foreign('team_id')
                  ->references('id')
                  ->on('teams');

            $table->index('team_id');

        });
    }

    public function down(): void
    {
        Schema::table('players', function (Blueprint $table) {

            $table->dropForeign(['team_id']);

        });
    }
};

━━━━━━━━━━━━━━━━━━━
■ コード理解
━━━━━━━━━━━━━━━━━━━

return new class extends Migration
↓
Migrationクラス継承した無名クラス

public function up()
↓
migration適用時処理

public function down()
↓
rollback時の取り消し処理

Schema::table('players')
↓
playersテーブル変更

Blueprint $table
↓
テーブル変更用オブジェクト

━━━━━━━━━━━━━━━━━━━
■ 外部キー理解
━━━━━━━━━━━━━━━━━━━

$table->foreign('team_id')
↓
players.team_id に外部キー設定開始

->references('id')
->on('teams')
↓
teams.id を参照

つまり：

players.team_id
↓
teams.id

━━━━━━━━━━━━━━━━━━━
■ 親テーブル / 子テーブル理解
━━━━━━━━━━━━━━━━━━━

親：
teams

子：
players

理由：

players.team_id
↓
teams.id を参照

参照される側
↓
親

参照する側
↓
子

━━━━━━━━━━━━━━━━━━━
■ index理解
━━━━━━━━━━━━━━━━━━━

$table->index('team_id');

意味：

team_id検索高速化用索引作成

目的：

WHERE team_id = 1

などの検索高速化。

index =
DB内部検索目次

━━━━━━━━━━━━━━━━━━━
■ rollback理解
━━━━━━━━━━━━━━━━━━━

php artisan migrate
↓
up()実行

php artisan migrate:rollback
↓
down()実行

今回：

up()
↓
外部キー追加

down()
↓
外部キー制約削除

重要：

team_idカラム削除ではない。

削除するのは：

外部キー制約ルール

━━━━━━━━━━━━━━━━━━━
■ 外部キーエラー理解
━━━━━━━━━━━━━━━━━━━

最初のmigrate失敗：

Cannot add or update a child row:
a foreign key constraint fails

意味：

外部キー制約違反。

原因：

players.team_id に存在する値と、
teams.id に存在する値が一致しなかった。

つまり：

players.team_id = 1
なのに
teams.id = 1
が存在しなかった可能性。

━━━━━━━━━━━━━━━━━━━
■ child row理解
━━━━━━━━━━━━━━━━━━━

child row
↓
子テーブル側データ

今回：

players

━━━━━━━━━━━━━━━━━━━
■ Teamモデル作成
━━━━━━━━━━━━━━━━━━━

実行：

php artisan make:model Team

作成：

app/Models/Team.php

意味：

teamsテーブル操作用Model

━━━━━━━━━━━━━━━━━━━
■ migrationとModelの違い
━━━━━━━━━━━━━━━━━━━

migration
↓
DB構造変更

Model
↓
DBデータ操作

矮小化イメージ：

migration
↓
テーブル土台操作

Model
↓
テーブル中身操作

━━━━━━━━━━━━━━━━━━━
■ tinker理解
━━━━━━━━━━━━━━━━━━━

php artisan tinker

意味：

Laravel対話実験環境

できること：

・Model実行
・DBデータ追加
・取得
・更新
・削除

━━━━━━━━━━━━━━━━━━━
■ Mass Assignment理解
━━━━━━━━━━━━━━━━━━━

実行したコード：

\App\Models\Team::create([
    'name' => 'Default Team'
]);

create([...])
↓
配列まとめ代入

Laravelでは：

Mass Assignment

と呼ぶ。

━━━━━━━━━━━━━━━━━━━
■ fillable理解
━━━━━━━━━━━━━━━━━━━

最初エラー：

MassAssignmentException

意味：

まとめ代入許可されていない。

修正：

app/Models/Team.php

<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Team extends Model
{
    protected $fillable = [
        'name'
    ];
}

fillable =
配列まとめ代入許可カラム一覧

━━━━━━━━━━━━━━━━━━━
■ create()内部理解
━━━━━━━━━━━━━━━━━━━

Team::create([
    'name' => 'Default Team'
]);

内部：

① Teamモデル使用
↓
② 配列受け取り
↓
③ fillable確認
↓
④ SQL生成

INSERT INTO teams ...

↓
⑤ MySQL送信
↓
⑥ DB保存

━━━━━━━━━━━━━━━━━━━
■ 最終的に成功したmigrate
━━━━━━━━━━━━━━━━━━━

php artisan migrate

結果：

2026_05_08_120502_add_foreign_key_to_players_team_id ... DONE

━━━━━━━━━━━━━━━━━━━
■ 現在成立していること
━━━━━━━━━━━━━━━━━━━

teams.id = 1
↓
players.team_id = 1

が正式接続。

現在DBは：

存在しないteam_idを禁止

する状態になった。

━━━━━━━━━━━━━━━━━━━
■ Day2 完了状態
━━━━━━━━━━━━━━━━━━━

・Player belongsTo Team がDBレベルで成立
・外部キー理解
・index理解
・rollback理解
・Model理解
・Mass Assignment理解
・fillable理解
・tinker理解
・migrationとModelの役割違い理解