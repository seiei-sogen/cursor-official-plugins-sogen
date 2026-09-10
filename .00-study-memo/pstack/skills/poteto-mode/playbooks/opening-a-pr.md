### PR の作成

他のすべての手順書の最後に呼び出す。

**worktree。** main から分岐した Git worktree で作業する。サブエージェントはそれを継承する。同じブランチで複数の `Agent` を呼ぶ場合も、それぞれ専用の worktree を与える。複数 worktree 間で 1 つのブランチを再利用するには、「出荷」の手順 1 で `<head-url>` を解決、検証し、`head_url` に取得する。そのうえで利用の切り替え時に `git fetch -- "$head_url" "refs/heads/$branch" && git reset --hard FETCH_HEAD` を実行する。無関係な作業でブランチが汚れている場合は、その作業をパッチに退避し、新しい worktree を作って適用する。worktree がもつれている場合は main からリセットし、最小限の作業をやり直す。

**コミット。** こまめにコミットする。PR を開く前に rebase し、小さく順序の整ったコミットにする。各コミットは将来の PR と考え、単独で取り込め、変更の流れを伝える順序にする。直前のコミットに属する修正なら amend し、分離できるなら新しいコミットにする。

**PR。** コミット前に差分へ `/deslop` を実行する。レビュー前に `/no-comments` を実行する。すべての PR タイトル、PR の説明、コミット本文を `/technical-writing` で書き、その後 `/unslop` を適用する。technical-writing は Diátaxis 以外のすべての層を適用する。各動作には 1 つの語を使い、冠詞を落とさず、普通の動詞で書ける場合は `-ing` を避ける。

**タイトル。** Conventional Commits の `type(scope): subject` 形式を使う。type は `feat`、`fix`、`docs`、`refactor`、`test`、`chore`、`perf` のいずれかとする。scope は `pstack`、`poteto-mode` など変更した領域にする。subject は短い命令形にする。変更を担う実際のシンボルがあれば名前を示す。例は `fix(pstack): retarget opening-a-pr babysit trigger`。末尾にピリオドを付けない。

**説明。** PR 本文は要点を伝える文書であり、実験ノートではない。差分を持つレビュー担当者が、変更の理由、対象外の範囲、動くことをどう実証したかを理解できるようにする。squash コミットの本文には PR 本文を使う。

次の節をこの順序で使う。書くことがない節は削る。

- `## Why`。意図と方式を 1〜2 個の短い段落で示す。SHA や rebase の系譜を並べない。「main をベースにした」という前置きも付けない。
- `## Scope`。実際のシンボルとパスを箇条書きにする。名前変更や参照先変更では変更前後の両方を示す。範囲の境界が重要な場合だけ、含むものと含まないものを書く。ファイルごとの長い説明にしない。
- `## Tradeoffs`。レビュー担当者がそうでなければ質問しそうな、不採用の代案だけを示す。実質的な選択がなければ省略する。
- `## Blast Radius`。誰や何に影響し、なぜ安全または危険かを 1〜3 文で示す。修正せず main のチェックが失敗し続けた場合の継続的な費用を述べる。
- `## Verification`。実際に試した各実行経路と結果を示す。性能変更なら、主要な数値を 1 つ、単位付きの `before → after` 形式で報告する。残りの証拠は arena または swarm のディレクトリへリンクする。標本数の方法論、swarm の各作業枠の羅列、指標の表は含めない。

これらの節の後に、主張を実証する動画やスクリーンショットを添付する。完全な SHA、swarm や arena の作業枠の羅列、検証道具の修正に関する長文、ファイルごとのチェックリスト、「CLEAN」判定を貼らない。その詳細はリンク先の成果物に置く。`## Summary` や `## Test plan` の定型文は使わない。コミット本文で件名を繰り返さない。

**ホスティングサービス。** 最初の PR 操作の前に使用するサービスを解決し、作成、編集、表示、監視、マージで同じ選択を保つ。既定は GitHub CLI の `gh` である。`command -v origin` が成功し、Origin がリポジトリを解決できれば `origin pr ...` を優先する。Origin がないかリポジトリを解決できなければ、`gh` を使い、代替したことを記録する。意図した PR のベースリポジトリを正規の `<base-repo>` として記録し、使用中のサービスで検証する。チェックアウトの既定リモートから推測しない。シェル変数へ取得し、すべての `gh pr` コマンドに `--repo "$base_repo"` を渡す。head のリポジトリがフォークなら、その実体を検証し、所有者とリポジトリ名を `<fork-owner>` と `<head-name>` として記録する。Graphite の `gt` を必須にしない。

**規模とスタック。** 大きな PR 1 件より、範囲を絞った PR 5 件を優先する。各子ブランチを親の正確な先端へ rebase し、下から上の順序を固定する。head とベースが同じリポジトリなら、ベースブランチの連鎖を作る。最下段の PR は trunk を対象とし、各子 PR は親ブランチを対象とする。同一リポジトリの子は、解決したサービスに応じて `origin pr create --status open --base "$parent_branch"` または `gh pr create --base "$parent_branch" --repo "$base_repo"` で作る。head がフォークなら、すべての PR はベースリポジトリの trunk を対象とし、積み重ねたローカルブランチは親子関係を保つ。フォークの PR は、解決した Origin のコマンドで作成する。GitHub では承認済みの PR タイトルと本文を `<title>` と `<body>` に取得し、`gh api --method POST "repos/$base_repo/pulls" -f "title=$title" -f "body=$body" -f "head=$fork_owner:$branch" -f "head_repo=$head_name" -f "base=$trunk" --jq .html_url` を実行する。準備状態の規則がドラフトを要求する場合だけ `-F draft=true` を追加する。フォークにしかない親ブランチを PR のベースにはできない。既存の子を rebase、force-push、またはベース変更する前に、その子とすべての子孫へ、「出荷」の手順 4 のマージ予約解除と確認の規則を適用する。同一リポジトリの子のベース変更には `origin pr edit "$pr" --base "$parent_branch"` または `gh pr edit "$pr" --base "$parent_branch" --repo "$base_repo"` を使う。フォークの子には、解決した Origin コマンドまたは `gh pr edit "$pr" --base "$trunk" --repo "$base_repo"` を使う。trunk から分岐するのは独立した作業だけにする。本格的なスタック作業の前に trunk へ rebase する。

**準備状態。** 既定では、各 PR をレビュー可能な状態で開く。Origin では `--status open` を渡す。GitHub CLI の作成コマンドでは `--draft` を省き、`--repo "$base_repo"` を渡す。上記の GitHub フォーク API では `draft` フィールドを省く。リポジトリの指示が、指定された証拠がそろうまでドラフトを要求する場合は、早期の PR をドラフトに保ち、証拠を記録してからレビュー可能にする。GitHub では CLI の作成コマンドに `--draft`、またはフォーク API に `-F draft=true` を渡し、後で `gh pr ready "$pr" --repo "$base_repo"` を実行する。Origin 使用時は、文書化されたドラフトとレビュー可能状態の操作を使う。ドラフト規則がないのにツールがドラフトで開いた場合は、サービスに応じて `origin pr ready "$pr"` または `gh pr ready "$pr" --repo "$base_repo"` を実行する。PR の状態に言及する前に、`origin pr view "$pr"` または `gh pr view "$pr" --repo "$base_repo"` を実行する。

**PR の見守り。** PR を開いただけでは見守りを始めない。URL を示し、実装を続ける。まずフェーズやスタックを終える。スタック全体がそろった後、ユーザーが求めた場合だけ、`babysit.md` に従って別の見守り作業を行う。新しい PR ごとに見守ると、実装が止まり、後の変更で再起動されるコミットにチェックを費やす。フィードバックが意図から外れたら異議を示す。

PR を開くサブエージェントは `interrogate`、`/deslop`、`/no-comments` を実行する。URL を返し、見守りは行わない。親へ戻る。
