# minmaxh3

MiniMax H3 の動画プロンプトを、公式のフィールド構造で書くための学習リポジトリです。

H3 はテキスト・画像・動画・音声をまとめて理解し、映像と同期音声を同時に出します。短い自由文でも生成できますが、安定させるにはモードごとの定型フィールド、ショット記法、参照ラベルを守ります。

## まず読むもの

1. [docs/h3-prompt-writing.md](docs/h3-prompt-writing.md) — 日本語の書き方
2. [`.cursor/skills/h3-prompt-writing/SKILL.md`](.cursor/skills/h3-prompt-writing/SKILL.md) — エージェント用の手順
3. [examples/loop-fl2va-body-sway.txt](examples/loop-fl2va-body-sway.txt) — 同一画像 FL2VA ループの実例
4. 公式原文
   - [base-en.txt](.cursor/skills/h3-prompt-writing/references/base-en.txt) — T2VA / I2VA / FL2VA / L2VA
   - [ref-en.txt](.cursor/skills/h3-prompt-writing/references/ref-en.txt) — Ref2VA

## モード

| モード | 入力 | プロンプトの核 |
| --- | --- | --- |
| T2VA | テキストのみ | コア3フィールド |
| I2VA | 先頭フレーム | アライメント1行 + コア3フィールド |
| FL2VA | 先頭と末尾 | アライメント1行 + コア3フィールド（原則1ショット） |
| FL2VA ループ | 同一画像を先頭と末尾 | 上に同じ。往復・周期運動だけ書き、末尾を先頭に戻す |
| L2VA | 末尾フレーム | アライメント1行 + コア3フィールド |
| Ref2VA | 役割参照（画像≤9、動画≤3、音声≤3、合計≤12） | 6セクション |

コア3フィールドは次の名前と順序で固定します。

```text
integrated_multimodal_description: [Shot 1] ...
overall_soundscape: ...
non_diegetic_music: ...
```

本文は英語。台詞・歌詞・画面内テキストだけ原文言語を残します。

## 絵コンテ資料

漫画1ページ（右上始まり）を 10秒の映像用絵コンテに展開した作例を同梱しています。尺 10秒 / 24fps / 16:9 / 7カット。

- 閲覧用：[`storyboard/index.html`](storyboard/index.html)
- テキスト版：[`storyboard/絵コンテ.md`](storyboard/絵コンテ.md)
- カット画：[`storyboard/cuts/`](storyboard/cuts/)

## ディレクトリ

| パス | 内容 |
| --- | --- |
| `docs/` | 日本語の解説ドキュメント |
| `examples/` | プロンプトの実例 |
| `workflows/` | ワークフロー JSON |
| `storyboard/` | 絵コンテ（HTML / Markdown / カット画） |
| `.cursor/` | エージェント用スキルとルール |

## 出典

公式スキルと参照ファイルは [MiniMax-AI/MiniMax-H3](https://github.com/MiniMax-AI/MiniMax-H3/tree/main/skills/h3-prompt-writing) の `skills/h3-prompt-writing` を学習用に配置しています。仕様の正は常に公式リポジトリと [Video Generation](https://platform.minimax.io/docs/guides/video-generation) です。
