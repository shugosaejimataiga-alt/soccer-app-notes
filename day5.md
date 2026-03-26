■ Day2 完全まとめ（Players機能）


● 今日やったこと

① フォーム作成
PlayerForm.tsx を作成
名前入力（input）
ポジション選択（select）
追加ボタンを設置

② state管理（フォーム）
name を useStateで管理
position を useStateで管理

👉 入力内容をReactで管理（制御コンポーネント）


③ 親 → 子 Props
<PlayerForm addPlayer={addPlayer} />

👉 親の関数を子に渡す


④ 子 → 親 呼び出し
props.addPlayer(name, position)

👉 子から親の関数を実行


⑤ 追加処理（核心）
const addPlayer = (...) => {
  const newPlayer = { ... }
  setPlayers([...players, newPlayer])
}

👉 配列に新しい選手を追加



● データの流れ（最重要）

① 入力
→ onChange

② state更新
→ name / position

③ ボタン押下
→ addPlayer実行

④ newPlayer作成
→ 配列に追加

⑤ 再レンダリング
→ 一覧更新



● Reactの本質理解

コンポーネント = 関数
初回表示で実行される
stateが変わると再実行される

👉 再レンダリング = 関数の再実行



● value の意味

value={name}

👉 inputの値 = state
👉 Reactが入力を完全管理



● Propsの本質

addPlayer={addPlayer}
右：親の関数
左：子で使う名前

👉 親 → 子へ機能を渡す



● 設計理解

状態 → Playersに集約
UI → PlayerFormに分離

👉 責務分離