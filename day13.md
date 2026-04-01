■ Day10まとめ（削除 完成）


■ 今日やったこと

● ① 詳細ページに削除機能を追加
👉 PlayerDetail.tsxで削除を実装

const handleDelete = () => {
  props.deletePlayer(Number(id))
  navigate("/players")
}


● ② App → Detailへ関数を渡した

<PlayerDetail
  players={players}
  deletePlayer={deletePlayer}
/>
👉 親の処理を子で使えるようにした


● ③ 削除ボタン追加

<button onClick={handleDelete}>
  削除
</button>


● ④ 削除後の遷移

👉 navigate("/players")
👉 一覧へ戻る動線を作成


● ⑤ エラー修正（重要）

deletPlayer → deletePlayer
👉 スペルミスで型不一致エラー



■ 学んだこと（核心理解）

● データの流れ（最重要）

App（state管理）
↓
propsで渡す
↓
PlayerDetail
↓
deletePlayer実行
↓
setPlayers
↓
再描画

👉 これがReactの基本構造


● 関数もpropsで渡せる

👉 データだけじゃない
👉 「処理」も子に渡せる


● 論理削除の完成

isActive = false

👉 データは消さない
👉 表示だけ消す


● UIとロジックの分離

onClick={handleDelete}

👉 直接書かない
👉 関数にまとめる


● 型の一致が絶対条件

👉 渡す側と受け取る側は完全一致

deletePlayer ≠ deletPlayer
👉 1文字でも違うとエラー


● 画面遷移の役割

👉 処理後の「次の行動」を作る

navigate("/players")



■ Day10の本質（1行）

👉 どの画面からでも削除できる設計にした



■ 現在の到達状態

✔ 一覧表示
✔ 作成
✔ 詳細表示
✔ 編集
✔ 削除（一覧・詳細どちらからも可能）

👉 CRUDすべて完成