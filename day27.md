
■ Day11 完全まとめ（見返し用）


■ ① 今日やったこと

👉 検索・フィルタ・ソートを1つのAPIに完全統合

name（部分一致検索）
position（完全一致フィルタ）
sort（並び替え）
デフォルト並び（position順）

👉 filled() を使って条件判定を統一



■ ② 実装の本質

条件があるものだけ$queryに追加し、最後にまとめて実行



■ ③ 処理の流れ

request（入力）
↓
validate（入力チェック）
↓
query（条件追加）
↓
get（DB実行）
↓
response（返却）



■ ④ 条件判定ルール（重要）

$request->filled('name')
$request->filled('position')
$request->filled('sort')

👉 すべて「値があるか」で統一



■ ⑤ なぜfilledが必要か

?name= や ?position= の空入力を無視するため
has → 空でもtrue（NG）
filled → 値があるときだけtrue（OK）



■ ⑥ ソート仕様

sort=name → 名前順
sort=position → GK→DF→MF→FW順
sortなし → position順（デフォルト）



■ ⑦ API仕様（完成形）

GET /api/players
クエリ
name
position
sort



■ ⑧ 動作

・nameあり → 名前検索
・positionあり → 絞り込み
・sortあり → 並び替え
・すべて同時使用可能



■ ⑨ 到達状態

CRUD ✔
論理削除 ✔
検索 ✔
フィルタ ✔
ソート ✔
複合条件 ✔




■ クエリとは何か

👉 データベースへの命令のこと



■ 今回のクエリの意味

あなたのコードのこれです：

$query = Player::query();

これは
「Playerテーブルからどうデータを取るか」を作る箱