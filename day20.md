■ Day4 完全まとめ（見返し用）


■ ① 今日やったこと
● Model作成
php artisan make:model Player

👉 DB操作用のクラスを作成

● fillable設定
protected $fillable = [
    'team_id',
    'name',
    'position',
    'photo_url',
    'is_active'
];

👉 一括登録（create）を可能にするため
👉 登録してよいカラムを明示



■ ② Modelの役割（最重要）

■ 結論
👉 DBとプログラムをつなぐもの



■ ③ データの流れ

Controller
  ↓
Model（Player）
  ↓
DB（playersテーブル）
  ↓
Model
  ↓
Playerオブジェクト



■ ④ 核の理解

● Model
👉 DBにアクセスするための窓口

● Playerオブジェクト
👉 DBから取得された「1行のデータ」



■ ⑤ 用語

■ DB側
id / name / position → カラム

■ プログラム側
$player->name → プロパティ



■ ⑥ なぜModelが必要か

👉 SQLを書かずにDB操作するため

Player::find(1);
Player::create([...]);
Player::where(...);



■ ⑦ 構造（最重要）
テーブルごとにModelが1つ
今回 → playersテーブル → Playerモデル



■ ⑧ 現在の状態
DB：完成 ✔
Model：完成 ✔
API：まだ ✖



■ ⑨ 今日の核心

👉 DBのデータをオブジェクトとして扱える状態を作った