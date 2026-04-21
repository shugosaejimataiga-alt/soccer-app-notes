

■ Day12：完全まとめ（レスポンス整形）



■ ① 今日やったこと

👉 APIの返却形式を制御する仕組みを導入

PlayerResource 作成
toArrayで返却データを定義
ControllerからResourceを使って返却



■ ② 実装内容


■ Resource作成

php artisan make:resource PlayerResource


■ レスポンス定義

return [
    'id' => $this->id,
    'teamId' => $this->team_id,
    'name' => $this->name,
    'position' => $this->position,
    'isActive' => (bool) $this->is_active,
];



■ Controller修正

■ 一覧（複数）

return PlayerResource::collection($query->get());

■ 作成・更新（単体）

return new PlayerResource($player);



■ ③ 現在のレスポンス仕様

{
  "data": [
    {
      "id": 1,
      "teamId": 1,
      "name": "田中",
      "position": "MF",
      "isActive": true
    }
  ]
}



■ ④ 重要理解（最重要）


■ DBとAPIは別物

DB → 保存用の構造

API → フロント用の構造



■ Resourceの役割

👉 APIの出口を制御する

カラム名変換（snake → camel）
型変換（1 → true）
不要データ削除



■ collection と new の違い

collection → 複数データ

new Resource → 単体データ



■ ⑤ 設計の本質

👉 責務分離

routes → URL
Controller → 処理
Model → DB
Resource → 出口



■ ⑥ 到達状態

APIレスポンス統一 ✔

フロント依存設計 ✔

DB依存排除 ✔



■ ⑦ 位置づけ

👉 Day11まで：処理を作る
👉 Day12：返し方を完成させる



■ 結論（核）

👉 「データの真実はバックエンド」＋「返し方もバックエンドが支配する」