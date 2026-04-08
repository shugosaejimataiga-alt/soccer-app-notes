
■ Day3 完全まとめ（見返し用）


■ ① 今日やったこと（流れ）
1. DBを使える状態にした
php artisan migrate

👉 Laravelの初期テーブルをDBに作成

2. playersテーブルの設計を作成
php artisan make:migration create_players_table

👉 playersテーブルの「設計ファイル」を作成

3. カラム（中身）を定義
$table->id();
$table->unsignedBigInteger('team_id');
$table->string('name');
$table->string('position');
$table->string('photo_url')->nullable();
$table->boolean('is_active')->default(true);
$table->timestamps();

👉 Playerの構造をDB用に定義

4. DBに反映
php artisan migrate

👉 playersテーブルを実際に作成



■ ② 詰まった原因（重要）
原因①：接続先ミス
.envとMySQLの設定がズレていた

👉 Access denied

原因②：DB未起動
MySQLが準備中で接続不可

👉 Connection refused

原因③：DBが空
テーブルが無くLaravelが動かない

👉 migrateが必要



■ ③ 核の理解（最重要）

■ DBの役割
👉 データを保存する場所


■ フロントとの違い
React → メモリ（消える）
DB → 永続（消えない）


■ 開発の流れ
① 設計を作る（migration）
↓
② 中身を決める（カラム）
↓
③ DBに作る（migrate）


■ 対応関係
ReactのPlayer型
↓
DBのplayersテーブル


■ ④ 一言で
👉 Playerを保存する本物の場所を作った