

■ Day8 完全まとめ


■ 今日やったこと（実装）

● 詳細ページ作成（PlayerDetail.tsx）
・/players/:id で画面表示


● id取得

const { id } = useParams()

👉 URLから取得（string）


● 1人の選手を取得

const player = props.players.find(
  (p) => p.id === Number(id)
)

👉 配列から該当データを探す


● 画面表示

<p>{player?.name}</p>
<p>{player?.position}</p>

👉 optional chainingで安全に表示


● 編集ページへの遷移

navigate(`/players/${id}/edit`)

👉 動的URLで遷移



■ 学んだこと（核心理解）


● URLが画面を決める

/players/:id → PlayerDetail


● useParamsの役割
👉 データではなく「idを取得するだけ」


● データ取得の流れ

URL → id取得 → 配列からfind → 1人取得


● 型の違い

id（URL） = string
id（データ） = number
👉 Number()で変換


● findの意味
👉 配列から条件一致の1件取得


● optional chaining

player?.name
👉 undefinedでもエラー回避


● ルーティングの本質
👉 URLが主役
👉 コンポーネントは結果


● データ設計
App（state）
 ↓
pages
 ↓
components

👉 データは1箇所に集約



■ 最重要まとめ（1行）

👉 URL → id取得 → 配列から1人取得 → 表示