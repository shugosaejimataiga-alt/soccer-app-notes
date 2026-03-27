

■ Day4まとめ（論理削除）


■ 実装内容（核心）

● 削除を「物理削除 → 論理削除」に変更

const deletePlayer = (id: number) => {
  const updatedPlayers = players.map((player) => {
    if (player.id === id) {
      return { ...player, isActive: false }
    }
    return player
  })

  setPlayers(updatedPlayers)
}



■ 表示制御

props.players
  .filter((player) => player.isActive)
  .map(...)

👉 isActive=true のみ表示



■ UI実装

<button onClick={() => props.deletePlayer(player.id)}>
  削除
</button>

👉 各playerごとに削除ボタンを設置



■ データの流れ（重要）

ユーザー操作
→ 削除ボタン押す
→ player.id が渡る
→ deletePlayer実行
→ isActive:false に変更
→ state更新
→ 再描画
→ filterで非表示



■ mapの理解（最重要）
全員を1人ずつ処理
条件に合う人だけ変更
配列はそのまま残る



■ filterの理解
条件に合うものだけ残す
表示制御に使う



■ findとの違い
filter → 配列
map → 配列
find → 単体



■ 設計の本質
データは消さない
状態で制御する
後で復活できる構造