

■ ① 今日やったこと（本質）

・フロント（React）からバックエンド（Laravel）へ接続
・GET /players を実装
・API → state → UI の流れを構築
・フロントを「UI + API接続」に進化させた



■ ② 全体構造（最重要）

React（Players.tsx）
↓
getPlayers（player.ts）
↓
fetch（HTTPリクエスト）
↓
URL（/api/players）
↓
nginx
↓
Laravel（Route）
↓
Controller
↓
データ返却（JSON）
↓
res
↓
data.data
↓
players（state）
↓
PlayerList
↓
画面表示



■ ③ 用語の理解（核心）

■ URL
👉 機能の入口（例：/api/players）

■ HTTPリクエスト
👉 サーバーへの命令（GETなど）


■ HTTPメソッド

GET    → 取得
POST   → 作成
PUT    → 更新
DELETE → 削除


■ CRUD対応

Create → POST
Read   → GET
Update → PUT
Delete → DELETE


■ res
👉 サーバーからの返事（結果）


■ data.data
👉 実際の中身だけ取り出している



■ ④ フロント設計（重要）

■ 分離構造
Players.tsx → UI・状態管理
player.ts   → API通信



■ 役割

■ Players.tsx
・useEffectでAPI呼び出し
・state管理（players）
・子へデータ渡す

■ player.ts
・fetchでAPI呼ぶ
・クエリ作成
・レスポンス整形

👉 責務分離（実務レベル）



■ ⑤ 実装の流れ

■ ① API関数

getPlayers()



■ ② useEffect

useEffect(() => {
  fetchPlayers()
}, [])

👉 初回のみ実行



■ ③ state
const [players, setPlayers] = useState<Player[]>([])



■ ④ データ流れ
API → players → PlayerList



■ ⑤ 表示
players.map(...)



■ ⑥ 重要ポイント

■ async / await
👉 非同期処理を待つ

■ URLSearchParams
👉 クエリ安全生成

■ res.ok
👉 成功判定

■ 無限ループ対策

useEffect(..., [])



■ ⑦ 型の扱い

■ Player型
👉 stateで使用（必須）

■ Props型（Players.tsx）
👉 不要（削除）



■ ⑧ 到達レベル
API接続（GET） 完全完成
フロント ⇄ バックエンド 接続完了



■ ⑨ 本質（最重要）
URL = 機能の入口
HTTP = 命令
Route = 振り分け
Controller = 処理
res = 結果



■ ⑩ 1行まとめ

👉 ReactからAPIを呼び、取得したデータをstate経由でUIに表示する仕組みを完成させた