

■ Day6まとめ（完全版）


■ 今日やったこと

● ① 一覧の表示制御

isActive=true のみ表示
論理削除した選手を非表示にした
players.filter((player) => player.isActive)


● ② カードUIに変更

Tailwindで見た目を整えた
横幅制限で見やすさ改善
className="rounded-xl shadow p-4 bg-white mb-2"
<div className="max-w-md mx-auto">


● ③ 詳細ページへの遷移（最重要）

カードクリックで詳細へ移動
const navigate = useNavigate()

onClick={() => navigate(`/players/${player.id}`)}



■ 学んだこと（核心理解）

● useNavigate

react-router-domから取得する
URLを変更する関数


● navigate

指定したURLへ移動する


● ルーティングの本質（最重要）

👉 自分は「URL」を指定しているだけ
👉 表示コンポーネントはRouteが決める


● Routeの役割

<Route path="/players/:id" element={<PlayerDetail />} />
URLとコンポーネントを結びつける


● :id の意味

動的パラメータ
/players/1 → id=1


● 流れ（完全理解）

navigate("/players/1")
URL変更
Route一致
PlayerDetail表示


● 絶対パスと相対パス

/players/... → 正解（絶対）
./players/... → NG（相対）


● 設計理解

URLが主役
コンポーネントは結果


■ 現在のコード状態

● PlayerList.tsx（完成）

import { useNavigate } from "react-router-dom"

export default function PlayerList(props: Props) {

  const navigate = useNavigate()

  return (
    <div>
      {props.players
        .filter((player) => player.isActive)
        .map((player) => (
          <div
            key={player.id}
            className="rounded-xl shadow p-4 bg-white mb-2 cursor-pointer"
            onClick={() => navigate(`/players/${player.id}`)}
          >
            <p className="font-bold">{player.name}</p>
            <p>{player.position}</p>

            <button
              className="mt-2 bg-red-500 text-white px-2 py-1 rounded"
              onClick={() => props.deletePlayer(player.id)}
            >
              削除
            </button>
          </div>
        ))}
    </div>
  )
}