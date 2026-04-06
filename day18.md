■ Day2：DB接続設定（MySQL切り替え）


■ ① 今日やったこと（事実）

・LaravelのDB接続をSQLite → MySQLに変更
・.envを修正（DB_CONNECTION / HOST / USER / PASSWORD）
・DockerのMySQLコンテナ（db）に接続する設定に変更
・config:clearで設定キャッシュを削除
・Laravelに新しいDB設定を反映



■ ② 変更内容（コード）
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=laravel


■ ③ 構造の理解（最重要）

● コンテナ構成
app（Laravel）
web（nginx）
db（MySQL）

👉 それぞれ別環境（別PCみたいなもの）


● SQLiteの正体
laravel/database/database.sqlite

👉 ただのファイル
👉 appコンテナ内にある


● MySQLの正体
dbコンテナ

👉 別の場所にあるDBサーバー



■ ④ 本質の理解

● SQLite
同じ場所にあるファイルを読む

👉 接続先指定ほぼ不要


● MySQL
別の場所にあるDBにアクセスする

👉 接続先が必要



■ ⑤ DB_HOST=db の意味
db = docker-composeのサービス名

👉 「dbコンテナに接続する」という意味



■ ⑥ localhostが使えない理由
appコンテナのlocalhost = app自身

👉 MySQLはいない



■ ⑦ なぜ最初SQLiteだったのか

Laravel初期設定がSQLite

理由👇
・設定不要
・すぐ動く

👉 MySQLは「作っただけで未使用」だった



■ ⑧ キャッシュの本質

● Laravelの動き

.envを読む
↓
設定を保存（キャッシュ）
↓
それを使い続ける


● 問題
.env変更しても反映されない


● 解決
php artisan config:clear

👉 キャッシュ削除
👉 新しい設定を読み直す



■ ⑨ エラーの理解（重要）

● エラー
Could not open input file: artisan

● 原因
/var/www/artisan を探した

● 実際
/var/www/laravel/artisan

👉 パスがズレていた

● 解決
php laravel/artisan



■ ⑩ 核（最重要）

👉 SQLite = ファイル
👉 MySQL = 別のサーバー

👉 接続先を変更したのがDay2