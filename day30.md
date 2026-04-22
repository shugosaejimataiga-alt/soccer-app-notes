
Day14 完全まとめ


■ ① 今日やったこと

・API最終確認（CRUD + 検索 + フィルタ + ソート）
・Postmanで実際に通信確認
・500エラーの原因特定（teamsテーブル未作成）
・バリデーション設計の理解（existsの意味）
・フロントのローカル状態を完全削除
・フロントを「UIのみ」にリセット



■ ② バックエンド状態（完全完成）

■ CRUD
GET /players ✔
POST /players ✔
PUT /players/{id} ✔
DELETE /players/{id} ✔（論理削除）

■ ロジック
論理削除（is_active） ✔
検索（name LIKE） ✔
フィルタ（position） ✔
ソート（name / position） ✔
複合条件 ✔

■ レスポンス
{
  "data": {
    "id": 2,
    "teamId": 1,
    "name": "佐藤",
    "position": "FW",
    "isActive": true
  }
}

■ 例外処理
findOrFail() ✔
存在しないID → 404 ✔

■ 注意点（重要）
'team_id' => ['required', 'exists:teams,id']

👉 teamsテーブル未作成だとエラー
👉 現在は一時的に外して動作確認



■ ③ フロントエンド状態（重要）

■ 削除したもの

・useStateによるplayers管理
・addPlayer / updatePlayer / deletePlayer
・props経由のデータ受け渡し
・ローカルでの検索・フィルタ・ソート


■ 修正したもの

App.tsx → props削除
Players.tsx → ロジック削除
PlayerForm → props削除
PlayerList → 完全空
PlayerDetail → ダミー画面
PlayerEdit → ダミー画面
PlayerCreate → ダミー画面
■ 現在の状態

👉 フロントはUIのみ

<PlayerList />
<PlayerForm />

👉 データは一切持たない



■ ④ 設計状態（最重要）

👉 完全バックエンド主導

データ管理 → バックエンド
ロジック → バックエンド
フロント → 表示のみ



■ ⑤ 責務分離
Controller → 処理
Model → DB
Resource → 出力
React → UI



■ ⑥ 現在地

👉 Players機能

API → 完成
フロント → リセット完了



■ ⑦ 次（Day14続き）

👉 API接続

GET → 一覧取得
POST → 作成
PUT → 更新
DELETE → 削除



■ 核

👉 データの真実はバックエンド
👉 フロントは表示のみ
👉 すべてAPI経由