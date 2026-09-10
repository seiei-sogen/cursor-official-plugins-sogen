# pstack の Codex ツール対応表

pstack の共有スキル文書には、Claude Code のツール名 `Skill`、`Agent`、`AskUserQuestion` を残している。Codex でもファイルは同じであり、これらのツール名の対応先だけが変わる。モデル実行の変換はここでは扱わない。親が管理する Claude/Codex/Grok の実行経路表と、プロバイダーを明示した記述子については、[`provider-dispatch.md`](provider-dispatch.md) を読む。

## ツール操作

| pstack / Claude の操作 | Codex での対応 |
|------------------------|------------------|
| ファイルを読む | `shell`（`cat`、`head`、`tail`） |
| ファイルを作成・編集・削除する | `apply_patch` |
| シェルコマンドを実行する | `shell` |
| ファイル内容を検索する / ファイルを探す | `shell`（`rg`、`grep`、`find`、`ls`） |
| URL を取得する | `shell` で `curl` / `wget` を使う |
| Web を検索する | `web_search` |
| スキルを呼び出す（`Skill` ツール、`/command`） | スキルはネイティブに読み込まれる。提示された指示に従う。 |
| フロントマターの `paths` で自動読み込みの範囲を指定する | Claude Code のみ。Codex では `pstack:typescript-best-practices` を名前で呼び出す。 |
| サブエージェントを起動する（`Agent` / `Task` ツール） | `spawn_agent` |
| 1 ターンで N 個のサブエージェントを並列起動する | 1 回の応答で `spawn_agent` を N 回呼ぶ |
| サブエージェントの結果を待つ | `wait_agent` |
| 終了したサブエージェントの枠を解放する | `close_agent` |
| タスクを追跡する（TODO リスト / `TodoWrite`） | `update_plan` |
| 人間に選択式の質問をする（`AskUserQuestion`） | 通常の文章で質問し、ユーザーの返答を待つ。Codex には構造化された選択式質問ツールがない。 |

サブエージェントの起動には `multi_agent` の有効化が必要である。`~/.codex/config.toml` に追加する。

```toml
[features]
multi_agent = true
```

無効なら、Codex のネイティブ作業枠は理由を明示した脱落として扱う。独立した外部作業枠は引き続き実行し、親はプロバイダー数が減ったことを記録する。パネルを単一モデルの逐次実行に置き換えてはならない。

## サブエージェントの方針

poteto-mode の「サブエージェント」節は、Claude 固有の既定値 `subagent_type: "poteto-agent"` と `run_in_background: true` を設定する。Codex では次のように対応する。

- `poteto-agent` というサブエージェント種別は存在しない。臨時のサブエージェントに poteto-mode の作法を適用するには、最初に `poteto-mode` スキルを全文読むよう指示した `spawn_agent` を起動する。
- `spawn_agent` は呼び出したターンと並行して動くため、`run_in_background: true` に対応する独立したフラグはない。起動したら作業を続ける。
- `comment-sicko` という種別も存在しない。Claude Code では **no-comments** スキルがそれを起動する。Codex では、最初に `agents/comment-sicko.md` を全文読むよう指示した `spawn_agent` を起動する。
- Claude Code はすべてのサブエージェントをこのマシン上で動かす。そのため **swarm** スキルの作業者と、並列展開する手順書 `orchestrate`、`autopilot-full`、`autopilot-stack` は、書き込み担当者を worktree で隔離する。Codex でも同じである。
- それ以外の方針は変えない。文脈を直接貼らずファイル参照を渡し、書き込みを行う各作業者には専用の worktree またはブランチを与え、すべての差分を自分でレビューする。

## モデルとプロバイダー

設定されたすべての項目を Codex モデルに置き換えない。`/setup-pstack` は `claude:fable@max`、`codex:gpt-5.6-sol@max`、`grok:grok-4.6@xhigh` のような、環境をまたいで使える記述子を書く。親が Codex の場合、ネイティブなのは `codex:*` だけである。Claude と Grok の記述子は、`provider-dispatch.md` の指定どおり外部ランチャーへ渡す。現在の既定パネルは、4 プロバイダーの最先端モデルの多様性を意図的に維持しており、古い GPT や Claude の代替モデルは含まない。

## pstack が参照する Claude 組み込みスキル

一部の適用条件では、pstack ではなく Claude Code に同梱されたスキルを指定している。これらは Codex に存在しないため、次の挙動で代替する。

| pstack が指定する Claude 組み込みスキル | Codex での対応 |
|---------------------------------|----------|
| `run`（CLI/TUI を操作し、変更が動くことを確かめる） | `shell` で自分自身がアプリを実行し、実際の出力を観察する。 |
| `verify`（UI を操作し、修正を確かめる） | 利用可能な自動化手段で UI を操作するか、ユーザーに具体的な手動確認を渡す。実物を観察せず完了を主張しない。 |
| `plugin-dev:skill-development`（Claude の SKILL.md 作成ガイド） | 利用中のプラットフォームのスキル作成ガイドに従う。`writing-skills` スキルがあれば使う。`name` と `description` のフロントマターと、情報を段階的に開示する構成を保つ。 |
| `loop`（定期的または自分のペースでの再呼び出し。`babysit` が使用） | Codex に `loop` スキルはない。自分で一定の間隔で手順を再実行するか、利用可能なら Codex のスケジュール済みタスクを使う。 |

## 同梱スクリプト

`skills/poteto-mode/scripts/` には、PR 監視用の `watch-pr`、ストア操作用 CLI の `orch`、`worktree-audit.sh`、`runner/pstack-runner` がある。通常の Bun と Bash なので Codex でも同じように動き、`shell` から呼び出せる。外部ランナーにはさらに、割り当てた `claude`、`codex`、`grok` の実行ファイルが認証済みであることが必要である。親が Codex の場合、Codex プロバイダーはネイティブの `spawn_agent` で扱うため、外部ランナーは拒否する。他のスクリプトには `bun`、`gh`、スタック作業では `gt`、`worktree-audit.sh` では `jq` と `rg` が必要である。`worktree-audit.sh` は `~/.claude/projects/` 内の Claude Code の会話記録を読む。別環境で実行するときは、その実行環境の会話記録ディレクトリを指定する。

## 指示ファイル

pstack のスキルが「自分の指示ファイル」と書く場合、Codex ではプロジェクトルートの `AGENTS.md` と、グローバルな `~/.codex/AGENTS.md` を指す。Claude Code では `CLAUDE.md` である。
