

■ Day5まとめ（完全版）


■ 今日やったこと

● ① ファイル構造の変更（最重要）

・pagesフォルダを作成
・画面ごとにファイルを分離

src
 ├ pages
 │   ├ Players.tsx
 │   ├ PlayerCreate.tsx
 │   ├ PlayerDetail.tsx
 │   └ PlayerEdit.tsx
 └ components
     ├ PlayerForm.tsx
     └ PlayerList.tsx


● ② importパス修正
import PlayerForm from "../components/PlayerForm"
import PlayerList from "../components/PlayerList"

・階層が変わったため ../ が必要になった


● ③ ルーティング実装（核心）

<Route path="/players" element={<Players />} />
<Route path="/players/create" element={<PlayerCreate />} />
<Route path="/players/:id" element={<PlayerDetail />} />
<Route path="/players/:id/edit" element={<PlayerEdit />} />


● ④ Sidebarから遷移確認

<Link to="/players">Players</Link>

・クリックで画面遷移できる状態になった



■ 学んだこと（最重要）

● ① importの本質
import = コンポーネントを使える状態にする（準備）

・まだ画面には表示されない


● ② Routeの本質
Route = URLと画面を結びつけるルール


● ③ 表示の仕組み
① import
② Route登録
③ URLアクセス
④ 一致したものが表示


● ④ :id の意味
/players/1
/players/2

・どの選手かを識別する値
・画面は1つでデータだけ変わる


● ⑤ ファイルとデータの違い（重要）
ファイル → 画面の型（1つ）
id → 表示するデータ


● ⑥ SPAの理解

・ページリロードしない
・URLだけ変わる
・中身だけ切り替わる


■ 現在のコード状態（要点）

● App.tsx
import Players from "./pages/Players"
import PlayerCreate from "./pages/PlayerCreate"
import PlayerDetail from "./pages/PlayerDetail"
import PlayerEdit from "./pages/PlayerEdit"

<Routes>
  <Route path="/players" element={<Players />} />
  <Route path="/players/create" element={<PlayerCreate />} />
  <Route path="/players/:id" element={<PlayerDetail />} />
  <Route path="/players/:id/edit" element={<PlayerEdit />} />
</Routes>


● Players.tsx（変更点）
import PlayerForm from "../components/PlayerForm"
import PlayerList from "../components/PlayerList"



■ 到達状態

・画面が4つに分離された
・URLで画面が切り替わる
・SPAルーティングが完成
・1画面1目的の設計になった



■ 本質まとめ（最重要）

画面 = コンポーネント
URL = 表示条件
Route = 条件分岐