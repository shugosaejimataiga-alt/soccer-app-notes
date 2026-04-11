
■ Day6：論理削除 完全まとめ


■ やったこと
DELETE処理を物理削除から論理削除に変更
GET処理で削除済みデータを除外


■ 修正内容

● destroy（削除）

delete() を廃止
↓
is_active = false に変更して保存


● index（一覧取得）

Player::all() を廃止
↓
is_active = true のみ取得



■ 処理の流れ
① 対象データ取得
② is_active を false に変更
③ save() でDBに反映


■ 重要理解
削除 = データを消すではない
削除 = 非表示にする



■ 設計の意味
・データは失われない
・復元可能
・履歴管理できる



■ 現在のAPI挙動

DELETE /players/{id}
→ is_active = false

GET /players
→ is_active = true のみ返す