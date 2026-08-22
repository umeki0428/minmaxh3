# minmaxh3

ストーリーからショートアニメまでを AI で通すための実験リポジトリ。あわせて MiniMax H3 の動画プロンプトの書き方を蓄積する。

## 目的

1. **H3 プロンプトの学習・蓄積** — MiniMax H3 の動画プロンプトを公式のフィールド構造で書けるようにし、書き方と実例を残す
2. **動画生成の API 自動化（実験）** — ストーリーを作り、そこから先の動画生成を API で回す
3. **ショートアニメ制作（実験）** — 1本の短編を最後まで仕上げる

2・3 は進行中の実験。ここにあるのは作品そのものではなく、各工程で固まった手順と書式。

## 工程

```
ストーリー  →  絵コンテ  →  動画生成  →  編集
              完全AI
              人間はFBのみ
```

| 工程 | やり方 | 状態 |
| --- | --- | --- |
| ストーリー | 元になる話を用意する | 未着手 |
| 絵コンテ | AI が全部作る。構成もカット割りも画も AI。人間はフィードバックを返すだけ | 一度通して検証済み。書式は [docs/絵コンテフォーマット.md](docs/絵コンテフォーマット.md) |
| 動画生成 | 絵コンテのカットを H3 で動画にする | 未着手。プロンプトの型（`docs/` `examples/`）は用意済み |
| 編集 | カットをつないで1本にする | 未着手 |

検証用に作った第1作（漫画1ページを 10秒 / 7カットに展開した絵コンテ）は、書式だけを [docs/絵コンテフォーマット.md](docs/絵コンテフォーマット.md) に残して削除した。生成した動画・音声はリポジトリに入れない（`.gitignore`）。

## H3 プロンプトの書き方

H3 はテキスト・画像・動画・音声をまとめて理解し、映像と同期音声を同時に出す。短い自由文でも生成できるが、安定させるにはモードごとの定型フィールド、ショット記法、参照ラベルを守る。

1. [docs/h3-prompt-writing.md](docs/h3-prompt-writing.md) — 日本語の書き方
2. [`.cursor/skills/h3-prompt-writing/SKILL.md`](.cursor/skills/h3-prompt-writing/SKILL.md) — エージェント用の手順
3. [examples/loop-fl2va-body-sway.txt](examples/loop-fl2va-body-sway.txt) — 同一画像 FL2VA ループの実例
4. 公式原文
   - [base-en.txt](.cursor/skills/h3-prompt-writing/references/base-en.txt) — T2VA / I2VA / FL2VA / L2VA
   - [ref-en.txt](.cursor/skills/h3-prompt-writing/references/ref-en.txt) — Ref2VA

| モード | 入力 | プロンプトの核 |
| --- | --- | --- |
| T2VA | テキストのみ | コア3フィールド |
| I2VA | 先頭フレーム | アライメント1行 + コア3フィールド |
| FL2VA | 先頭と末尾 | アライメント1行 + コア3フィールド（原則1ショット） |
| FL2VA ループ | 同一画像を先頭と末尾 | 上に同じ。往復・周期運動だけ書き、末尾を先頭に戻す |
| L2VA | 末尾フレーム | アライメント1行 + コア3フィールド |
| Ref2VA | 役割参照（画像≤9、動画≤3、音声≤3、合計≤12） | 6セクション |

コア3フィールドは次の名前と順序で固定する。

```text
integrated_multimodal_description: [Shot 1] ...
overall_soundscape: ...
non_diegetic_music: ...
```

本文は英語。台詞・歌詞・画面内テキストだけ原文言語を残す。

## ディレクトリ

| パス | 内容 |
| --- | --- |
| `docs/` | 絵コンテの書式と、H3 プロンプトの日本語解説 |
| `examples/` | 実際に使ったプロンプト |
| `workflows/` | ComfyUI 用ワークフロー JSON（FL2VA ループのローカル生成） |
| `.cursor/` | エージェント用スキルとルール |

## 出典

公式スキルと参照ファイルは [MiniMax-AI/MiniMax-H3](https://github.com/MiniMax-AI/MiniMax-H3/tree/main/skills/h3-prompt-writing) の `skills/h3-prompt-writing` を学習用に配置している。仕様の正は常に公式リポジトリと [Video Generation](https://platform.minimax.io/docs/guides/video-generation)。
