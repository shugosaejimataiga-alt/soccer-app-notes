■ Day1まとめ（完全版）


■ ① 何を作ったか
Docker上にLaravel環境を構築

構成：

nginx（Webサーバー）
↓
php（アプリ実行）
↓
Laravel（フレームワーク）
↓
DB（MySQL / SQLite）



■ ② やった流れ

① docker-composeでコンテナ作成
② nginx設定（root設定）
③ Laravelインストール（composer）
④ コンテナ再構築（Dockerfile）
⑤ パス修正
⑥ 権限修正
⑦ 起動確認（ブラウザ表示）



■ ③ 詰まった原因（超重要）

■ 1. 環境不足
composer / git / zip が無かった
↓
Laravelインストール失敗

👉 Dockerは最小構成なので自分で追加する必要がある


■ 2. パスずれ
nginx → /var/www/public を見ていた
実際 → /var/www/laravel/public

👉 見る場所が違ってエラー


■ 3. 権限問題
DBに書き込めない
↓
readonlyエラー

👉 chmodで解決



■ ④ 本質理解（最重要）
Docker = 空の環境
パス = ファイルの場所
権限 = ファイルを操作できるか



■ ⑤ インフラの核心3つ
① 環境 → そもそも動くか
② パス → 見つけられるか
③ 権限 → 触れるか



■ ⑥ 最終状態
http://localhost:8000
↓
Laravel Welcome画面表示

👉 完全成功