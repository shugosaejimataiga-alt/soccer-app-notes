■ Day11 完全まとめ（UI整備）


■ ① モーダル風遷移の実装（最重要）

■ やったこと

・useLocationを使った背景保持
・state: { background: location }で遷移
・App.tsxで二重Routes構成

■ 構造
一覧（背景）
＋
Create / Edit（前面）

■ コード核心
<Routes location={location.state?.background || location}>

👉 背景を制御

{location.state?.background && <Routes>...</Routes>}

👉 モーダル表示

■ 本質

👉 URLはそのまま、表示だけ変える



■ ② UIの基本構造理解

■ レイヤー
背景（一覧）
↓
半透明レイヤー
↓
中央カード
↓
フォーム

■ UIの3要素

・配置（flex / center）
・サイズ（max-w）
・余白（p / gap）

👉 UIはこれだけで成立する



■ ③ グリッドレイアウト

■ 概念

👉 縦横のマスで並べる

grid grid-cols-3 gap-4

■ 問題と解決

・3列で潰れる
→ 親が狭いのが原因

■ 修正
max-w-md → max-w-5xl

👉 親の幅が重要



■ ④ 表示とロジックの分離（重要）

■ Players.tsx
👉 ロジック

・検索
・フィルタ
・並び替え

■ PlayerList.tsx
👉 表示

・カードUI
・グリッド
・ポジション分け

■ 原則
ロジック → 親
表示 → 子



■ ⑤ ポジションカラー

■ 実装
const positionStyle = {
  GK: "text-yellow-600 bg-yellow-100",
  ...
}

■ 適用
className={`... ${positionStyle[player.position]}`}

■ 本質
👉 データ → 見た目に変換



■ ⑥ 共通化（設計）

■ ファイル
src/constants/positionStyle.ts

■ 理由
・再利用
・一元管理
・変更しやすい

■ 分類
components → UI部品
constants → 定義



■ ⑦ 1行文字問題

■ 解決
break-words
👉 折り返して全表示

■ NG
truncate
👉 情報が消える



■ ⑧ 丸バッジUI

■ 実装
w-8 h-8 flex items-center justify-center rounded-full

■ 本質
👉 サイズ固定で円になる



■ ⑨ フロントとバックエンドの理解

■ バックエンド
・CRUD
・検索
・フィルタ
・並び替え

■ フロント
・UI
・レイアウト
・モーダル
・色

■ 今の状態

👉 フロントで仮実装



■ ⑩ 編集ページUI

■ 方針
・Createと同じ構造
・モーダル
・中央カード

■ 本質

👉 UIは共通、処理だけ違う