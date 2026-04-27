

■ Day14（POST + 詳細画面 + バグ修正）完全まとめ

【① 今日やったこと】

・POST /players（作成API）をReactから接続
・フォーム → 親 → API の流れを構築
・一覧クリック → 詳細画面遷移実装
・GET /players/:id（単体取得）実装
・PlayerDetailでデータ取得＆表示
・追加後に一覧が更新されない問題を修正


【② フロント → バックエンドの流れ（POST）】

PlayerForm（入力）
↓
onCreate(name, position)
↓
PlayerCreate（handleCreate）
↓
createPlayer（api層）
↓
fetch POST /players
↓
Laravel Controller（store）
↓
DB保存
↓
navigate("/players")
↓
location変化
↓
useEffect再実行
↓
一覧更新


【③ API層（重要設計）】

・api/player.tsに通信を集約
→ UIと通信を分離

getPlayers → 一覧取得
createPlayer → 作成
getPlayer → 単体取得

👉 実務でもこの構成が基本


【④ POSTの仕組み】

fetch(url, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({...})
})

・method → 通信種別（作成）
・headers → JSON形式宣言
・body → 送信データ


【⑤ 非同期処理（超重要）】

await createPlayer(...)

・API完了まで待つ
・その後に画面遷移

👉 順序保証のため


【⑥ 詳細画面の流れ】

/players/:id に遷移
↓
useParams()でid取得
↓
useEffectでAPI実行
↓
getPlayer(id)
↓
setPlayer
↓
表示


【⑦ Loadingが消えなかった原因】

原因①：useEffectがreturnより下 → 実行されない
原因②：catchのスペルミス
原因③：show未実装（バックエンド）

👉 useEffectは必ずreturnより上


【⑧ 一覧が更新されなかった原因】

useEffectの依存配列

[] → 初回のみ実行

👉 修正

[location] → 画面戻るたび再取得


【⑨ バリデーション問題】

'team_id' => ['required', 'exists:teams,id']

👉 teamsテーブルが無いため失敗

修正：

'team_id' => ['required']

👉 一時的に通す


【⑩ DB・Resourceの理解】

・DB構造 → すでに正しい
・Resource → レスポンス整形
・保存処理 → 正常

👉 今回の問題はバリデーションのみ


【⑪ 最重要設計思想】

・データの真実はバックエンド
・フロントはUI＋API接続のみ
・通信はapi層に集約
・処理はすべてAPI経由


【⑫ 現在の完成度】

・一覧取得 ✔
・作成 ✔
・詳細表示 ✔
・画面遷移 ✔
・再取得 ✔



【⑬ 次にやること】

・DELETE API接続
・UPDATE API接続
・teamsテーブル導入
・検索/フィルタをAPI連動


【最重要まとめ】

・「保存できない」→ バリデーション
・「表示されない」→ useEffect or API
・「更新されない」→ state再取得不足

👉 問題は必ず「どこで止まっているか」で判断する