

# 追加Day4 完全まとめ

（users.profile_image 追加）

━━━━━━━━━━━━━━━━━━━

# 現在のロードマップ位置

━━━━━━━━━━━━━━━━━━━

現在：

追加Day4 完了

完了済み：

・追加Day1
　登録設計・role設計

・追加Day2
　invite_code追加

・追加Day3
　users.role追加

・追加Day4
　users.profile_image追加

次回：

追加Day5
新規登録API① チーム作成

━━━━━━━━━━━━━━━━━━━

# 今日の目的

━━━━━━━━━━━━━━━━━━━

users テーブルに：

profile_image

カラムを追加し、

プロフィール画像の保存先を管理できるようにする。

━━━━━━━━━━━━━━━━━━━

# 今日理解した超重要本質

━━━━━━━━━━━━━━━━━━━

profile_image
≠
画像そのもの

# profile_image

画像の保存場所（住所）

━━━━━━━━━━━━━━━━━━━

# 実務での画像管理

━━━━━━━━━━━━━━━━━━━

画像本体をDBへ保存すると、

・DB容量が大きくなる
・検索が重くなる
・バックアップが大きくなる

ため、

実務では

DB
↓
画像の保存場所

storage
↓
画像本体

で管理する。

━━━━━━━━━━━━━━━━━━━

# イメージ

━━━━━━━━━━━━━━━━━━━

画像本体：

storage/app/public/profile_images/user1.jpg

DB保存：

profile_images/user1.jpg

つまり：

# DB

住所録

# storage

写真倉庫

という役割分担。

━━━━━━━━━━━━━━━━━━━

# nullable を付けた理由

━━━━━━━━━━━━━━━━━━━

全ユーザーが

プロフィール画像を設定するとは限らない。

そのため：

```php
->nullable()
```

を付けた。

意味：

```text
画像未設定
↓
null を許可
```

━━━━━━━━━━━━━━━━━━━

# after('role') の意味

━━━━━━━━━━━━━━━━━━━

```php
->after('role')
```

は：

role の次に

profile_image

を配置する。

目的：

見やすさ整理。

━━━━━━━━━━━━━━━━━━━

# 今日作成した migration

━━━━━━━━━━━━━━━━━━━

作成コマンド：

```bash
php artisan make:migration add_profile_image_to_users_table --table=users
```

作成ファイル：

```text
database/migrations/2026_06_04_033518_add_profile_image_to_users_table.php
```

━━━━━━━━━━━━━━━━━━━

# 現在の migration コード

━━━━━━━━━━━━━━━━━━━

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {

            $table->string('profile_image')
                  ->nullable()
                  ->after('role');

        });
    }

    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {

            $table->dropColumn('profile_image');

        });
    }
};
```

━━━━━━━━━━━━━━━━━━━

# migration の意味

━━━━━━━━━━━━━━━━━━━

今回も重要なのは：

migration
≠
画像アップロード機能

migration は：

保存場所を作っただけ。

つまり：

```php
$table->string('profile_image');
```

は、

画像を保存できるようにした

のではなく、

画像の保存先を記録する箱

を作っただけ。

━━━━━━━━━━━━━━━━━━━

# migrate 実行

━━━━━━━━━━━━━━━━━━━

実行：

```bash
php artisan migrate
```

結果：

```text
2026_06_04_033518_add_profile_image_to_users_table
DONE
```

DBへ反映成功。

━━━━━━━━━━━━━━━━━━━

# 現在の users テーブル

━━━━━━━━━━━━━━━━━━━

```text
id
name
email
email_verified_at
password
remember_token
team_id
role
profile_image
created_at
updated_at
```

━━━━━━━━━━━━━━━━━━━

# Tinker確認

━━━━━━━━━━━━━━━━━━━

起動：

```bash
php artisan tinker
```

確認：

```php
\App\Models\User::first();
```

結果：

```php
App\Models\User {
    id: 1,
    name: "Test User",
    email: "test@example.com",
    team_id: 1,
    role: "owner",
    profile_image: null,
}
```

━━━━━━━━━━━━━━━━━━━

# profile_image: null の意味

━━━━━━━━━━━━━━━━━━━

これは：

```text
profile_image カラムが存在する
↓
まだ画像未設定
↓
null が入っている
```

という意味。

つまり、

migration成功
DB反映成功
確認成功

である。

━━━━━━━━━━━━━━━━━━━

# 今日理解した重要ポイント

━━━━━━━━━━━━━━━━━━━

# migration

保存場所作成

# Controller

値を保存

# storage

画像本体保存

# DB

画像の住所保存

# profile_image

画像そのものではなく
画像の保存先

━━━━━━━━━━━━━━━━━━━

# 現在のフォルダ状態

━━━━━━━━━━━━━━━━━━━

database/
└── migrations/
├── add_invite_code_to_teams_table.php
├── add_role_to_users_table.php
└── add_profile_image_to_users_table.php

app/
└── Models/
├── User.php
├── Team.php
└── Player.php

routes/
└── api.php

━━━━━━━━━━━━━━━━━━━

# 現在のDB状態

━━━━━━━━━━━━━━━━━━━

teams テーブル：

・invite_codeあり
・unique制約あり

users テーブル：

・team_idあり
・roleあり
・profile_imageあり
・role default=member

users.id=1：

```text
team_id = 1
role = owner
profile_image = null
```

━━━━━━━━━━━━━━━━━━━

# 追加Day4 到達状態

━━━━━━━━━━━━━━━━━━━

✓ profile_image カラム追加完了

✓ nullable 設定完了

✓ migrate 完了

✓ Tinker確認完了

✓ profile_image: null 確認完了

✓ storage と DB の役割理解完了

到達状態：

プロフィール画像を保存するためのDB設計が完成した。
