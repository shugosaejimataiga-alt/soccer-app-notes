
■ Day5 完全まとめ（見返し用）


■ ■ 目的
👉 PlayerのCRUD APIを作る



■ ■ 全体構造（本質）

Route → Controller → Model → DB

👉 URLをきっかけに処理が動く



■ ■ APIの本質

👉 APIは

「呼ばれた時だけ実行される処理」

👉 自動では動かない
👉 フロントが呼ぶ



■ ■ CRUDとHTTP
GET    → 取得
POST   → 作成
PUT    → 更新
DELETE → 削除

👉 methodで処理が決まる



■ ■ 実装内容

■ ① Controller作成
php artisan make:controller PlayerController


■ ② 一覧取得（Read）

public function index()
{
    return Player::all();
}


■ ③ 新規作成（Create）

public function store(Request $request)
{
    return Player::create($request->all());
}


■ ④ 更新（Update）

public function update(Request $request, $id)
{
    $player = Player::findOrFail($id);
    $player->update($request->all());

    return $player;
}


■ ⑤ 削除（Delete）

public function destroy($id)
{
    $player = Player::findOrFail($id);
    $player->delete();

    return response()->json(['message' => 'deleted']);
}



■ ■ ルーティング
Route::get('/players', [PlayerController::class, 'index']);
Route::post('/players', [PlayerController::class, 'store']);
Route::put('/players/{id}', [PlayerController::class, 'update']);
Route::delete('/players/{id}', [PlayerController::class, 'destroy']);



■ ■ 重要理解①（ルート）

👉 Laravelは

「URL + HTTPメソッド」で処理を決定

👉 同時に複数動くことはない



■ ■ 重要理解②（playersの意味）

👉 /players は

選手データというグループ名

👉 機能は method で分かれる



■ ■ 重要理解③（Reactとの違い）
フロント
URL = 画面
バックエンド
URL = データ操作

👉 完全に別物



■ ■ 重要理解④（Request）
update → 新しいデータ必要 → Requestあり
delete → idだけでOK → Requestなし



■ ■ 重要理解⑤（APIの動き）
フロントが呼ぶ → API実行 → DB操作 → JSON返却



■ ■ 到達状態
GET    /players        ✔
POST   /players        ✔
PUT    /players/{id}   ✔
DELETE /players/{id}   ✔

👉 CRUD完成



■ ■ 本質まとめ（最重要）

👉 APIとは

「データを操作するための入口」

👉 画面とは無関係
👉 呼ばれた時だけ動く