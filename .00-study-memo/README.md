# フォーク版 cursor plugins

## 本家との差分

本家は [cursor/plugins](https://github.com/cursor/plugins) の `main`。
2026-11-24 時点で本家の変更はすべて取り込み済み（`005d8f6` まで）。
このフォークにしかない変更は次の 4 つ。

| 場所 | 中身 | コミット |
| --- | --- | --- |
| `pstack/skills/poteto-mode/SKILL.md`<br>`harness/pstack/claude-code/skills/poteto-mode/SKILL.md` | `disable-model-invocation` を `true` から `false` に変更。`/poteto-mode` と打たなくても、モデルが自分の判断で poteto-mode を読み込めるようにした。本家のファイルを書き換えている差分はこれだけ。 | `a914bb1` |
| `.github/workflows/sync-upstream.yml` | 2 日に 1 回、06:17 JST に本家の `main` を取り込む。競合したら止まるので、そのときは手で取り込む。 | `6d3e6f3` |
| `.github/workflows/regenerate-pstack-harness.yml` | `pstack/` を変えて push すると、`harness/pstack/` を作り直して push する。Claude Code が入れるのは `harness/pstack/claude-code/` なので、`pstack/` だけ直しても Claude Code には届かないため。 | `2cc47f0`（PR #1） |
| `.00-study-memo/` | 勉強用のメモ置き場。`pstack/` を 2026-09-10 時点でコピーし（`9076d0c`）、そのうち `skills/poteto-mode/` 以下（SKILL.md、playbooks、references、scripts）を日本語に訳した（`71b4c1b`）。本家を取り込んでもこのコピーは更新されない。 | `9076d0c`, `71b4c1b` |

差分を調べ直すときは次のコマンドを使う。

```sh
git fetch https://github.com/cursor/plugins.git main
git log --oneline FETCH_HEAD..HEAD    # このフォークにしかないコミット
git diff --stat FETCH_HEAD HEAD       # 本家とのファイル差分
```
