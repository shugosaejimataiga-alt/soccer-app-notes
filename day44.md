

📘 Players強化 Day9 完全版まとめ
（ControllerへPolicy適用 + 認可実動作確認）

━━━━━━━━━━━━━━━━━━━
■ Day9 の目的
━━━━━━━━━━━━━━━━━━━

Day8 で作成した：

PlayerPolicy

を、

実際の Controller 処理へ適用する。

目的：

他チームのPlayerを：

・閲覧禁止
・更新禁止
・削除禁止

にすること。

つまり：

認可ルールを
実際のAPI処理へ接続

する日。

━━━━━━━━━━━━━━━━━━━
■ Day9 開始時点の状態
━━━━━━━━━━━━━━━━━━━

Day8時点では：

Policyルール自体は完成済み

だった。

しかし：

Controller が
まだ authorize() を使っていなかった

ため、

実際のAPIでは：

認可されていなかった

状態だった。

つまり：

ルールブックは存在する
↓
しかし実際の警備員は未配置

状態。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した超重要本質
━━━━━━━━━━━━━━━━━━━

Laravel の認可は：

Policyを書くだけ

では動かない。

必ず：

$this->authorize()

を Controller で呼ぶ必要がある。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した Laravel 認可の流れ
━━━━━━━━━━━━━━━━━━━

Controller
↓
authorize()
↓
Playerモデルを見る
↓
Laravel が対応Policyを自動特定
↓
PlayerPolicy 実行
↓
許可 or 拒否

━━━━━━━━━━━━━━━━━━━
■ なぜ自動で Policy が見つかるのか
━━━━━━━━━━━━━━━━━━━

以下を実行したため：

php artisan make:policy PlayerPolicy --model=Player

この：

--model=Player

によって Laravel が：

Playerモデル
↓
PlayerPolicy担当

と自動認識している。

つまり：

Model
↓
そのModel専用Policy

という関係。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した Model と Policy の役割分担
━━━━━━━━━━━━━━━━━━━

Model
=
データ操作担当

Policy
=
そのデータを
誰が触っていいか担当

つまり：

Playerモデル
↓
playersテーブル操作

PlayerPolicy
↓
そのPlayerを
操作してよいか判定

━━━━━━━━━━━━━━━━━━━
■ 今回理解した authorize() の本質
━━━━━━━━━━━━━━━━━━━

$this->authorize('view', $player);

は：

現在ログイン中Userは、
このPlayerを見ていい？

を Laravel に確認している。

Laravel は内部で：

PlayerPolicy::view()

を実行する。

━━━━━━━━━━━━━━━━━━━
■ 今回追加した authorize()
━━━━━━━━━━━━━━━━━━━

━━━━━━━━━
① show()
━━━━━━━━━

追加：

$this->authorize('view', $player);

意味：

このPlayerを見てよいか確認

━━━━━━━━━
② update()
━━━━━━━━━

追加：

$this->authorize('update', $player);

意味：

このPlayerを更新してよいか確認

━━━━━━━━━
③ destroy()
━━━━━━━━━

追加：

$this->authorize('delete', $player);

意味：

このPlayerを削除してよいか確認

━━━━━━━━━━━━━━━━━━━
■ 今回発生した重要エラー
━━━━━━━━━━━━━━━━━━━

発生：

500 Internal Server Error

原因調査のため：

tail -n 50 storage/logs/laravel.log

を実行。

━━━━━━━━━━━━━━━━━━━
■ tail -n 50 の意味
━━━━━━━━━━━━━━━━━━━

ファイル最後50行を見る

つまり：

tail -n 50 storage/logs/laravel.log

↓

Laravelエラーログの最新50行確認

━━━━━━━━━━━━━━━━━━━
■ 今回の500エラー原因
━━━━━━━━━━━━━━━━━━━

ログ：

Call to undefined method
PlayerController::authorize()

意味：

PlayerController が
authorize機能を持っていなかった

━━━━━━━━━━━━━━━━━━━
■ 解決方法
━━━━━━━━━━━━━━━━━━━

追加：

use Illuminate\Foundation\Auth\Access\AuthorizesRequests;

さらに：

use AuthorizesRequests;

を Controller 内へ追加。

━━━━━━━━━━━━━━━━━━━
■ この意味
━━━━━━━━━━━━━━━━━━━

Laravelの authorize機能 を
Controllerへ追加

した。

これにより：

$this->authorize()

が使用可能になった。

━━━━━━━━━━━━━━━━━━━
■ 今回理解した認証と認可の違い
━━━━━━━━━━━━━━━━━━━

━━━━━━━━━
認証 authentication
━━━━━━━━━

誰なのか確認

例：

auth:sanctum

━━━━━━━━━
認可 authorization
━━━━━━━━━

そのデータを
操作してよいか確認

例：

Policy
authorize()

━━━━━━━━━━━━━━━━━━━
■ 今回確認した PlayerPolicy
━━━━━━━━━━━━━━━━━━━

return $user->team_id === $player->team_id;

意味：

同じteam_idなら許可
違うteam_idなら拒否

━━━━━━━━━━━━━━━━━━━
■ 今回のPostman確認
━━━━━━━━━━━━━━━━━━━

━━━━━━━━━
① ログイン
━━━━━━━━━

POST /api/login

取得：

Bearer Token

━━━━━━━━━
② show確認
━━━━━━━━━

GET /api/players/26

結果：

403 Forbidden

成功。

━━━━━━━━━
③ update確認
━━━━━━━━━

PUT /api/players/26

結果：

403 Forbidden

成功。

━━━━━━━━━
④ destroy確認
━━━━━━━━━

DELETE /api/players/26

結果：

403 Forbidden

成功。

━━━━━━━━━━━━━━━━━━━
■ 今回確認できたこと
━━━━━━━━━━━━━━━━━━━

他チームPlayerを：

・閲覧できない
・編集できない
・削除できない

状態になった。

━━━━━━━━━━━━━━━━━━━
■ 現在の PlayerController.php の重要状態
━━━━━━━━━━━━━━━━━━━

追加済み：

use Illuminate\Foundation\Auth\Access\AuthorizesRequests;

さらに：

use AuthorizesRequests;

追加済み。

━━━━━━━━━━━━━━━━━━━
■ 現在の authorize 状態
━━━━━━━━━━━━━━━━━━━

show：

$this->authorize('view', $player);

update：

$this->authorize('update', $player);

destroy：

$this->authorize('delete', $player);

━━━━━━━━━━━━━━━━━━━
■ 現在の Laravel 認可構造
━━━━━━━━━━━━━━━━━━━

auth:sanctum
↓
ログイン確認（認証）

authorize()
↓
PlayerPolicy確認（認可）

team_id比較
↓
許可 or 拒否

━━━━━━━━━━━━━━━━━━━
■ Day9 完了時点の到達状態
━━━━━━━━━━━━━━━━━━━

PlayerPolicy が
実際のController処理へ適用された

さらに：

他チームPlayerアクセス禁止

が実際に動作確認済み。

━━━━━━━━━━━━━━━━━━━
■ 次回 Day10
━━━━━━━━━━━━━━━━━━━

次回は：
一覧取得へ team_id 制限追加
を行う。

現在の問題：
GET /players

では：
全チームの選手一覧
が見えている。

次回は：
ログインUserの team_id のみ取得
へ変更する。

つまり：
自チーム選手だけ一覧表示
を実装する。