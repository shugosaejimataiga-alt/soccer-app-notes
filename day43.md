

📘 Players強化 Day8 完全まとめ
（PlayerPolicy 作成 + 認可ロジック理解）

━━━━━━━━━━━━━━━━━━━
■ Day8 の目的
━━━━━━━━━━━━━━━━━━━

「ログインしているだけ」

ではなく、

「そのデータを操作してよいか」

まで判定できるようにする。

つまり：

認証(authentication)
↓
「あなたは誰？」

認可(authorization)
↓
「あなたはそれを操作していい？」

を実現する。

今回の目的：

他チームの選手を：

・閲覧できない
・更新できない
・削除できない

状態を作るための土台を作成する。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した最重要概念
━━━━━━━━━━━━━━━━━━━

auth:sanctum
=
ログイン確認

PlayerPolicy
=
そのデータを操作していいか確認

つまり：

認証(authentication)
≠
認可(authorization)

である。

━━━━━━━━━━━━━━━━━━━
■ 今回作成したもの
━━━━━━━━━━━━━━━━━━━

作成コマンド：

php artisan make:policy PlayerPolicy --model=Player

━━━━━━━━━━━━━━━━━━━
■ make:policy の意味
━━━━━━━━━━━━━━━━━━━

Laravel に：

「Policy機能を使います」

と伝える artisan コマンド。

Laravel は：

・Policiesフォルダ作成
・PlayerPolicy.php作成
・namespace設定
・use文追加
・Policy用メソッド生成

を自動で行う。

━━━━━━━━━━━━━━━━━━━
■ --model=Player の意味
━━━━━━━━━━━━━━━━━━━

このPolicyは：

Playerモデル用です

という意味。

これによりLaravelは：

use App\Models\Player;

を自動追加し、

public function view(User $user, Player $player)

のような：

Player型付きメソッド

を自動生成する。

━━━━━━━━━━━━━━━━━━━
■ 今回生成されたファイル
━━━━━━━━━━━━━━━━━━━

app/Policies/PlayerPolicy.php

━━━━━━━━━━━━━━━━━━━
■ Laravel の本質理解
━━━━━━━━━━━━━━━━━━━

Laravel は：

「フォルダ構造そのものが設計」

になっている。

例：

app/Models
→ Model置き場

app/Http/Controllers
→ Controller置き場

database/migrations
→ migration置き場

app/Policies
→ Policy置き場

artisan の make:○○ は：

Laravel設計ルールに沿った
正しい雛形を生成する仕組み。

━━━━━━━━━━━━━━━━━━━
■ PlayerPolicy.php の役割
━━━━━━━━━━━━━━━━━━━

Playerデータへの：

アクセス権限ルール

を書く専用クラス。

つまり：

誰が
どのPlayerを
操作していいか

を決める場所。

━━━━━━━━━━━━━━━━━━━
■ 今回生成されたメソッド
━━━━━━━━━━━━━━━━━━━

viewAny()
→ 一覧を見ていい？

view()
→ このPlayerを見ていい？

create()
→ 作成していい？

update()
→ 更新していい？

delete()
→ 削除していい？

restore()
→ 復元していい？

forceDelete()
→ 完全削除していい？

━━━━━━━━━━━━━━━━━━━
■ viewAny と view の違い
━━━━━━━━━━━━━━━━━━━

viewAny()
=
一覧権限

GET /players 用

まだ特定Playerが存在しないため：

Player $player

は渡されない。

━━━━━━━━━━━━━━━━━━━

view()
=
単体データ権限

GET /players/5 用

特定Playerを扱うため：

Player $player

が渡される。

━━━━━━━━━━━━━━━━━━━
■ Laravel が自動で渡してくれるもの
━━━━━━━━━━━━━━━━━━━

public function view(User $user, Player $player)

の：

$user
=
現在ログインしているUser

$player
=
現在操作対象のPlayer

Laravel が：

「誰が」
「どのPlayerを」
操作しようとしているか

を自動で渡してくれる。

━━━━━━━━━━━━━━━━━━━
■ 今回実装した核心ロジック
━━━━━━━━━━━━━━━━━━━

view()

update()

delete()

へ：

return $user->team_id === $player->team_id;

を追加。

━━━━━━━━━━━━━━━━━━━
■ この比較の意味
━━━━━━━━━━━━━━━━━━━

ログインUserの team_id
=
対象Playerの team_id

なら true。

つまり：

同じチームなら許可。

違うチームなら false。

━━━━━━━━━━━━━━━━━━━
■ 現在の PlayerPolicy.php
━━━━━━━━━━━━━━━━━━━

<?php

namespace App\Policies;

use App\Models\Player;
use App\Models\User;
use Illuminate\Auth\Access\Response;

class PlayerPolicy
{
    public function viewAny(User $user): bool
    {
        return false;
    }

    public function view(User $user, Player $player): bool
    {
        return $user->team_id === $player->team_id;
    }

    public function create(User $user): bool
    {
        return false;
    }

    public function update(User $user, Player $player): bool
    {
        return $user->team_id === $player->team_id;
    }

    public function delete(User $user, Player $player): bool
    {
        return $user->team_id === $player->team_id;
    }

    public function restore(User $user, Player $player): bool
    {
        return false;
    }

    public function forceDelete(User $user, Player $player): bool
    {
        return false;
    }
}

━━━━━━━━━━━━━━━━━━━
■ return false の意味
━━━━━━━━━━━━━━━━━━━

Laravel は最初：

全部拒否

の安全状態でPolicyを生成する。

つまり：

default deny

思想。

許可したいものだけを
明示的に true にする。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した超重要概念
━━━━━━━━━━━━━━━━━━━

現在やったのは：

認可ルール定義

だけ。

つまり：

Policyは作った。

しかし：

Controller側には
まだ適用していない。

現在：

ルールブックだけ存在

している状態。

━━━━━━━━━━━━━━━━━━━
■ 次回 Day9 でやること
━━━━━━━━━━━━━━━━━━━

Controller に：

$this->authorize(...)

を追加する。

すると：

Controller
↓
authorize()
↓
PlayerPolicy呼び出し
↓
team_id比較
↓
許可 or 拒否

が実際に動く。

━━━━━━━━━━━━━━━━━━━
■ 今回の到達状態
━━━━━━━━━━━━━━━━━━━

・PlayerPolicy作成完了
・認可(authorization)理解
・team_id比較ロジック作成完了
・view / update / delete 認可作成完了
・認証と認可の違い理解
・Controller未適用状態を理解

━━━━━━━━━━━━━━━━━━━
■ 今回の本質
━━━━━━━━━━━━━━━━━━━

認証(authentication)
=
本人確認

認可(authorization)
=
そのデータを操作していいか確認

Policy
=
データ単位の権限制御