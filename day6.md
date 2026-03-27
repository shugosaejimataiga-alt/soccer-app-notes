

■ Day3まとめ（完全版）


■ 今日やったこと


● 一覧表示の分離（最重要）
Players.tsxにあったmap表示を削除し、PlayerList.tsxに移動


● PlayerList.tsx作成
players（配列）をPropsで受け取り、mapで表示


● 親→子 Props連携

<PlayerList players={players} />

👉 stateを子コンポーネントに渡した


● コンポーネント分割完成

Players → state管理
PlayerForm → 入力UI
PlayerList → 表示UI



■ 現在のファイル構成
src
 ├ Players.tsx
 └ components
     ├ PlayerForm.tsx
     └ PlayerList.tsx



■ 現在のコード状態（重要部分）

● Players.tsx（責務：状態管理）

import { useState } from "react"
import PlayerForm from "./components/PlayerForm"
import PlayerList from "./components/PlayerList"

type Player = {
  id: number
  teamId: number
  name: string
  position: "GK" | "DF" | "MF" | "FW"
  photoUrl?: string
  isActive: boolean
}

export default function Players() {
  
  const [players, setPlayers] = useState<Player[]>([])

  const addPlayer = (name: string, position: "GK" | "DF" | "MF" | "FW") => {
    const newPlayer = {
      id: Date.now(),
      teamId: 1,
      name,
      position,
      isActive: true
    }

    setPlayers([...players, newPlayer])
  }

  return (
    <div>
      <h1>Players</h1>

      <PlayerForm addPlayer={addPlayer} />

      <PlayerList players={players} />
    </div>
  )
}


● PlayerList.tsx（責務：表示）

type Player = {
  id: number
  teamId: number
  name: string
  position: "GK" | "DF" | "MF" | "FW"
  photoUrl?: string
  isActive: boolean
}

type Props = {
  players: Player[]
}

export default function PlayerList(props: Props) {
  return (
    <div>
      {props.players.map((player) => (
        <div key={player.id}>
          <p>{player.name}</p>
          <p>{player.position}</p>
        </div>
      ))}
    </div>
  )
}



■ 学んだこと（核心）

● Propsの本質
👉 親が子にデータを渡す仕組み

<PlayerList players={players} />
👉
props = { players: players }


● 型の理解

Player → 1人
Player[] → 複数人
Props → データの設計図
props → 実データ


● Reactの流れ（超重要）

入力
addPlayer実行
setPlayersで更新
再レンダリング
props更新
UI更新

👉 state変更 → 画面更新


● コンポーネント設計（最重要）

親：データ管理
子：UI担当

👉 責務分離