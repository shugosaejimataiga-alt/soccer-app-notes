

■ Day1 完全まとめ（Players機能）


■ 実装の流れ（本質）

① データの形を決める
② データを保存する
③ 仮データを入れる
④ 画面に表示する



■ 型定義（データ設計）

type Player = {
  id: number
  teamId: number
  name: string
  position: "GK" | "DF" | "MF" | "FW"
  photoUrl?: string
  isActive: boolean
}

・アプリで扱う「1人分の選手データ」を定義
・positionは固定値（Union型）
・photoUrlは任意（?）



■ 状態管理（最重要）

const [players, setPlayers] = useState<Player[]>([])

・players → データ本体
・setPlayers → 更新用
・Player[] → 型付き配列

👉 状態はここに集約される（最重要設計）



■ 仮データ（動作確認用）
const [players, setPlayers] = useState<Player[]>([
  {
    id: 1,
    teamId: 1,
    name: "田中 太郎",
    position: "FW",
    isActive: true
  },
  {
    id: 2,
    teamId: 1,
    name: "佐藤 次郎",
    position: "MF",
    isActive: true
  }
])

・最初から配列にデータを入れる
・画面表示のテスト用



■ 一覧表示（map）

{players.map((player) => (
  <div key={player.id}>
    <p>{player.name}</p>
    <p>{player.position}</p>
  </div>
))}

・配列 → 1つずつ取り出す
・player → 1人分
・key → 必須（識別用）

👉 「データ → UI」に変換している



■ ルーティング連携

import Players from "./Players"

<Route path="/players" element={<Players />} />

・Players.tsxを読み込む
・URLで表示を切り替える



■ 最重要理解（核心）

・状態（useState）がデータの中心
・mapで「データをUIに変換」する
・型（type）が全ての土台
・コンポーネントは関数で動く