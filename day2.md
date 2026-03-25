📘 Day2 完全まとめ（Tailwind導入）
■ 目的

Reactにデザイン（UI）をつけるための仕組みを導入する

■ やったこと（流れ）
① Tailwindインストール
npm install -D tailwindcss@3 postcss autoprefixer

👉 CSSフレームワークを追加

② 設定ファイル作成
npx tailwindcss init -p

👉 以下2つが生成された

tailwind.config.js
postcss.config.js
③ Tailwindの対象範囲設定
content: [
  "./index.html",
  "./src/**/*.{js,ts,jsx,tsx}",
],

👉 classを検知するファイルを指定

④ CSSにTailwind読み込み

src/index.css の先頭に追加

@tailwind base;
@tailwind components;
@tailwind utilities;

👉 Tailwindの本体を読み込む

⑤ 動作確認
export default function App() {
  return (
    <div className="bg-red-500 text-white p-10">
      Tailwind 動作確認
    </div>
  )
}
npm run dev

👉 赤背景表示 → 成功

■ 学んだこと（本質）
① Tailwindとは

👉 classでデザインを書くCSS

② 最重要仕組み

👉 使ったclassだけCSSが生成される

（軽量・高速）

③ content設定の意味

👉 どのファイルを監視するか

④ index.cssの3行

👉 Tailwindを読み込む本体

⑤ node_modulesの理解

👉 フォルダごとに依存関係を持つ

■ 現在の状態
React起動OK
Tailwind使用可能
UI開発準備完了