

■ Players強化ロードマップ Day4 完全まとめ

━━━━━━━━━━━━━━━━━━━
■ Day4の目的
━━━━━━━━━━━━━━━━━━━

User と Team を正式接続する。

目的：

・ログインUserがどのTeam所属か判断できるようにする
・将来的に「自TeamのPlayersのみ取得」を行うため

最終的にやりたい流れ：

ログインUser
↓
所属Team取得
↓
そのTeamのPlayersだけ取得

━━━━━━━━━━━━━━━━━━━
■ Day4でやったこと
━━━━━━━━━━━━━━━━━━━

① users.team_id 追加 migration作成
② usersテーブルへteam_id追加
③ 外部キー制約追加
④ User belongsTo Team
⑤ Team hasMany Users
⑥ belongsTo / hasMany 理解
⑦ User / Team / Player の役割整理

━━━━━━━━━━━━━━━━━━━
■ 現在のDB構造
━━━━━━━━━━━━━━━━━━━

teams
├ id
├ name
├ created_at
└ updated_at

users
├ id
├ name
├ email
├ password
├ team_id
├ created_at
└ updated_at

━━━━━━━━━━━━━━━━━━━
■ team_id の意味
━━━━━━━━━━━━━━━━━━━

User がどのTeam所属かを表す。

例：

User
├ 山田監督
└ team_id = 1

Team
├ id = 1
└ AAA高校

━━━━━━━━━━━━━━━━━━━
■ migration作成
━━━━━━━━━━━━━━━━━━━

実行コマンド：

php artisan make:migration add_team_id_to_users_table --table=users

生成ファイル：

database/migrations/
2026_05_14_034554_add_team_id_to_users_table.php

━━━━━━━━━━━━━━━━━━━
■ migration最終コード
━━━━━━━━━━━━━━━━━━━

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {

            $table->foreignId('team_id')
                  ->nullable()
                  ->constrained('teams');

        });
    }

    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {

            $table->dropForeign(['team_id']);
            $table->dropColumn('team_id');

        });
    }
};

━━━━━━━━━━━━━━━━━━━
■ foreignId('team_id') 理解
━━━━━━━━━━━━━━━━━━━

$table->foreignId('team_id')

意味：

usersテーブルへ
team_idカラム追加

━━━━━━━━━━━━━━━━━━━
■ nullable() 理解
━━━━━━━━━━━━━━━━━━━

->nullable()

意味：

NULL許可

つまり：

team_id が空でもOK

━━━━━━━━━━━━━━━━━━━
■ constrained('teams') 理解
━━━━━━━━━━━━━━━━━━━

->constrained('teams')

意味：

外部キー制約追加

つまり：

users.team_id
↓
teams.id

を接続。

━━━━━━━━━━━━━━━━━━━
■ 外部キー制約の意味
━━━━━━━━━━━━━━━━━━━

存在しないteam_id禁止。

例えば：

team_id = 999

でも、

teams.id = 999

が存在しなければ保存不可。

━━━━━━━━━━━━━━━━━━━
■ 親子関係理解
━━━━━━━━━━━━━━━━━━━

teams = 親
users = 子

理由：

users.team_id
↓
teams.id

を参照しているため。

━━━━━━━━━━━━━━━━━━━
■ migrate実行
━━━━━━━━━━━━━━━━━━━

実行：

php artisan migrate

意味：

up() 実行
↓
DBへ反映

━━━━━━━━━━━━━━━━━━━
■ down() 理解
━━━━━━━━━━━━━━━━━━━

① 外部キー削除

$table->dropForeign(['team_id']);

意味：

team_id の外部キー制約解除

② カラム削除

$table->dropColumn('team_id');

意味：

team_idカラム自体削除

━━━━━━━━━━━━━━━━━━━
■ なぜ順番重要？
━━━━━━━━━━━━━━━━━━━

① 外部キー解除
↓
② カラム削除

先にカラム削除すると：

外部キーが残っていてエラーになる。

━━━━━━━━━━━━━━━━━━━
■ User / Team / Player の役割整理
━━━━━━━━━━━━━━━━━━━

■ User

ログインする人。

例：

・監督
・コーチ
・管理者

■ Team

所属チーム。

例：

・AAA高校
・レアル
・バルサ

■ Player

サッカー選手データ。

例：

・田中
・佐藤
・鈴木

━━━━━━━━━━━━━━━━━━━
■ belongsTo 理解
━━━━━━━━━━━━━━━━━━━

User.php に追加：

public function team()
{
    return $this->belongsTo(Team::class);
}

意味：

1 User
↓
所属Teamは1つ

━━━━━━━━━━━━━━━━━━━
■ belongsTo の役割
━━━━━━━━━━━━━━━━━━━

Laravelへ：

User は Team に所属

を教える。

すると：

$user->team

で所属Team取得可能になる。

━━━━━━━━━━━━━━━━━━━
■ Laravel内部で起きること
━━━━━━━━━━━━━━━━━━━

$user->team

↓

Laravel内部：

select * from teams
where teams.id = users.team_id

を自動実行。

━━━━━━━━━━━━━━━━━━━
■ Team::class 理解
━━━━━━━━━━━━━━━━━━━

Team::class

意味：

Teamモデル

であり、

teams.id

ではない。

━━━━━━━━━━━━━━━━━━━
■ Laravel規約理解
━━━━━━━━━━━━━━━━━━━

public function team()

を見るとLaravelは：

users.team_id

を推測。

さらに：

belongsTo(Team::class)

を見ると：

teams.id

を接続先として推測。

最終的に：

users.team_id
↓
teams.id

を自動組み立て。

━━━━━━━━━━━━━━━━━━━
■ hasMany 理解
━━━━━━━━━━━━━━━━━━━

Team.php に追加：

public function users()
{
    return $this->hasMany(User::class);
}

意味：

1 Team
↓
複数Users所属

━━━━━━━━━━━━━━━━━━━
■ users() が複数形な理由
━━━━━━━━━━━━━━━━━━━

返ってくるのが
複数Usersだから。

━━━━━━━━━━━━━━━━━━━
■ team() が単数形な理由
━━━━━━━━━━━━━━━━━━━

返ってくるのが
1つのTeamだから。

━━━━━━━━━━━━━━━━━━━
■ Team側から取得
━━━━━━━━━━━━━━━━━━━

$team->users

↓

そのTeam所属のUsers一覧を取得可能。

━━━━━━━━━━━━━━━━━━━
■ User側から取得
━━━━━━━━━━━━━━━━━━━

$user->team

↓

そのUser所属のTeamを取得可能。

━━━━━━━━━━━━━━━━━━━
■ 現在のModel状態
━━━━━━━━━━━━━━━━━━━

■ User.php

public function team()
{
    return $this->belongsTo(Team::class);
}

■ Team.php

public function users()
{
    return $this->hasMany(User::class);
}

━━━━━━━━━━━━━━━━━━━
■ Day4 完了状態
━━━━━━━━━━━━━━━━━━━

・users.team_id追加完了
・外部キー制約追加完了
・User belongsTo Team 完了
・Team hasMany Users 完了
・ログインUserの所属Team取得土台完成
・User / Team / Player の役割整理完了
・belongsTo / hasMany 理解
・Laravel規約理解

━━━━━━━━━━━━━━━━━━━
■ 今後の流れ
━━━━━━━━━━━━━━━━━━━

Day5：

Laravel Sanctum導入
↓
ログインAPI作成
↓
ログインUser取得
↓
Auth::user()->team

で、

ログインUser所属Team判定

へ進む。