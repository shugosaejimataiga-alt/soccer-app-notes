
■ Day7 完全まとめ（バリデーション）


■ ① 今日やったこと

● store（新規作成）にバリデーション追加
● update（更新）にバリデーション追加

👉 フロントから送られてくるデータをそのまま使うのを禁止
👉 必ずチェックを通したデータだけ使うようにした



■ ② バリデーション内容

$validated = $request->validate([
    'team_id' => 'required|exists:teams,id',
    'name' => 'required|string',
    'position' => 'required|in:GK,DF,MF,FW',
]);


■ 各ルールの意味

・team_id
→ 必須
→ teamsテーブルに存在するIDのみ許可

・name
→ 必須
→ 文字列のみ

・position
→ 必須
→ GK / DF / MF / FW のみ許可



■ ③ is_activeの扱い（重要）

$validated['is_active'] = true;

👉 サーバー側で強制的にtrueを設定
👉 フロントから送られても無視する


■ 理由

・is_activeは削除管理用
・ユーザーに操作させない
・作成時は必ず有効状態にする



■ ④ updateの安全化

$player->update($validated);

👉 validate済みデータのみで更新
👉 $request->all() は使用禁止



■ ⑤ PHPの読み方（復習）

$〇〇 = 〇〇〇〇;

👉 右を実行 → 左に入れる


■ 具体例

$validated = $request->validate(...)

→ validate結果を$validatedに入れる


$player = Player::findOrFail($id)

→ DBから取得して$playerに入れる


$player->update($validated)

→ $playerを$validatedの内容で更新する



■ ⑥ 現在のController（完成形）

public function store(Request $request)
{
    $validated = $request->validate([
        'team_id' => 'required|exists:teams,id',
        'name' => 'required|string',
        'position' => 'required|in:GK,DF,MF,FW',
    ]);

    $validated['is_active'] = true;

    return Player::create($validated);
}

public function update(Request $request, $id)
{
    $player = Player::findOrFail($id);

    $validated = $request->validate([
        'team_id' => 'required|exists:teams,id',
        'name' => 'required|string',
        'position' => 'required|in:GK,DF,MF,FW',
    ]);

    $player->update($validated);

    return $player;
}


■ ⑦ 到達状態

・不正データはすべて弾かれる
・安全なデータのみDBに保存・更新
・フロントの入力を信用しない設計


■ 核（重要）

・データの真実はバックエンド
・入力はすべて疑う
・必ず検証してから使う