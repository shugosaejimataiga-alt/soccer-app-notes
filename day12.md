■ Day9：編集機能（完全まとめ）


■ ① 全体の流れ（最重要）
URL
↓
useParamsでid取得
↓
findで対象player取得
↓
存在チェック（if）
↓
useStateに初期値セット
↓
inputで編集
↓
updatePlayer実行
↓
navigateで詳細に戻る



■ ② データの流れ（核心）

App（state本体）
↓ propsで渡す
PlayerEdit
↓
updatePlayer実行
↓
AppのsetPlayersが動く
↓
画面更新



■ ③ 編集の本質

👉 元データは直接触らない

player（元データ）
↓
useStateでコピー
↓
コピーを編集
↓
updatePlayerで本体更新



■ ④ useStateの意味
const [name, setName] = useState(player.name)

👉 入力欄の状態を持つ



■ ⑤ player取得
const player = props.players.find(
  (p) => p.id === Number(id)
)

👉 配列から1人特定



■ ⑥ 重要：存在チェック（ガード処理）
if (!player) {
  return <div>選手が存在しません</div>
}

👉 存在しないデータを防ぐ



■ ⑦ ? と || の正体（分離理解）
player?.name || ""

👉 ? → playerがあるかチェック
👉 || → 値が無いときの代替

👉 別物



■ ⑧ updatePlayerの役割
props.updatePlayer(id, name, position)

👉 Appのstateを更新する関数



■ ⑨ handle関数の意味
const handleUpdate = () => { ... }

👉 イベント処理の入口

👉 可読性・拡張性のため



■ ⑩ 保存処理
<button onClick={handleUpdate}>
const handleUpdate = () => {
  props.updatePlayer(Number(id), name, position)
  navigate(`/players/${id}`)
}

👉 更新＋画面遷移



■ ⑪ inputとselect
<input value={name} onChange={...} />
<select value={position} onChange={...} />

👉 stateと連動



■ ⑫ 超重要理解（TypeScript）

👉 findは

見つかる or undefined

👉 だから

👉 必ず対応する必要がある



■ ⑬ 概念まとめ

Props → データと関数を渡す
useState → 入力状態
updatePlayer → 本体更新
navigate → 画面遷移



■ 最短まとめ（1行）

👉 編集 = 取得 → コピー → 編集 → 更新 → 遷移