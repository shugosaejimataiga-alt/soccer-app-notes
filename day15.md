Day12 完全まとめ（検索・フィルタ）

■ 実装した機能

・名前検索
・ポジション絞り込み
・複合条件フィルタ


■ 実装の流れ（本質）

① 検索条件をstateで持つ

const [searchName, setSearchName] = useState("")
const [filterPosition, setFilterPosition] = useState<"ALL" | "GK" | "DF" | "MF" | "FW">("ALL")


② 入力UI → stateに反映

<input
  value={searchName}
  onChange={(e) => setSearchName(e.target.value)}
/>

<select
  value={filterPosition}
  onChange={(e) =>
    setFilterPosition(e.target.value as "ALL" | "GK" | "DF" | "MF" | "FW")
  }
/>


③ filterで絞る（最重要）

props.players
  .filter((player) => player.isActive)
  .filter((player) => player.name.includes(searchName))
  .filter((player) =>
    filterPosition === "ALL"
      ? true
      : player.position === filterPosition
  )


④ 絞ったデータを子へ渡す

<PlayerList players={フィルタ済み配列} />



■ 重要理解（設計）

・状態 → 親（Players）
・ロジック（filter）→ 親
・表示（map）→ 子（PlayerList）



■ filterの意味

・isActive → 論理削除の制御
・includes → 部分一致検索
・ALL → フィルタ無効
・それ以外 → 完全一致



■ TypeScript理解

・selectはstringで来る
・asで型を絞る必要がある

e.target.value as "ALL" | "GK" | "DF" | "MF" | "FW"



■ 最重要ポイント

👉 「表示するデータを親が決める」