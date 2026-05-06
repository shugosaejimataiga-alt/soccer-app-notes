

■ Day14：検索・絞り込み・並び替え API接続 完全まとめ

───────────────────────────────────

■ 今日の目的

React側で持っていた：

・検索
・絞り込み
・並び替え

を、

Laravel APIへ query として送る構成へ変更した。

つまり：

フロントで検索する構成
↓
バックエンドへ条件を送る構成

へ変更した。

───────────────────────────────────

■ 最重要思想

・データの真実はバックエンド
・検索ロジックはLaravel
・ReactはUIと状態管理のみ
・Reactは検索条件を送るだけ

つまり：

React
↓
GET /players?query
↓
Laravel
↓
DB検索
↓
結果返却

という構成。

───────────────────────────────────

■ 今日やった流れ

① getPlayers を query対応
② Players.tsx から queryを送る
③ state変更時に再取得する

───────────────────────────────────

■ ① player.ts 修正

対象：

src/api/player.ts

───────────────────────────────────

■ getPlayers 修正前

export const getPlayers = async () => {
  const res = await fetch(`${API_BASE_URL}/players`)
}

───────────────────────────────────

■ getPlayers 修正後

export const getPlayers = async (params?: {
  name?: string
  position?: "GK" | "DF" | "MF" | "FW"
  sort?: string
}) => {

  const query = new URLSearchParams()

  if(params?.name) query.append("name", params.name)

  if(params?.position) {
    query.append("position", params.position)
  }

  if(params?.sort) {
    query.append("sort", params.sort)
  }

  const url = query.toString()
    ? `${API_BASE_URL}/players?${query.toString()}`
    : `${API_BASE_URL}/players`

  const res = await fetch(url)

  if(!res.ok) {
    throw new Error("取得失敗")
  }

  const data = await res.json()

  return data.data
}

───────────────────────────────────

■ 学んだこと①：URLSearchParams

const query = new URLSearchParams()

は、

query文字列を作るためのもの。

────────────────

例：

query.append("name", "田中")

↓

name=田中

────────────────

複数あると：

name=田中&position=MF

になる。

───────────────────────────────────

■ 学んだこと②：query.toString()

query.toString()

は、

queryを文字列に変換する。

────────────────

例：

name=田中&position=MF

───────────────────────────────────

■ 学んだこと③：三項演算子

const url = query.toString()
  ? A
  : B

意味：

条件 ? YES : NO

────────────────

今回は：

queryがある？
↓
YESなら query付きURL
NOなら 普通URL

────────────────

YES：

/api/players?name=田中

NO：

/api/players

───────────────────────────────────

■ なぜ ? を分けたか

これでも動く：

/api/players?

Laravel的には普通に通る。

ただ、

queryがないなら：

/api/players

の方が自然。

実務でもよくやる。

───────────────────────────────────

■ ② Players.tsx 修正

対象：

src/pages/Players.tsx

───────────────────────────────────

■ 修正前

const fetchPlayers = async () => {
  const data = await getPlayers()
  setPlayers(data)
}

───────────────────────────────────

■ 修正後

const fetchPlayers = async () => {
  const data = await getPlayers({
    name: searchName || undefined,

    position:
      filterPosition === "ALL"
        ? undefined
        : filterPosition,

    sort:
      sortType === "none"
        ? undefined
        : sortType,
  })

  setPlayers(data)
}

───────────────────────────────────

■ 学んだこと④：空文字を送らない

name: searchName || undefined

────────────────

searchName が：

"田中"
↓
送る

""
↓
undefined

────────────────

つまり：

空なら queryを送らない。

───────────────────────────────────

■ 学んだこと⑤：ALLを送らない

フロント：

"ALL"

Laravel：

GK / DF / MF / FW しか知らない。

だから：

ALLなら undefined にする。

───────────────────────────────────

■ 学んだこと⑥：noneを送らない

sortType === "none"
  ? undefined
  : sortType

────────────────

並び替えなしなら：

sort を送らない。

───────────────────────────────────

■ ③ useEffect 修正

修正前：

}, [location])

────────────────

修正後：

}, [
  location,
  searchName,
  filterPosition,
  sortType
])

───────────────────────────────────

■ 学んだこと⑦：依存配列

依存配列に入ったstateが変わると：

useEffect が再実行される。

────────────────

つまり：

searchName変更
↓
useEffect再実行
↓
fetchPlayers()
↓
API再取得

───────────────────────────────────

■ 現在の完成構造

input/select変更
↓
searchName/filterPosition/sortType 更新
↓
useEffect再実行
↓
fetchPlayers()
↓
getPlayers({...})
↓
GET /players?query
↓
Laravel検索
↓
DB取得
↓
返却
↓
setPlayers(data)
↓
画面更新

───────────────────────────────────

■ 現在完成しているAPI

GET /players
POST /players
GET /players/:id
PUT /players/:id
DELETE /players/:id

───────────────────────────────────

■ 現在完成しているquery

GET /players?name=田中

GET /players?position=MF

GET /players?sort=name

GET /players?name=田中&position=MF&sort=name

───────────────────────────────────

■ 今日の本質

Reactは検索しない。

Reactは：

・状態管理
・入力
・表示

のみ。

検索処理そのものは：

Laravel側。

つまり：

「ロジックはバックエンド」

を実際にAPI接続で実現した。