

■ Day13 完全まとめ（復習用）


■ 今日やったこと

・例外処理の確認
・findOrFail() による存在チェックの理解


■ 実装の本質
$player = Player::findOrFail($id);


■ 何が起きているか

・存在する → データ取得して処理続行
・存在しない → 自動で404エラー


■ なぜ重要か

・不正なIDアクセスを防ぐ
・バグを防ぐ
・APIの安全性を担保


■ 自分がやったことの本質

👉 if文を書かずに例外処理を実現した


■ Laravelの仕組み理解

・find() → 見つからないとnull
・findOrFail() → 見つからないと404


■ 現在のコード状態

・update → findOrFail 使用 ✔
・destroy → findOrFail 使用 ✔


■ 到達状態

・存在チェック ✔
・404対応 ✔
・安全なAPI ✔