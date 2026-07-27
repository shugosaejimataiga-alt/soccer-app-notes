# 追加Day13：Team情報取得API 完全まとめ

## 1. 今日の目的

今日の目的は、ログイン中ユーザーが所属しているTeam情報と、そのTeamに所属しているUser一覧を取得するAPIを作ることでした。

完成したAPIは次です。

```text
GET /api/team
```

このAPIをtoken付きで呼ぶと、次の2つがJSONで返ります。

```text
team
→ ログイン中ユーザーが所属するチーム情報

users
→ そのチームに所属するユーザー一覧
```

---

# 2. 今日の作業全体の流れ

今日の作業は、大きく3段階でした。

```text
① データベース・Modelを確認する

② Controllerに処理を書く

③ routes/api.phpにAPIの入口を作る
```

その後、Postmanで実際にAPIが動くか確認しました。

```text
データベース・Model確認
↓
TeamController作成
↓
処理作成
↓
APIルート作成
↓
Postmanでログイン
↓
token付きでGET /api/team
↓
レスポンス確認
↓
重複していたusersを修正
```

---

# 3. データベース構造の確認

まず、現在のデータベースにどのカラムがあるか確認しました。

## usersテーブル

Tinkerで次を実行しました。

```php
Schema::getColumnListing('users');
```

確認できたカラムは次です。

```text
id
name
email
email_verified_at
password
remember_token
created_at
updated_at
team_id
role
profile_image
```

特に今回重要だったのは、

```text
team_id
```

です。

`users.team_id`には、そのUserが所属しているTeamのIDが入っています。

たとえば、

```text
users.id = 5
users.name = 田中
users.team_id = 2
```

なら、田中さんは、

```text
teams.id = 2
```

のTeamに所属しています。

---

## teamsテーブル

次に、teamsテーブルを確認しました。

```php
Schema::getColumnListing('teams');
```

確認できたカラムは次です。

```text
id
name
created_at
updated_at
invite_code
```

今回のデータベース上の関係は次です。

```text
teams.id
↑
users.team_id
```

1つのTeamに、複数のUserが所属できます。

つまり、

```text
Team：1
User：多
```

の1対多の関係です。

---

# 4. SchemaとModelの違い

## Schema

```php
Schema::getColumnListing('users');
```

の`Schema`は、データベースの構造を扱います。

```text
どんなテーブルがあるか
どんなカラムがあるか
カラムを追加・変更・削除する
```

などを扱います。

今回の、

```php
Schema::getColumnListing('users');
```

は、

```text
usersテーブルには、どんなカラムがあるか
```

を確認する処理でした。

---

## Model

Modelは、テーブルの中に保存されている実際のデータをLaravelから扱うためのものです。

たとえば、

```php
App\Models\User::all();
```

なら、usersテーブルに保存されているUserデータをすべて取得します。

整理すると、

```text
Schema
→ テーブルの構造を見る・変更する

Model
→ テーブル内の実際のデータを操作する
```

です。

---

# 5. Userモデルの確認

現在の`User.php`には、次のリレーションがありました。

```php
public function team()
{
    return $this->belongsTo(Team::class);
}
```

意味は、

```text
1人のUserは、1つのTeamに所属する
```

です。

データベース上では、

```text
users.team_id
↓
teams.id
```

を使います。

このリレーションがあることで、UserからTeamを次のように取得できます。

```php
$user->team
```

これは、

```text
このUserが所属しているTeam
```

という意味です。

---

# 6. Teamモデルの確認

現在の`Team.php`は次の状態でした。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Team extends Model
{
    protected $fillable = [
        'name',
        'invite_code',
    ];

    public function users()
    {
        return $this->hasMany(User::class);
    }
}
```

重要なのは次のリレーションです。

```php
public function users()
{
    return $this->hasMany(User::class);
}
```

意味は、

```text
1つのTeamは、複数のUserを持つ
```

です。

このリレーションがあることで、Teamに所属するUserを取得できます。

```php
$team->users
```

または、

```php
$team->users()->get()
```

を使えます。

---

# 7. belongsToとhasMany

今回、両方向のリレーションを確認しました。

## User側

```php
public function team()
{
    return $this->belongsTo(Team::class);
}
```

```text
UserはTeamに所属する
```

## Team側

```php
public function users()
{
    return $this->hasMany(User::class);
}
```

```text
Teamは複数のUserを持つ
```

全体は次です。

```text
User → Team
belongsTo

Team → Users
hasMany
```

---

# 8. MigrationとModelの役割

データベースを作るときは、MigrationとModelは基本的に別の役割があります。

## Migration

```text
データベースのテーブルやカラムを設計する
```

たとえば、

```text
teamsテーブルを作る
nameカラムを作る
invite_codeカラムを作る
```

などです。

作成コマンド例：

```bash
php artisan make:migration create_teams_table
```

Migrationを書いた後に、

```bash
php artisan migrate
```

を実行すると、実際のデータベースに反映されます。

---

## Model

```text
作られたテーブルのデータをLaravelから操作する
```

作成コマンド例：

```bash
php artisan make:model Team
```

ModelとMigrationを同時に作る場合は、

```bash
php artisan make:model Team -m
```

です。

整理すると、

```text
Migration
→ テーブルを作る・設計する

Model
→ テーブル内のデータを操作する
```

です。

---

# 9. TeamControllerの作成

今回はTeamに関するAPI処理を書くため、新しくTeamControllerを作りました。

実行したコマンドは次です。

```bash
php artisan make:controller TeamController
```

このコマンドによって、Laravelが自動的に次のファイルを作りました。

```text
app/Http/Controllers/TeamController.php
```

`make:controller`と書くことで、LaravelはController用のフォルダにControllerファイルを作ります。

分解すると、

```text
php artisan
→ Laravelのコマンドを実行する

make:controller
→ Controllerを作る

TeamController
→ 作るController名
```

です。

---

# 10. TeamControllerの役割

TeamControllerは、単に「Teamページを開いたときに動くファイル」ではありません。

正確には、

```text
Teamに関するAPI処理を担当するController
```

です。

今回の流れは次です。

```text
ReactでTeamページを開く
↓
ReactがGET /api/teamを送る
↓
routes/api.phpが受け取る
↓
TeamControllerのindex()が実行される
↓
Team情報とUsers一覧を取得する
↓
JSONでReactへ返す
```

ページを開いたこと自体で直接Controllerが動くのではなく、ReactがAPIリクエストを送ったときにControllerが動きます。

---

# 11. Request $requestとは何か

TeamControllerのメソッドは次のように作りました。

```php
public function index(Request $request)
```

`Request $request`は、Reactから送られてきたHTTPリクエストをもとに、Laravelが作成したRequestオブジェクトです。

重要なのは、

```text
ReactがRequestオブジェクトを送るわけではない
```

ということです。

Reactが送るのは、次のようなHTTPリクエストです。

```text
URL
GET・POSTなどのHTTPメソッド
Authorizationヘッダー
送信データ
```

たとえば、

```text
GET /api/team
Authorization: Bearer token
```

です。

Laravelは、そのHTTPリクエストを受け取り、扱いやすいRequestオブジェクトにまとめてControllerへ渡します。

```text
React
↓
HTTPリクエストを送る
↓
Laravelが受け取る
↓
Requestオブジェクトを作る
↓
ControllerのRequest $requestに渡す
```

---

# 12. auth:sanctumと$request->user()の違い

ここは今日、特に重要だった部分です。

## auth:sanctum

`routes/api.php`では、次のmiddlewareを使っています。

```php
Route::middleware('auth:sanctum')
```

この`auth:sanctum`が、Controllerに到達する前にtokenを確認します。

処理の流れは次です。

```text
Reactがtokenを送る
↓
auth:sanctumがtokenを取得する
↓
personal_access_tokensテーブルなどと照合する
↓
tokenの持ち主であるUserを特定する
↓
認証に成功した場合だけControllerへ進む
```

つまり、tokenの照合と認証はmiddlewareで済んでいます。

---

## $request->user()

Controllerでは、次を書きました。

```php
$user = $request->user();
```

これは、tokenをもう一度認証しているわけではありません。

意味は、

```text
auth:sanctumが認証したUserを取得する
```

です。

整理すると、

```text
auth:sanctum
→ tokenを確認して、ログイン中Userを特定する

$request->user()
→ 認証済みUserをControllerで取得する
```

です。

認証が2回行われているのではなく、

```text
認証
↓
認証結果のUserを取り出す
```

という2段階です。

---

# 13. tokenは$request->user()に入っているのか

`$request->user()`にtokenそのものが入っているわけではありません。

`$request->user()`で取得できるのは、tokenを使って特定されたUserモデルです。

```text
token
↓
auth:sanctumが照合
↓
Userを特定
↓
$request->user()
↓
Userモデルを取得
```

Userモデルには、たとえば次の情報があります。

```text
id
name
email
team_id
role
profile_image
```

token文字列自体は、HTTPリクエストのAuthorizationヘッダーに入っています。

```text
Authorization: Bearer token
```

token文字列が必要なら、

```php
$request->bearerToken();
```

で取得できます。

ただし今回はtoken文字列が欲しいのではなく、

```text
そのtokenの持ち主は誰か
```

を知りたいので、

```php
$request->user();
```

を使いました。

---

# 14. localStorageにtokenを保存する意味

tokenはログイン時に一度LaravelからReactへ返されます。

```text
POST /api/login
↓
認証成功
↓
Laravelがtokenを発行
↓
Reactへtokenを返す
```

Reactは、そのtokenをlocalStorageに保存します。

```tsx
localStorage.setItem("token", data.token)
```

その後は、APIを呼ぶたびにLaravelからtokenを受け取り直すのではありません。

React側に保存してある同じtokenを取り出して、毎回APIへ送ります。

```tsx
const token = localStorage.getItem("token")
```

流れは次です。

```text
ログイン時
→ tokenを受け取ってlocalStorageに保存

ログイン後
→ localStorageからtokenを取得

APIを呼ぶたび
→ Authorizationヘッダーにtokenを付けて送る

APIのレスポンス
→ teamやusersなど必要なデータだけ返る
```

tokenは毎回送信しますが、毎回Laravelから返してもらうわけではありません。

また、localStorageは画面更新やブラウザのタブを閉じても保存が残るため、次に画面を開いたときも、

```tsx
localStorage.getItem("token")
```

で取得できます。

ログアウト時には、

```tsx
localStorage.removeItem("token")
```

で削除します。

---

# 15. TeamControllerの処理の流れ

最初に作った処理は次です。

```php
public function index(Request $request)
{
    $user = $request->user();

    $team = $user->team;

    $users = $team->users;
}
```

処理の意味は次です。

## ① ログイン中Userを取得

```php
$user = $request->user();
```

```text
auth:sanctumで認証されたUserを取得する
```

## ② Userの所属Teamを取得

```php
$team = $user->team;
```

Userモデルに定義した、

```php
public function team()
{
    return $this->belongsTo(Team::class);
}
```

を使います。

```text
ログイン中User
↓
users.team_id
↓
teams.id
↓
所属Teamを取得
```

## ③ Teamに所属するUsersを取得

最終的には次の書き方にしました。

```php
$users = $team->users()->get();
```

Teamモデルに定義した、

```php
public function users()
{
    return $this->hasMany(User::class);
}
```

を使います。

```text
Teamのid
↓
users.team_id
↓
同じteam_idを持つUserをすべて取得
```

---

# 16. hasManyの意味

最初は、

```php
return $this->hasMany(User::class);
```

を単に、

```text
1対多の関係であることを書く決まり
```

だと理解していました。

しかし、実際にはそれだけではありません。

`hasMany`は、Laravelに次のことを教えています。

```text
このTeamに関係する複数のUserを、
どのようにデータベースから探すか
```

今回なら、

```text
このTeamのid
と
users.team_id
が同じUserを探す
```

という検索条件を作ります。

たとえば、

```text
$team->id = 3
```

なら、Laravelはイメージとして、

```text
usersテーブルから
team_id = 3
のUserをすべて探す
```

という処理をします。

つまり、

```php
$this->hasMany(User::class)
```

は、

```text
このTeamが持っている複数のUserを探すための関係
```

です。

---

# 17. $team->usersと$team->users()の違い

ここが今日一番難しかった部分です。

## ()なし

```php
$team->users
```

これは、usersリレーションの結果であるUser一覧を読みます。

```text
Teamに所属するUser一覧そのものを取得する
```

Laravelでは、リレーションをこの形で取得すると、取得したUser一覧が`$team`の読み込み済みリレーションとして保持されます。

そのため、

```php
$users = $team->users;
```

を実行した後の`$team`は、JSONにするとイメージとして次の状態になります。

```text
$team
├─ id
├─ name
├─ invite_code
├─ created_at
├─ updated_at
└─ users
   ├─ User1
   ├─ User2
   └─ User3
```

そして、同じユーザー一覧が`$users`にも入ります。

```text
$users
├─ User1
├─ User2
└─ User3
```

---

## ()あり

```php
$team->users()
```

これは、Teamモデルに書いた`users()`メソッドを直接呼びます。

```php
public function users()
{
    return $this->hasMany(User::class);
}
```

`users()`が返すものはUser一覧そのものではなく、

```text
このTeamに所属するUserを探すためのhasManyリレーション処理
```

です。

その処理に対して、

```php
->get()
```

を付けます。

```php
$team->users()->get();
```

これで、実際にデータベースを検索してUser一覧を取得します。

整理すると、

```text
$team->users()
→ このTeamに所属するUserを探す処理

get()
→ その検索を実行してUser一覧を取得
```

です。

---

# 18. なぜ後ろのコードを変えると$teamの出力が変わったのか

前のコードは次でした。

```php
$team = $user->team;

$users = $team->users;
```

最初の、

```php
$team = $user->team;
```

の時点では、`$team`はTeam情報だけでした。

```text
$team
├─ id
├─ name
├─ invite_code
├─ created_at
└─ updated_at
```

しかし、後ろで、

```php
$users = $team->users;
```

と書きました。

この`$team->users`によって、LaravelはUser一覧を取得すると同時に、そのリレーション結果を同じ`$team`にも読み込みました。

そのため、後ろの行を実行した後は、

```text
$team
├─ id
├─ name
├─ invite_code
├─ created_at
├─ updated_at
└─ users
```

となりました。

つまり、

```text
後ろのコードが$usersを取得しただけではなく、
同じ$teamオブジェクトの状態にも影響していた
```

ためです。

その状態で、

```php
return response()->json([
    'team' => $team,
    'users' => $users,
]);
```

と返したので、

```json
{
  "team": {
    "id": 1,
    "name": "チーム名",
    "users": [
      "ユーザー一覧"
    ]
  },
  "users": [
    "ユーザー一覧"
  ]
}
```

と、同じUser一覧が2か所に出ました。

---

# 19. なぜ$team->users()->get()に変えると直ったのか

現在のコードは次です。

```php
$users = $team->users()->get();
```

この書き方では、

```text
users()が返すhasManyの検索処理を使う
↓
get()でUser一覧を取得する
↓
取得結果を$usersに入れる
```

という動きです。

この取得結果は、`$team`の`users`リレーションとしては読み込まれません。

そのため、

```text
$team
├─ id
├─ name
├─ invite_code
├─ created_at
└─ updated_at
```

のままです。

別に、

```text
$users
├─ User1
├─ User2
└─ User3
```

が作られます。

結果として、レスポンスはきれいに分かれます。

```json
{
  "team": {
    "id": 1,
    "name": "チーム名",
    "invite_code": "ABC123"
  },
  "users": [
    {
      "id": 1,
      "name": "ユーザー名"
    }
  ]
}
```

---

# 20. $user->teamでも同じことが起こるのか

はい、同じ仕組みです。

```php
$team = $user->team;
```

とすると、`$user`側にTeamリレーションが読み込まれます。

イメージは次です。

```text
$user
├─ id
├─ name
├─ email
├─ team_id
└─ team
   └─ Team情報
```

ただし、今回レスポンスで返したのは、

```php
'team' => $team,
'users' => $users,
```

だけです。

`$user`は返していません。

そのため、`$user`の中に読み込まれた`team`はJSONには出ません。

もし次のように`$user`も返した場合、

```php
return response()->json([
    'user' => $user,
    'team' => $team,
    'users' => $users,
]);
```

`$user`の中にもTeam情報が出る可能性があります。

```json
{
  "user": {
    "id": 1,
    "name": "田中",
    "team": {
      "id": 2,
      "name": "奈良FC"
    }
  },
  "team": {
    "id": 2,
    "name": "奈良FC"
  },
  "users": [
    {
      "id": 1,
      "name": "田中"
    }
  ]
}
```

つまり、

```text
$user->team
→ $user側にteamリレーションが読み込まれる

$team->users
→ $team側にusersリレーションが読み込まれる
```

です。

---

# 21. PHPの()あり・なし

今日、PHPの読み方として重要な点も学びました。

## ()あり

```php
users()
get()
response()
json()
```

`()`が付いている場合は、基本的に関数やメソッドを実行しています。

たとえば、

```php
$team->users()
```

は、Teamモデルの`users()`メソッドを呼びます。

```php
->get()
```

は、検索を実行して結果を取得します。

```php
response()
```

は、レスポンスを作る処理を呼びます。

```php
->json()
```

は、JSONレスポンスを作ります。

今回の覚え方は、

```text
()あり
→ 処理・メソッド・関数を呼ぶ
```

です。

---

## ()なし

```php
$team->users
```

のように`()`がない場合は、値や結果を読む形です。

ただし、LaravelのEloquentでは特殊な仕組みがあり、`users`という普通のプロパティがなくても、Laravelが`users()`というリレーションメソッドを探し、その結果を取得してくれます。

今回の覚え方は、

```text
()なし
→ リレーションの取得結果を読む

()あり
→ リレーションの処理を呼ぶ
```

です。

---

# 22. フロントを信用しないとは何か

途中で、

```text
フロントを信用しないなら、$userも返すべきではないか
```

という疑問がありました。

ここで重要なのは、

```text
バックエンドで安全に処理すること
```

と、

```text
フロントへどのデータを返すか
```

は別の話だということです。

---

## フロントを信用しないためにバックエンドですること

### ① 本人をバックエンドで特定する

フロントから、

```text
私はuser_id = 3です
```

と送られても、その値を信用しません。

tokenを使って、

```php
$user = $request->user();
```

でバックエンドが本人を特定します。

---

### ② 所属Teamをバックエンドで判断する

フロントから、

```text
team_id = 5です
```

と送らせて、その値をそのまま信じません。

認証済みUserを起点に、

```php
$team = $user->team;
```

で所属Teamを取得します。

---

### ③ 権限をバックエンドで確認する

たとえばownerだけがTeam名を変更できる場合、フロントでボタンを隠すだけでは不十分です。

フロント画面は改変できるため、バックエンドでも、

```php
if ($user->role !== 'owner') {
    return response()->json([
        'message' => '権限がありません',
    ], 403);
}
```

などの確認が必要です。

---

### ④ 入力値をバックエンドで検証する

フロント側で入力チェックをしていても、APIへ直接不正なデータを送ることができます。

そのため、POSTやPUTではバックエンドでもバリデーションします。

```php
$request->validate([
    'name' => ['required', 'string', 'max:255'],
]);
```

---

## 今回のGET /api/team

今回、フロントから`user_id`や`team_id`を受け取っていません。

バックエンドが、

```text
tokenからUserを特定
↓
そのUserからTeamを取得
↓
そのTeamからUsersを取得
```

と判断しています。

これが、

```text
フロントを信用せず、バックエンドで判断する
```

ということです。

---

# 23. なぜ$userをレスポンスで返さないのか

バックエンドでは、

```php
$user = $request->user();
```

が必要です。

これは所属Teamを探すための起点だからです。

```text
認証済みUser
↓
所属Team
↓
Team所属Users
```

しかし、処理で使ったデータをすべてフロントへ返す必要はありません。

今回のReact側では、すでに、

```text
GET /api/user
↓
AuthContextのuserに保存
```

という仕組みがあります。

つまりReactは、ログイン中ユーザー情報をすでに持っています。

そのため、Team APIでは追加で必要な、

```text
team
users
```

だけを返します。

```php
return response()->json([
    'team' => $team,
    'users' => $users,
]);
```

整理すると、

```text
$userを取得する
→ バックエンド内部の処理に必要

$userを返す
→ Reactが必要なら返す

今回はAuthContextにすでにある
→ Team APIでは返さない
```

です。

---

# 24. $usersだけではログイン中本人は分からない

`$users`にはチーム全員が入っています。

たとえば、

```text
田中
佐藤
山田
```

と入っていても、これだけでは誰がログイン中かは分かりません。

`role`でも判断できません。

```text
owner
member
```

は権限を表すものであり、ログイン中本人を表すものではありません。

ログイン中本人を画面側で判断したい場合は、AuthContextの`user.id`と、users一覧の各Userの`id`を比較できます。

```tsx
member.id === user.id
```

ただし、この比較は表示上の判定です。

権限やセキュリティの判断は、必ずバックエンドでも行う必要があります。

---

# 25. 今回バリデーションが不要だった理由

今回のAPIは、

```text
GET /api/team
```

です。

フロントから、検証する入力値を受け取っていません。

```text
name
email
team_id
```

などを送信していないため、通常の入力バリデーションは不要です。

認証は、

```php
auth:sanctum
```

が担当しています。

ただし、将来的には所属Teamがない場合などのエラーハンドリングを追加できます。

```php
if (!$team) {
    return response()->json([
        'message' => '所属チームがありません',
    ], 404);
}
```

これは入力値のバリデーションではなく、取得結果に対するエラー処理です。

整理すると、

```text
フロントから入力値を受け取る
→ バリデーション

ログイン済みか確認する
→ auth:sanctum

Teamが存在するか確認する
→ エラーハンドリング
```

です。

---

# 26. routes/api.phpの作成

TeamControllerをAPIから呼べるようにするため、`routes/api.php`に次を追加しました。

## TeamControllerの読み込み

```php
use App\Http\Controllers\TeamController;
```

## Team APIのルート

```php
Route::get('/team', [TeamController::class, 'index']);
```

認証が必要なので、`auth:sanctum`グループ内に置きました。

最終的な関連部分は次です。

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\PlayerController;
use App\Http\Controllers\AuthController;
use App\Http\Controllers\RegisterController;
use App\Http\Controllers\TeamController;

Route::middleware('auth:sanctum')->group(function () {

    Route::get('/players', [PlayerController::class, 'index']);
    Route::post('/players', [PlayerController::class, 'store']);
    Route::put('/players/{id}', [PlayerController::class, 'update']);
    Route::delete('/players/{id}', [PlayerController::class, 'destroy']);
    Route::get('/players/{id}', [PlayerController::class, 'show']);

    Route::get('/team', [TeamController::class, 'index']);

});
```

このルートの意味は次です。

```php
Route::get('/team', [TeamController::class, 'index']);
```

```text
Route::get
→ GETリクエストを受け付ける

'/team'
→ APIのパス

TeamController::class
→ TeamControllerを使う

'index'
→ indexメソッドを実行する
```

`routes/api.php`に書いているため、実際のURLには自動で`/api`が付きます。

```text
GET /api/team
```

です。

---

# 27. Postmanでの動作確認

## Dockerの起動確認

次を実行しました。

```bash
docker compose ps
```

結果は次でした。

```text
laravel_app   Up 5 days
laravel_db    Up 5 days
laravel_web   Up 5 days
```

すべて起動していました。

特に重要だったのは、webコンテナのポートです。

```text
0.0.0.0:8000->80/tcp
```

意味は、

```text
パソコン側の8000番ポート
↓
Docker内のNginxの80番ポート
```

です。

そのため、PostmanのURLは、

```text
http://localhost:8000
```

を使います。

最初に、

```text
http://localhost/api/login
```

へ接続したため、パソコン側の80番ポートへ接続しようとして、

```text
ECONNREFUSED 127.0.0.1:80
```

になりました。

正しいURLは、

```text
http://localhost:8000/api/login
```

です。

---

# 28. Postmanでログイン

ログインAPIは、`routes/api.php`の次のコードから確認できます。

```php
Route::post('/login', [AuthController::class, 'login']);
```

そのため、Postmanでは次の設定にしました。

```text
Method：POST
URL：http://localhost:8000/api/login
```

Bodyは、

```text
Body
↓
raw
↓
JSON
```

を選択します。

入力内容は次です。

```json
{
  "email": "ログインするメールアドレス",
  "password": "ログインするパスワード"
}
```

ログインに成功し、tokenを取得できました。

---

# 29. PostmanでGET /api/team

次にPostmanで新しいリクエストを作りました。

```text
Method：GET
URL：http://localhost:8000/api/team
```

Authorizationタブで、

```text
Type：Bearer Token
Token：ログイン時に取得したtoken
```

を設定しました。

最初はPOSTのまま送信したため、

```text
405 Method Not Allowed
```

が返りました。

エラー内容は、

```text
The POST method is not supported for route api/team.
Supported methods: GET, HEAD.
```

でした。

`routes/api.php`では、

```php
Route::get('/team', ...)
```

と書いているため、POSTではなくGETで送る必要があります。

GETに変更して再送した結果、Team情報とUsers一覧が正常に返りました。

---

# 30. ownerでなくてもGET /api/teamは使える

ログインしたUserはownerでしたが、現在の`GET /api/team`はowner限定ではありません。

ルートには、

```php
auth:sanctum
```

しか設定していません。

そのため、

```text
ログイン済み
＋
Teamに所属している
```

なら、ownerでもmemberでもアクセスできます。

owner限定にしたい場合は、別途バックエンドでroleを確認する必要があります。

---

# 31. 最終的なTeamController

最終的な`TeamController.php`は次です。

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TeamController extends Controller
{
    public function index(Request $request)
    {
        $user = $request->user(); // 所属Teamを探すため、認証済みUserを取得する

        $team = $user->team;

        $users = $team->users()->get();

        return response()->json([
            'team' => $team,
            'users' => $users,
        ]);
    }
}
```

---

# 32. 最終的な処理の流れ

```text
① ReactがlocalStorageからtokenを取得する

② GET /api/teamを送る
   Authorization: Bearer token

③ routes/api.phpのauth:sanctumを通る

④ Sanctumがtokenを照合する

⑤ tokenの持ち主であるUserを特定する

⑥ TeamControllerのindex()が実行される

⑦ $request->user()で認証済みUserを取得する

⑧ $user->teamで所属Teamを取得する

⑨ $team->users()->get()で所属User一覧を取得する

⑩ teamとusersをJSONで返す

⑪ ReactがJSONを受け取って画面表示に使う
```

---

# 33. 今日の重要理解

## 重要理解1

```text
Migrationでデータベースの関係を作る
↓
Modelでリレーションを定義する
↓
Controllerでリレーションを使う
```

---

## 重要理解2

```text
auth:sanctum
→ tokenを照合して認証する

$request->user()
→ 認証済みUserを取得する
```

認証は2回ではありません。

---

## 重要理解3

```text
フロントを信用しない
=
本人・所属・権限・入力値をバックエンドで確認する
```

返すデータを多くすることではありません。

---

## 重要理解4

```text
処理で使ったデータ
と
Reactへ返すデータ
は別
```

`$user`は処理に必要でも、レスポンスに必ず含める必要はありません。

---

## 重要理解5

```text
$team->users
→ User一覧を取得する
→ $teamのusersリレーションとしても読み込まれる

$team->users()->get()
→ users()のhasMany検索処理を使う
→ get()でUser一覧を取得する
→ 結果は$usersに入る
→ $teamの中には読み込まれない
```

---

## 重要理解6

```text
()あり
→ メソッド・関数・処理を呼び出す

()なし
→ 値やリレーション結果を読む
```

ただし、`$team->users`でリレーションを自動取得する部分はLaravel独自の仕組みです。

---

## 重要理解7

```text
hasMany
→ 単なる1対多の宣言ではない
→ 関連するデータを探す検索処理も作る
```

今回なら、

```text
teams.id
と
users.team_id
が一致するUserをすべて探す
```

という処理です。

---

# 34. Day13完了状況

```text
✅ usersテーブル確認
✅ teamsテーブル確認
✅ UserモデルのbelongsTo確認
✅ TeamモデルのhasMany確認
✅ TeamController作成
✅ index()作成
✅ 認証済みUser取得
✅ 所属Team取得
✅ Team所属Users取得
✅ JSONレスポンス作成
✅ TeamControllerをroutes/api.phpでuse
✅ GET /api/teamを追加
✅ auth:sanctum内に配置
✅ Docker起動確認
✅ Postmanでログイン
✅ token取得
✅ Bearer Token設定
✅ GET /api/team実行
✅ 405エラーの原因理解
✅ TeamとUsersが返ることを確認
✅ usersの重複原因を理解
✅ $team->usersから$team->users()->get()へ変更
✅ 修正後のレスポンス確認
```

# 追加Day13「Team情報取得API」完了
