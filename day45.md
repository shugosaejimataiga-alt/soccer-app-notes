

📘 Players強化 Day10 完全版まとめ
（一覧取得へ team制限追加 + users.team_id NULL禁止化）

━━━━━━━━━━━━━━━━━━━
■ Day10 の目的
━━━━━━━━━━━━━━━━━━━

GET /players

に：

ログインユーザーの team_id 制限

を追加し、

自チームの選手だけ一覧表示

を実現する。

さらに今回：

users.team_id が NULL許可

になっていた問題も修正した。

━━━━━━━━━━━━━━━━━━━
■ Day10 開始時点の状態
━━━━━━━━━━━━━━━━━━━

Day9時点では：

show
update
delete

には Policy が適用済みだった。

つまり：

他チームPlayerの閲覧
編集
削除

は禁止できていた。

しかし：

GET /players

だけは：

全チームの選手一覧

が取得できる状態だった。

━━━━━━━━━━━━━━━━━━━
■ 今回の超重要目的
━━━━━━━━━━━━━━━━━━━

現在ログイン中ユーザーの：

team_id

だけ一覧取得する。

つまり：

チームごとのデータ分離

を実現する。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した超重要本質
━━━━━━━━━━━━━━━━━━━

一覧取得でも：

認可的制限

が必要。

show/update/delete だけ守っても：

一覧APIで全部見えたら意味がない

つまり：

一覧取得にも team 制限が必要

━━━━━━━━━━━━━━━━━━━
■ 今回理解した auth()->user()
━━━━━━━━━━━━━━━━━━━

auth()->user()

意味：

現在ログイン中のUserモデル取得

さらに：

auth()->user()->team_id

意味：

ログイン中Userの team_id

━━━━━━━━━━━━━━━━━━━
■ 今回追加した超重要コード
━━━━━━━━━━━━━━━━━━━

追加場所：

PlayerController.php
index()

追加コード：

$query->where('team_id', auth()->user()->team_id);

━━━━━━━━━━━━━━━━━━━
■ このコードの意味
━━━━━━━━━━━━━━━━━━━

$query->where('team_id', auth()->user()->team_id);

↓

WHERE team_id = ログインUser.team_id

つまり：

自チーム選手だけ取得

━━━━━━━━━━━━━━━━━━━
■ 今回理解した query() の本質
━━━━━━━━━━━━━━━━━━━

$query = Player::query();

意味：

playersテーブル検索準備開始

まだSQL実行ではない。

━━━━━━━━━━━━━━━━━━━
■ where() の本質
━━━━━━━━━━━━━━━━━━━

$query->where(...)

意味：

検索条件追加

━━━━━━━━━━━━━━━━━━━
■ get() の本質
━━━━━━━━━━━━━━━━━━━

$query->get()

意味：

ここで初めてSQL実行
DBアクセス
データ取得

━━━━━━━━━━━━━━━━━━━
■ 今回理解した index() の流れ
━━━━━━━━━━━━━━━━━━━

Player::query()
↓
where追加
↓
検索条件追加
↓
ソート追加
↓
get()実行
↓
DB取得

━━━━━━━━━━━━━━━━━━━
■ 今回最初 data: [] になった理由
━━━━━━━━━━━━━━━━━━━

最初：

{
    "data": []
}

になった。

原因調査した結果：

ログインUser.team_id = null

だった。

━━━━━━━━━━━━━━━━━━━
■ なぜ null だったのか
━━━━━━━━━━━━━━━━━━━

usersテーブル作成時：

team_id は存在していなかった

後から Day4 で：

users.team_id

を追加したため、

既存User：

test@example.com

には：

team_id が入っていなかった

━━━━━━━━━━━━━━━━━━━
■ 今回理解した nullable()
━━━━━━━━━━━━━━━━━━━

元migrationでは：

nullable()

が付いていた。

意味：

NULL許可

つまり：

team_id なしUser許可

状態。

━━━━━━━━━━━━━━━━━━━
■ 今回修正した既存User
━━━━━━━━━━━━━━━━━━━

Tinker：

$user = User::find(1);
$user->team_id = 1;
$user->save();

意味：

test@example.com を team_id=1 所属へ変更

━━━━━━━━━━━━━━━━━━━
■ 今回理解した Tinker の本質
━━━━━━━━━━━━━━━━━━━

php artisan tinker

意味：

Laravel用PHP実験環境

ここでは：

User::find(1)

などPHPコードを書く。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した通常ターミナルとの違い
━━━━━━━━━━━━━━━━━━━

通常ターミナル：

php artisan make:migration

など：

Laravelコマンド実行場所

Tinker：

User::find(1)

など：

PHPコード実験場所

━━━━━━━━━━━━━━━━━━━
■ 今回発見した重要問題
━━━━━━━━━━━━━━━━━━━

現在：

users.team_id

は：

NULL許可

だった。

しかし現在は：

認証
認可
一覧取得

すべてで：

team_id

を使っている。

つまり：

Userは必ずTeam所属

であるべき。

━━━━━━━━━━━━━━━━━━━
■ 今回確認した NULL User
━━━━━━━━━━━━━━━━━━━

Tinker：

User::whereNull('team_id')->get();

結果：

all: []

意味：

NULL User は存在しない

━━━━━━━━━━━━━━━━━━━
■ 今回作成した migration
━━━━━━━━━━━━━━━━━━━

作成：

php artisan make:migration change_users_team_id_to_not_nullable

━━━━━━━━━━━━━━━━━━━
■ 今回理解した Schema::table()
━━━━━━━━━━━━━━━━━━━

Schema::table('users', function (Blueprint $table)

意味：

usersテーブル変更

━━━━━━━━━━━━━━━━━━━
■ 今回理解した nullable(false)
━━━━━━━━━━━━━━━━━━━

$table->foreignId('team_id')->nullable(false)->change();

意味：

team_id の NULL禁止

つまり：

team_id 必須化

━━━━━━━━━━━━━━━━━━━
■ 今回理解した change()
━━━━━━━━━━━━━━━━━━━

change()

意味：

既存カラム変更

新規作成ではない。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した nullable()
━━━━━━━━━━━━━━━━━━━

$table->foreignId('team_id')->nullable()->change();

意味：

rollback時
NULL許可へ戻す

━━━━━━━━━━━━━━━━━━━
■ 今回理解した migration の本質
━━━━━━━━━━━━━━━━━━━

up()
↓
変更適用

down()
↓
元へ戻す

つまり：

DB変更履歴管理

━━━━━━━━━━━━━━━━━━━
■ 今回実行した migrate
━━━━━━━━━━━━━━━━━━━

php artisan migrate

成功：

users.team_id
↓
NOT NULL化成功

━━━━━━━━━━━━━━━━━━━
■ 今回一時的に使った重要デバッグ
━━━━━━━━━━━━━━━━━━━

return response()->json([
    'login_user_id' => auth()->user()->id,
    'login_user_team_id' => auth()->user()->team_id,
    'players_count' => Player::where('team_id', auth()->user()->team_id)
        ->where('is_active', true)
        ->count(),
]);

意味：

現在ログインUser
team_id
取得件数

確認。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した optimize:clear
━━━━━━━━━━━━━━━━━━━

実行：

php artisan optimize:clear

意味：

Laravelキャッシュ削除

今回：

古い状態

を見ていた可能性があり、

実行後正常動作した。

━━━━━━━━━━━━━━━━━━━
■ 現在の index() 本質
━━━━━━━━━━━━━━━━━━━

現在の GET /players は：

自チーム
+
is_active=true
+
検索
+
絞り込み
+
ソート

を同時に実現している。

━━━━━━━━━━━━━━━━━━━
■ 現在の PlayerController.php 超重要部分
━━━━━━━━━━━━━━━━━━━

$query = Player::query();

$query->where('team_id', auth()->user()->team_id);

$query->where('is_active', true);

━━━━━━━━━━━━━━━━━━━
■ 現在達成できたこと
━━━━━━━━━━━━━━━━━━━

ログインUser.team_id
↓
そのチームの選手だけ一覧取得

さらに：

論理削除済み選手除外

も維持できている。

