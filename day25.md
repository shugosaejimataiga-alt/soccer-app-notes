
■ Day9：フィルタ機能 実装まとめ（完全版）


■ ① 今日やったこと


● フィルタ機能実装（position）

・GET /players にフィルタを統合
・positionパラメータによる完全一致検索を実装
・whereによる条件追加の理解



■ ② 実装内容（本質）

追加したコード：

if ($request->has('position')) {
    $query->where('position', $request->position);
}

意味：

・URLのpositionを取得
・一致するデータのみ抽出



■ ③ index()の完成形（現時点）

public function index(Request $request)
{
    $query = Player::query();

    $query->where('is_active', true);

    if ($request->has('name')) {
        $query->where('name', 'LIKE', '%' . $request->name . '%');
    }

    if ($request->has('position')) {
        $query->where('position', $request->position);
    }

    return $query->get();
}



■ ④ API仕様（現在）


● 一覧取得

GET /players
GET /players?name=田中
GET /players?position=MF
GET /players?name=田中&position=MF



■ ⑤ 動作結果

・MFのみ取得成功
・検索とフィルタの併用成功
・論理削除（is_active=true）のみ取得維持
