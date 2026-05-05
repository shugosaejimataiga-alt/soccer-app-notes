

■ Day14（UPDATE API接続 + 差分更新）完全まとめ

【① 今日の到達点】

・PlayerEdit画面で既存データ取得完成
・PUT /players/{id} API接続完成
・UPDATE保存処理完成
・保存中UI完成
・UPDATE差分更新完成
・一覧再取得なしで即反映完成

---

【② 今日の最重要テーマ】

UPDATE = 「置き換え」

---

【③ UPDATE全体フロー】

編集画面表示
↓
useEffect
↓
getPlayer(id)
↓
inputに既存データ表示
↓
編集
↓
保存クリック
↓
PUT /players/{id}
↓
updatedPlayer返却
↓
navigate("/players", state)
↓
Players.tsxで受け取る
↓
該当1件だけ置き換える

---

【④ PlayerEdit.tsx の役割】

■ useEffect

編集画面表示時に既存データ取得

■ 流れ

PlayerEdit表示
↓
useEffect実行
↓
getPlayer(id)
↓
setName
setPosition
↓
フォームへ反映

■ 理由

編集画面は既存データを表示する必要があるため。

---

【⑤ idガードの理解】

if (!id) return

■ 理由

URLはユーザーが自由に壊せる

例：
/players/edit
/players/abc/edit

だから
idが存在する保証がない

■ 本質

入力は信用しない

これはWeb開発の基本思想。

---

【⑥ updatePlayer の理解】

export const updatePlayer = async (
  id: number,
  data: {
    name: string
    position: "GK" | "DF" | "MF" | "FW"
  }
) => {
  const res = await fetch(`${API_BASE_URL}/players/${id}`, {
    method: "PUT",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      team_id: 1,
      name: data.name,
      position: data.position,
    }),
  })

  if (!res.ok) {
    throw new Error("更新失敗")
  }

  const json = await res.json()

  return json.data
}

---

【⑦ JSON.stringify の理解】

オブジェクトをJSON文字列へ変換

■ 理由

fetchのbodyは文字列しか送れない

■ headersとの関係

"Content-Type": "application/json"

↓

JSON形式ですよ
をサーバーへ伝える。

---

【⑧ 保存中UI（isSaving）】

■ state

const [isSaving, setIsSaving] = useState(false)

■ 役割

保存中かどうか管理

---

【⑨ handleUpdate の流れ】

const handleUpdate = async () => {
  if (!id) return

  setIsSaving(true)

  try {
    const updatedPlayer = await updatePlayer(Number(id), {
      name,
      position
    })

    navigate("/players/", {
      state:{updatedPlayer}
    })
  } catch (e) {
    alert("更新失敗")
  } finally {
    setIsSaving(false)
  }
}

---

【⑩ try / catch / finally の理解】

■ try

成功するか試す

■ catch

失敗時のみ実行

■ finally

成功でも失敗でも必ず実行

■ 今回

setIsSaving(false)

↓

保存終了処理

---

【⑪ disabled の理解】

disabled={isSaving}

■ 役割

保存中はボタン押せなくする

■ 理由

連打防止
PUT多重送信防止

---

【⑫ 三項演算子の理解】

{isSaving ? "保存中" : "保存"}

■ 意味

isSaving true → 保存中
false → 保存

---

【⑬ Players.tsx UPDATE差分更新】

if (location.state?.updatedPlayer) {
  setPlayers(prev =>
    prev.map(p =>
      p.id === location.state.updatedPlayer.id
        ? location.state.updatedPlayer
        : p
    )
  )
  return
}

---

【⑭ map の理解】

配列の1人ずつを見る

■ p

選手1人

■ 条件

p.id === updatedPlayer.id

↓

更新対象か？

■ 三項演算子

同じid → updatedPlayerへ置換
違う → そのまま

---

【⑮ UPDATE差分更新の本質】

一覧全件GETし直さない

■ やること

変更された1件だけ置き換える

---

【⑯ CREATE / DELETE / UPDATE の違い】

■ CREATE

POST成功
↓
newPlayer受け取る
↓
追加

■ DELETE

先にUI削除
↓
DELETE
↓
失敗時戻す

■ UPDATE

PUT成功
↓
updatedPlayer受け取る
↓
置き換える

---

【⑰ 差分更新と楽観更新の違い】

■ 差分更新

変わった部分だけ更新

■ 楽観更新

サーバー成功前にUI変更

---

【⑱ 今回の分類】

CREATE → 差分更新（安全）
DELETE → 楽観更新 + 差分更新
UPDATE → サーバー結果反映 + 差分更新

---

【⑲ なぜUPDATEは楽観更新しないか】

UPDATEは、

・バリデーション
・不正値
・サーバー整形

などがある。

だから

保存失敗なのに成功表示

が危険。

---

【⑳ UX改善の理解】

通信速度は変わっていない

■ 改善しているのは

ユーザー体感

■ 方法

保存中表示

---

【㉑ React Queryの理解】

サーバーデータ管理ライブラリ

■ 解決するもの

fetch
loading
cache
更新
再取得

■ 今のあなた

React Queryが内部でやる事を手動実装している

