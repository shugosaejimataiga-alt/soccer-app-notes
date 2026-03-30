

Day7 完全まとめ（復元用・最終版）

■ 今日やったこと（全体）
PlayersにあったstateをAppに移動
→ 複数ページでデータ共有できる構造に変更

作成ページ（PlayerCreate）を実装
→ フォーム入力 → 追加 → 一覧へ戻る

削除処理もAppに移動
→ 一覧から削除が機能

イベント伝播バグを修正
→ 削除ボタンが正常動作


■ 最重要変更（核心）

stateの場所を変更した

Before
Players.tsx（1画面専用）

After
App.tsx（全ページ共有）



■ 構造（完成形）

App（データ管理）
 ↓
pages（画面）
 ↓
components（部品）



■ 各役割

App.tsx
・playersを持つ
・addPlayer / deletePlayerを持つ
・Routeで画面切替

Players.tsx
・一覧表示
・作成ページへ遷移
・PlayerListへデータ渡す

PlayerCreate.tsx
・フォーム表示
・追加処理を呼ぶ
・追加後に一覧へ戻る

PlayerForm.tsx
・入力UI
・入力値を親に渡す

PlayerList.tsx
・一覧表示
・削除ボタン
・詳細ページへ遷移



■ データの流れ

App（players）
 ↓
Players
 ↓
PlayerList



■ 処理の流れ（追加）

PlayerForm
 ↓
handleAdd（PlayerCreate）
 ↓
addPlayer（App）
 ↓
players更新
 ↓
一覧再描画



■ 処理の流れ（削除）

削除ボタン
 ↓
deletePlayer（App）
 ↓
isActive=false
 ↓
filterで非表示



■ ルーティングの流れ

navigate("/players/create")
 ↓
URL変更
 ↓
Routeが一致
 ↓
PlayerCreate表示



■ 今日のバグと解決

問題
削除ボタン押すと詳細に遷移してしまう

原因
子（button）→ 親（div）にクリックが伝播

解決
e.stopPropagation()

■ stopPropagationの意味
子のクリックを親に伝えない



■ 重要理解（本質）

データは1箇所（App）に置く

データは上から下へ流れる
処理は下から上へ戻る

URLが画面を決める



■ 今日の到達状態

一覧 → 作成 → 保存 → 一覧
削除も正常動作

👉 CRUDの土台完成