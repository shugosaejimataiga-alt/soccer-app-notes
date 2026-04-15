
■ Day8 完全まとめ（検索機能）


■ ① 今日やったこと

・GET /players に検索機能を追加
・nameパラメータを使った部分一致検索を実装
・ルーティングミス（GET重複）を修正
・Controllerの記述ミス（Player, where, Request）を修正
・Postman / ブラウザで動作確認



■ ② 最終コード（index）

public function index(Request $request)
{
    $query = Player::query();

    $query->where('is_active', true);

    if ($request->has('name')) {
        $query->where('name', 'LIKE', '%' . $request->name . '%');
    }

    return $query->get();
}



■ ③ 重要理解

● 一覧APIの考え方

・GET /players = 一覧取得
・検索は「一覧の条件」であり別APIにしない


● Requestの役割

・Request = ユーザー入力の箱
・$request->name → クエリ値取得
・$request->has('name') → 存在確認


● LIKE検索（核心）

・LIKE = あいまい検索
・% = 何でもOK

%田中% → 「田中を含むすべて」


● クエリビルダ
$query = Player::query();

・条件を後から追加できる
・whereを積み重ねる設計


● ルーティング理解
GET /players → index
POST /players → store
PUT /players/{id} → update
DELETE /players/{id} → destroy



■ ④ 発生した問題と原因

● 404エラー

原因：GETがstoreに上書きされていた


● Eloquentエラー

原因：
・player（小文字）
・Player:: の誤使用
・$query->Player::where の誤り


● Request未定義

原因：indexにRequest未指定



■ ⑤ 現在の到達状態

・論理削除 → 完成
・CRUD → 完成
・バリデーション → 完成
・検索（name） → 完成
・GET統合設計 → 完成



■ ⑥ APIの現在仕様

GET /players
GET /players?name=田中
POST /players
PUT /players/{id}
DELETE /players/{id}



■ ⑦ 核

・データ取得はすべてGET
・条件はクエリで渡す
・ロジックはすべてバックエンド