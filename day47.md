

📘 追加Day2 完全まとめ
（teams に invite_code 追加）

━━━━━━━━━━━━━━━━━━━
■ 今回の目的
━━━━━━━━━━━━━━━━━━━

チーム参加機能のために：

invite_code

をDBで扱えるようにする。

最終目的：

invite_code
↓
Team特定
↓
users.team_id 決定
↓
チーム所属

を可能にすること。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した超重要本質
━━━━━━━━━━━━━━━━━━━

本当に所属を決めているのは：

users.team_id

である。

しかし：

team_id を直接入力

させるのは危険。

理由：

・他チームへ勝手参加可能
・id総当たり可能
・内部DB構造露出

そのため：

invite_code

を中間に置く。

つまり：

invite_code
↓
Team検索
↓
Team.id取得
↓
users.team_id 保存

という流れ。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した invite_code の本質
━━━━━━━━━━━━━━━━━━━

invite_code は：

Team参加の鍵

である。

さらに本質として：

invite_code 自体が所属を決める

のではなく、

invite_code
↓
Team特定
↓
Team.id取得
↓
users.team_id 保存

によって所属が決まる。

━━━━━━━━━━━━━━━━━━━
■ なぜ teams テーブルに保存するのか
━━━━━━━━━━━━━━━━━━━

理由：

1チームにつき1つの参加コード

だから。

つまり：

invite_code は Team の属性

である。

User の属性ではない。

━━━━━━━━━━━━━━━━━━━
■ 今回作成した migration
━━━━━━━━━━━━━━━━━━━

実行コマンド：

php artisan make:migration add_invite_code_to_teams_table --table=teams

作成ファイル：

database/migrations/
2026_05_27_083847_add_invite_code_to_teams_table.php

━━━━━━━━━━━━━━━━━━━
■ 今回書いた migration コード
━━━━━━━━━━━━━━━━━━━

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::table('teams', function (Blueprint $table) {

            $table->string('invite_code')->unique();

        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::table('teams', function (Blueprint $table) {

            $table->dropUnique(['invite_code']);
            $table->dropColumn('invite_code');

        });
    }
};

━━━━━━━━━━━━━━━━━━━
■ 今回理解した migration 本質
━━━━━━━━━━━━━━━━━━━

migration は：

DB変更の設計図

である。

つまり：

Schema::table()

を書いただけでは、

本物DBは変化しない

実際には：

php artisan migrate

を実行して初めて：

本物DB変更

が行われる。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した up()
━━━━━━━━━━━━━━━━━━━

$table->string('invite_code')->unique();

意味：

teams テーブルへ
invite_code カラム追加

さらに：

->unique()

によって：

invite_code 重複禁止

になる。

理由：

invite_code
↓
Team特定

をするため。

もし重複すると：

どのTeamかわからなくなる

━━━━━━━━━━━━━━━━━━━
■ 今回理解した down()
━━━━━━━━━━━━━━━━━━━

down() は：

rollback時の逆処理

である。

━━━━━━━━━━━━━━━━━━━
■ dropUnique の意味
━━━━━━━━━━━━━━━━━━━

$table->dropUnique(['invite_code']);

意味：

invite_code の
unique制約解除

つまり：

重複禁止ルール削除

━━━━━━━━━━━━━━━━━━━
■ dropColumn の意味
━━━━━━━━━━━━━━━━━━━

$table->dropColumn('invite_code');

意味：

invite_code カラム自体削除

━━━━━━━━━━━━━━━━━━━
■ なぜ dropUnique → dropColumn の順なのか
━━━━━━━━━━━━━━━━━━━

理由：

制約付きカラムを
そのまま削除すると危険

だから。

そのため：

制約解除
↓
カラム削除

の順番。

━━━━━━━━━━━━━━━━━━━
■ 今回実行した migrate
━━━━━━━━━━━━━━━━━━━

php artisan migrate

実行結果：

migrationファイル読込
↓
Schema::table 実行
↓
teams テーブル変更
↓
invite_code カラム追加
↓
unique制約追加

━━━━━━━━━━━━━━━━━━━
■ 今回理解した超重要本質
━━━━━━━━━━━━━━━━━━━

migrationファイル
≠ DB実体

である。

本当にDBを変えるのは：

php artisan migrate

━━━━━━━━━━━━━━━━━━━
■ 今回のDB確認
━━━━━━━━━━━━━━━━━━━

実行：

php artisan tinker

確認：

Schema::getColumnListing('teams');

結果：

[
    "id",
    "name",
    "created_at",
    "updated_at",
    "invite_code",
]

つまり：

teams.invite_code

が本当にDBへ追加された。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した Model の本質
━━━━━━━━━━━━━━━━━━━

Model は：

DB操作クラス

である。

つまり：

Team::create()
Team::where()
Team::find()

など：

DB保存
DB検索
DB取得

を担当する。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した Controller の本質
━━━━━━━━━━━━━━━━━━━

Controller は：

アプリの流れ
処理
ロジック

を書く場所。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した Str::random(8)
━━━━━━━━━━━━━━━━━━━

Str::random(8)

意味：

8文字ランダム文字列生成

例：

ABCD1234
X9YZ7KLM

━━━━━━━━━━━━━━━━━━━
■ なぜ Str::random(8) は Modelではないのか
━━━━━━━━━━━━━━━━━━━

理由：

DB操作ではなく
処理ロジック

だから。

つまり：

Controller側責務

━━━━━━━━━━━━━━━━━━━
■ 将来的な実際の流れ
━━━━━━━━━━━━━━━━━━━

Controller
↓
Str::random(8)
↓
invite_code生成
↓
Team::create()
↓
ModelがDB保存
↓
teams テーブルへ保存

━━━━━━━━━━━━━━━━━━━
■ 今回理解した unique制約の本質
━━━━━━━━━━━━━━━━━━━

randomでも：

理論上重複可能

である。

そのため：

アプリ側
↓
Str::random()

DB側
↓
unique制約

の二重防御になっている。

━━━━━━━━━━━━━━━━━━━
■ 現在のDB状態
━━━━━━━━━━━━━━━━━━━

存在テーブル：

users
teams
players
personal_access_tokens

現在の teams テーブル：

teams
├ id
├ name
├ invite_code
├ created_at
└ updated_at

━━━━━━━━━━━━━━━━━━━
■ 現在の追加ロードマップ進行位置
━━━━━━━━━━━━━━━━━━━

✅ 追加Day1 完了
✅ 追加Day2 完了

次：

追加Day3
users に role 追加

次回やること：

・users.role migration作成
・owner/member 管理
・既存ユーザー確認