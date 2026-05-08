

■ Players強化ロードマップ Day1 完全まとめ

【目的】

Players機能を「チーム所属型」にするため、
まず teams テーブルを作成した。

今後、

User
↓
Team
↓
Player

という構造にしていく土台。

━━━━━━━━━━━━━━━━━━━
■ 今日理解した超重要概念
━━━━━━━━━━━━━━━━━━━

【Docker構造】

現在の環境：

Windows
↓
Docker Desktop
↓
Docker
 ├ appコンテナ
 │    ├ Linux
 │    ├ PHP
 │    ├ Laravel
 │    └ artisan
 │
 ├ webコンテナ
 │    ├ Linux
 │    └ nginx
 │
 └ dbコンテナ
      ├ Linux
      └ MySQL

理解したこと：

・コンテナごとに小さいLinux環境がある
・Laravelは app(PHP)コンテナ内に存在
・MySQLは dbコンテナ内に存在
・nginxは webコンテナ内に存在

━━━━━━━━━━━━━━━━━━━
■ docker compose exec app bash の意味
━━━━━━━━━━━━━━━━━━━

docker compose
↓
Docker Compose使用

exec
↓
起動中コンテナでコマンド実行

app
↓
appコンテナ指定

bash
↓
Linuxターミナル起動

つまり：

「appコンテナ内のLinuxへ入る」

実行後：

root@1cdf4c222cb5:/var/www#

になった。

理解したこと：

root
↓
Linux管理者ユーザー

1cdf4c222cb5
↓
コンテナID

/var/www
↓
現在フォルダ

━━━━━━━━━━━━━━━━━━━
■ Linuxコマンド理解
━━━━━━━━━━━━━━━━━━━

ls
↓
現在フォルダの中を見る

cd
↓
フォルダ移動

今回：

ls

を実行して、

Dockerfile
composer.phar
docker
docker-compose.yml
laravel

を確認。

つまり：

Laravel本体は
/var/www/laravel
に存在すると理解。

その後：

cd laravel

でLaravelフォルダへ移動。

━━━━━━━━━━━━━━━━━━━
■ artisanとは何か
━━━━━━━━━━━━━━━━━━━

artisan は：

Laravel専用操作ファイル

Laravelプロジェクト作成時に自動生成される。

artisan は：

「Laravelへの命令入口」

イメージ：

php artisan migrate

↓
Laravelへ
「migration実行して」
と命令

理解した重要点：

docker compose exec
↓
Dockerへの命令

artisan
↓
Laravelへの命令

役割が違う。

━━━━━━━━━━━━━━━━━━━
■ migrationとは何か
━━━━━━━━━━━━━━━━━━━

migration =
PHPコードでDB構造を管理する仕組み

通常MySQLでは：

CREATE TABLE teams (...)

などSQLを書く。

Laravelでは：

Schema::create('teams', ...)

などPHPコードで書ける。

Laravel内部で：

PHPコード
↓
SQL変換
↓
MySQLへ送信

を行う。

━━━━━━━━━━━━━━━━━━━
■ migrationファイルとは何か
━━━━━━━━━━━━━━━━━━━

DBへ何をするかを書く設計書。

例：

・teamsテーブル作成
・playersにteam_id追加

など。

━━━━━━━━━━━━━━━━━━━
■ 実際に行ったこと
━━━━━━━━━━━━━━━━━━━

【① migrationファイル作成】

実行：

php artisan make:migration create_teams_table

意味：

Laravelへ
「teamsテーブル用migrationファイル作成して」
と命令。

結果：

database/migrations/
2026_05_08_055625_create_teams_table.php

作成成功。

created successfully
↓
正常作成完了

━━━━━━━━━━━━━━━━━━━
■ create_teams_table.php 初期状態
━━━━━━━━━━━━━━━━━━━

Laravel自動生成：

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('teams', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('teams');
    }
};

━━━━━━━━━━━━━━━━━━━
■ 追加したコード
━━━━━━━━━━━━━━━━━━━

$table->string('name');

最終コード：

Schema::create('teams', function (Blueprint $table) {
    $table->id();

    $table->string('name');

    $table->timestamps();
});

━━━━━━━━━━━━━━━━━━━
■ timestamps() 理解
━━━━━━━━━━━━━━━━━━━

$table->timestamps();

は内部で：

created_at
updated_at

を自動作成する。

つまり teams テーブルには：

id
name
created_at
updated_at

が作られる。

━━━━━━━━━━━━━━━━━━━
■ migration実行
━━━━━━━━━━━━━━━━━━━

実行：

php artisan migrate

意味：

Laravelへ
「migrationを実行して」
と命令。

Laravel内部：

migrationファイル読む
↓
SQLへ変換
↓
MySQLへ送信
↓
teamsテーブル作成

結果：

INFO Running migrations.

2026_05_08_055625_create_teams_table ... DONE

DONE
↓
DB反映成功

━━━━━━━━━━━━━━━━━━━
■ Day1終了時点
━━━━━━━━━━━━━━━━━━━

MySQL内：

teams
 ├ id
 ├ name
 ├ created_at
 └ updated_at

完成。

━━━━━━━━━━━━━━━━━━━
■ 今日理解した本質
━━━━━━━━━━━━━━━━━━━

Docker
↓
コンテナへ入る

Linux
↓
Laravelを操作

artisan
↓
Laravelへ命令

migration
↓
PHPコードでDB構造管理

migrate
↓
MySQLへ実際反映

という流れを理解した。