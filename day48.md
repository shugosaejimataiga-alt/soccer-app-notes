

# 追加Day3 完全まとめ
（users.role 追加 / owner・member 管理）

━━━━━━━━━━━━━━━━━━━
# 今日の目的
━━━━━━━━━━━━━━━━━━━

users テーブルに：

role

カラムを追加し、

owner
member

をDBで管理できるようにする。

━━━━━━━━━━━━━━━━━━━
# 今日理解した超重要本質
━━━━━━━━━━━━━━━━━━━

今回最重要だったのは：

migration

と

Controller / API

の役割の違い。

━━━━━━━━━━━━━━━━━━━
① migration の役割
━━━━━━━━━━━━━━━━━━━

migration は：

DBの保存場所を設計する

もの。

つまり：

$table->string('role');

は：

role を保存する箱

をDBに追加している。

migration は：

何を保存できるか

を決める場所。

━━━━━━━━━━━━━━━━━━━
② Controller / API の役割
━━━━━━━━━━━━━━━━━━━

Controller や API は：

どんな値を保存するか

を決める。

つまり：

'role' => 'owner'

を書くのは、

実際の登録処理側。

━━━━━━━━━━━━━━━━━━━
# 超重要理解
━━━━━━━━━━━━━━━━━━━

今回理解したこと：

保存場所を作る
≠
どんな値を入れるか決める

migration：

role という箱を作る

Controller：

owner を入れる
member を入れる

━━━━━━━━━━━━━━━━━━━
# なぜ role カラム1つなのか
━━━━━━━━━━━━━━━━━━━

今回：

owner カラム
member カラム

を作らなかった理由。

もし：

is_owner
is_member

にすると：

両方 true
両方 false

など、

矛盾した状態が作れる。

だから今回は：

role

という1つの箱に：

owner
member

のどちらか1つだけを入れる設計。

━━━━━━━━━━━━━━━━━━━
# default('member') の本質
━━━━━━━━━━━━━━━━━━━

->default('member')

は：

DB側の安全装置

Controllerで role を書き忘れても：

member

を自動で入れる。

つまり：

Controller
= アプリ側ルール

DB default
= DB側保険

という二重安全設計。

━━━━━━━━━━━━━━━━━━━
# after('team_id') の意味
━━━━━━━━━━━━━━━━━━━

->after('team_id')

は：

role カラムを
team_id の後ろに置く

という意味。

これは：

見やすさ整理

目的。

━━━━━━━━━━━━━━━━━━━
# 今日作成した migration
━━━━━━━━━━━━━━━━━━━

作成：

php artisan make:migration add_role_to_users_table --table=users

作成されたファイル：

database/migrations/2026_05_28_195038_add_role_to_users_table.php

━━━━━━━━━━━━━━━━━━━
# 今日書いた migration コード
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
        Schema::table('users', function (Blueprint $table) {

            $table->string('role')
                  ->default('member')
                  ->after('team_id');

        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {

            $table->dropColumn('role');

        });
    }
};

━━━━━━━━━━━━━━━━━━━
# migrate 実行
━━━━━━━━━━━━━━━━━━━

実行：

php artisan migrate

結果：

users.role

が実際のDBに追加された。

━━━━━━━━━━━━━━━━━━━
# Tinker確認
━━━━━━━━━━━━━━━━━━━

起動：

php artisan tinker

━━━━━━━━━━━━━━━━━━━
# users確認
━━━━━━━━━━━━━━━━━━━

実行：

\App\Models\User::first();

確認結果：

role: "member"

━━━━━━━━━━━━━━━━━━━
# ここで理解した重要点
━━━━━━━━━━━━━━━━━━━

昔作ったユーザーなのに：

role: "member"

になっていた。

これは：

default('member')

がDB側で自動適用されたため。

つまり：

default はDBルール

━━━━━━━━━━━━━━━━━━━
# owner 保存確認
━━━━━━━━━━━━━━━━━━━

取得：

$user = \App\Models\User::find(1);

変更：

$user->role = 'owner';

保存：

$user->save();

再確認：

\App\Models\User::find(1);

結果：

role: "owner"

━━━━━━━━━━━━━━━━━━━
# 今日理解した Eloquent 超重要本質
━━━━━━━━━━━━━━━━━━━

$user->role = 'owner';

だけでは：

メモリ上変更

であり、

DBはまだ変わらない。

━━━━━━━━━━━━━━━━━━━

$user->save();

で初めて：

DB更新

される。

つまり：

オブジェクト変更
≠
DB保存

━━━━━━━━━━━━━━━━━━━
# 現在の users テーブルイメージ
━━━━━━━━━━━━━━━━━━━

id
name
email
password
team_id
role
created_at
updated_at

━━━━━━━━━━━━━━━━━━━
# 現在の状態
━━━━━━━━━━━━━━━━━━━

現在：

id=1

のユーザーは：

role = owner

になっている。

これは今後の：

Team管理
owner権限確認
ユーザー除外
owner譲渡
チーム削除

などで使うため、

owner のままでOK。