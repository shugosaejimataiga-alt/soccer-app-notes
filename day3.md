📘 Day3まとめ（完全版）
■ 今日やったこと
① Sidebarコンポーネント作成
Sidebar.tsx を作成
メニューUIを部品として分離

👉 UIをコンポーネント単位で管理

② AppにSidebarを配置
<Sidebar />

👉 作った部品を画面に表示

③ flexでレイアウト作成
<div className="flex">

👉 横並びレイアウト

④ Sidebar + Main構造作成
[ Sidebar ][ Main ]

👉 画面構造を分離

⑤ MainエリアにRoutes配置
<Routes>...</Routes>

👉 表示エリアを固定して中身だけ変える

⑥ ルーティング設定
<Route path="/players" element={<div>Players</div>} />

👉 URLごとに画面を切り替え

⑦ BrowserRouter設定（main.tsx）
<BrowserRouter>
  <App />
</BrowserRouter>

👉 ルーティング機能を有効化

⑧ SidebarにLink追加
<Link to="/players">Players</Link>

👉 クリックでページ移動

⑨ Sidebarのサイズ調整
w-64 min-h-screen

👉 左側固定の縦メニューに

⑩ Mainを可変幅に
flex-1

👉 残りの幅を全部使う

⑪ #root問題の修正（超重要）
#root {
  width: 100%;
}

👉 中央寄せを解除

■ 最終構造
App
 ├ Sidebar（左・固定）
 └ Main（右・切り替え）
     └ Routesでページ変更
■ 学んだこと（本質）
① コンポーネントの役割分離
App → 配置・全体構造
Sidebar → 部品（UI）
② レイアウトは親が決める

👉 「どこに置くか」はApp

③ UIは子が決める

👉 「見た目」はSidebar

④ flexの本質
flex → 横
flex-col → 縦
⑤ flex-1の意味

👉 残りスペースを全部使う

⑥ ルーティングの仕組み
BrowserRouter → 有効化
Routes → 管理
Route → 1ページ
Link → 移動
⑦ UI設計の基本構造
固定部分 + 可変部分

👉 Sidebar（固定）＋ Main（変化）