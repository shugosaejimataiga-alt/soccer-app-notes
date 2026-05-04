

■ Day14後半：CREATE高速化＋DELETE完全化まとめ（完全版）

【① 今日やったこと】

・DELETE機能を完成（楽観更新＋エラー対策）
・CREATE機能を高速化（navigate＋state渡し）
・APIの役割整理（get / create / delete / getOne）
・フロントとバックエンドの責務の明確化
・遅さの原因と解決（再取得→直接反映）

---

【② DELETE（削除）の最終構成】

■ フロー

ボタン押下
↓
UIから即削除（setPlayers）
↓
DELETE /players/{id}
↓
成功 → そのまま
失敗 → 復元（reloadなど）

---

■ 実装（重要）

```tsx
const handleDeleteClick = async (e: React.MouseEvent, id: number) => {
  e.stopPropagation()

  try {
    onDelete(id) // 先にUIから削除

    await deletePlayer(id)

  } catch (error) {
    console.error("削除失敗", error)
    alert("削除に失敗しました")

    window.location.reload() // 復元
  }
}
```

---

■ ポイント

・削除は表示制御（既存データ）
・ズレは起きうるが影響が小さい
・実務では「楽観更新＋失敗時ロールバック」

---

【③ CREATE（作成）の最終構成】

■ フロー

作成ボタン押下
↓
POST /players
↓
newPlayer取得（サーバーの真実）
↓
navigate("/players", stateで渡す)
↓
Playersで受け取る
↓
setPlayersで追加（即反映）

---

■ PlayerCreate.tsx

```tsx
const handleCreate = async (
  name: string,
  position: "GK" | "DF" | "MF" | "FW"
) => {
  try {
    const newPlayer = await createPlayer({ name, position })

    navigate("/players", {
      state: { newPlayer }
    })

  } catch (error) {
    console.error("作成失敗", error)
    alert("作成に失敗しました")
  }
}
```

---

■ Players.tsx

```tsx
useEffect(() => {
  if (location.state?.newPlayer) {
    setPlayers(prev => [...prev, location.state.newPlayer])
    return
  }

  fetchPlayers()
}, [location])
```

---

■ ポイント

・createはデータ生成（バックエンドが真実）
・APIの戻り値を使う → 要件遵守
・再取得を省略 → 高速化
・navigateのstateは通信ではなく即時データ受け渡し

---

【④ APIの役割整理】

getPlayers
→ 一覧取得

getPlayer
→ 1件取得（詳細・編集用）

createPlayer
→ 作成＋作成データ返却

deletePlayer
→ 論理削除

---

■ 重要

create後は
→ getPlayer不要（すでにデータ持っている）

---

【⑤ 遅さの原因と解決】

■ 遅い原因

・毎回GET /playersしている
・サーバー通信が入る

---

■ 解決

DELETE
→ state直接更新

CREATE
→ newPlayerを直接追加

---

■ 本質

「再取得をやめる」

---

【⑥ データの真実の考え方】

■ 原則

データの真実はバックエンド

---

■ 今回の実装

CREATE
→ API結果のみ使用（OK）

DELETE
→ UI先行だが最終的に同期（OK）

---

■ NG

フロントでデータ生成
サーバーと同期しない

---

【⑦ useEffectの意味】

```tsx
useEffect(() => {
  if (location.state?.newPlayer) {
    setPlayers(prev => [...prev, location.state.newPlayer])
    return
  }

  fetchPlayers()
}, [location])
```

意味：

・create後 → newPlayerを追加
・通常アクセス → DBから取得

---

【⑧ setPlayersの意味】

```tsx
setPlayers(prev => [...prev, newPlayer])
```

prev
→ 現在のplayers配列

[...]
→ 配列展開

→ 新しい選手を末尾に追加


---

【最重要まとめ】

・DELETE → 即削除＋失敗時戻す
・CREATE → API結果を即反映
・遅さの原因 → 再取得
・解決 → state直接操作

👉 「再取得を減らす」が高速化の本質
