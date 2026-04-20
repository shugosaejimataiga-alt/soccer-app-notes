

■ Day10 完全まとめ（並び替え）


■ 目的

👉 GET /players に並び替え（sort）を追加



■ 全体の流れ

request → validate → query構築 → DB実行



■ request（入力）

/api/players?name=田中&position=MF&sort=name

👉 ユーザーの操作はすべてURLになる



■ validate（入力チェック）

$request->validate([
    'name' => ['nullable', 'string'],
    'position' => ['nullable', 'in:GK,DF,MF,FW'],
    'sort' => ['nullable', 'in:name,position']
]);

意味

nullable → 入力しなくてもOK

string → 文字列のみ

in → 許可された値のみ

👉 不正な入力はここで止める



■ query（DBへの指示）

$query = Player::query();

$query->where('is_active', true);

👉 論理削除済みは除外



■ 絞り込み

if ($request->has('name')) {
    $query->where('name', 'LIKE', '%' . $request->name . '%');
}

if ($request->has('position')) {
    $query->where('position', $request->position);
}

👉 条件がある時だけ適用



■ 並び替え

if ($request->sort === 'name') {
    $query->orderBy('name');
} elseif ($request->sort === 'position') {
    $query->orderByRaw("FIELD(position, 'GK', 'DF', 'MF', 'FW')");
} else {
    // デフォルト
    $query->orderByRaw("FIELD(position, 'GK', 'DF', 'MF', 'FW')");
}



■ 並び替えの種類

name → 名前順
position → GK→DF→MF→FW順

👉 orderByRawでカスタム順を実現



■ DB実行

return $query->get();

👉 ここで初めてSQLが実行される



■ 重要な理解

● validateとqueryの違い

validate → 入力チェック
query → DBへの命令



● whereとorderByの違い

where → データを絞る
orderBy → 順番を変える


● nullableの意味

👉 入力しなくてもOK



● デフォルト並び

👉 sortが無い場合に適用




■ 最終状態

👉 検索 + フィルタ + 並び替えが同時に可能