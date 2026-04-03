■ Day13 完全まとめ（並び替え）


■ ① 今日やったこと（全体）

● 並び替え機能の実装

並び替え用stateを追加

selectで切り替えUI作成

sortで並び替え処理実装


● データ処理の流れを完成
players
 ↓
検索（name）
 ↓
フィルタ（position）
 ↓
並び替え（sort）
 ↓
PlayerListへ渡す

👉 一覧処理が完全に完成



■ ② コード構造（最重要）

● Players.tsx（データ処理）

const filteredPlayers = props.players
  .filter((player) => player.isActive)
  .filter((player) => player.name.includes(searchName))
  .filter((player) =>
    filterPosition === "ALL"
      ? true
      : player.position === filterPosition
  )

const sortedPlayers = [...filteredPlayers].sort((a, b) => {

  if (sortType === "name") {
    return a.name.localeCompare(b.name)
  }

  const order = {
    GK: 0,
    DF: 1,
    MF: 2,
    FW: 3
  }

  return order[a.position] - order[b.position]
})
● PlayerList.tsx（表示）
props.players.map(...)

👉 渡された順番で表示するだけ



■ ③ 学んだこと（核心理解）

● ① データとUIの役割分離
Players → データを作る
PlayerList → 表示する

👉 ロジックは親に集約


● ② propsの本質
<PlayerList players={sortedPlayers} />

👉 sortedPlayersがそのまま渡る

props.players = sortedPlayers

👉 データがそのまま流れる


● ③ sortの仕組み
.sort((a, b) => return ???)

👉 2つを比較して順番を決める

ルール
マイナス → aが前
プラス → bが前
0 → 同じ


● ④ localeCompare
a.name.localeCompare(b.name)

👉 文字列の比較

👉 辞書順（アルファベット順）になる


● ⑤ カスタム並び替え
const order = {
  GK: 0,
  DF: 1,
  MF: 2,
  FW: 3
}

👉 文字 → 数値に変換

👉 数値で並び替える


● ⑥ 並び替えの設計

👉 「1つだけ適用する」

if (sortType === "name") {
  return 名前順
} else {
  return ポジション順
}

👉 混ぜるとバグになる


● ⑦ 配列コピーの重要性
[...filteredPlayers]

👉 元データを壊さない

👉 Reactでは必須ルール



■ ④ 最終状態

● 一覧機能
表示
追加
削除（論理）
詳細
編集
検索
フィルタ
並び替え

👉 完全実装



■ ⑤ 本質まとめ（最重要）

👉 一覧は「加工パイプライン」

データ
 ↓
絞る
 ↓
並べる
 ↓
表示する

👉 表示は最後
👉 それまでに全部決める



■ ⑥ 状態管理構造
App（データ）
 ↓
Players（加工）
 ↓
PlayerList（表示）

👉 データは上、UIは下