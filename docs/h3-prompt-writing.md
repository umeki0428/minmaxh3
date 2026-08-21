# MiniMax H3 動画プロンプト手引き

H3 はテキスト・画像・動画・音声をまとめて理解し、4–15 秒・最大 2K・ネイティブステレオ音声の動画を出す。短い自由文でも動くが、学習データに沿ったフィールド構造のほうが安定する。

公式ソース:

- [MiniMax H3 Prompt Writing Skill](https://github.com/MiniMax-AI/MiniMax-H3/tree/main/skills/h3-prompt-writing)
- [Video Generation API](https://platform.minimax.io/docs/guides/video-generation)
- [H3 Feature Highlights](https://platform.minimax.io/docs/guides/video-prompt)

エージェントが実際に従う原文は次の2ファイル。

- [base-en.txt](../.cursor/skills/h3-prompt-writing/references/base-en.txt) — T2VA / I2VA / FL2VA / L2VA
- [ref-en.txt](../.cursor/skills/h3-prompt-writing/references/ref-en.txt) — Ref2VA

## モードの選び方

```text
参照ファイルは何か？
├─ なし → T2VA（テキストから映像+音を全部作る）
├─ 画像 1 枚を先頭フレームにする → I2VA
├─ 画像 2 枚を先頭と末尾にする → FL2VA
├─ 画像 1 枚を末尾フレームにする → L2VA
└─ キャラ顔・動き・声・スタイル・編集リズムなど「役割」として使う
     （画像≤9、動画≤3、音声≤3、合計≤12）→ Ref2VA
```

先頭/末尾フレームと役割参照は別物。同じ人物写真でも「0秒の画にする」なら I2VA、「顔と服だけ借りる」なら Ref2VA。

## Base モード（T2VA / I2VA / FL2VA / L2VA）

### 最終プロンプトの骨格

キーフレームがあるときだけ、1行目にアライメント指示を置き、空行を1行挟む。

```text
integrated_multimodal_description: [Shot 1] Live-action, cinematic, ...

overall_soundscape: ...

non_diegetic_music: ...
```

この3つのフィールド名は変えない。H3 はこのキーに沿って本文・環境音・観客用BGMを分けて読む。

### アライメント指示（公式定型文）

I2VA:

```text
For the target video, at 0.00 seconds into the target video, <Picture 1> (from [Shot 1]) is fully referenced.
```

FL2VA（`N` は最終ショット、`S.SS` は動画長・小数2桁）:

```text
How the reference pictures align with the target video — Picture 1 (from Shot 1) aligns with the 0.00-second mark of the target video; Picture 2 (from Shot N) aligns with the S.SS-second mark of the target video.
```

L2VA:

```text
How the reference pictures align with the target video — <Picture 1> (from [Shot N]) aligns with the S.SS-second mark of the target video.
```

### 本体の書き順

`[Shot 1]` の直後にスタイルと初期構図。以降は再生順だけ。各ショットで次を具体化する。

1. 誰がどこにどう見えるか
2. 空間・小道具・光
3. 今起きている動作と結果
4. カメラ（種類 + 必要な振幅/速度）
5. 聞こえる劇中音・台詞

単純なワンショットは 150–250 語、複雑なタイムラインは 350–450 語を目安にする。40語の短い指示はモデルに穴埋めを任せる。

### ショットとカメラ

- `[Shot 1]` に時刻は付けない
- 2本目以降は `[Shot 2] At 00:03.500, the camera cuts to...`
- 時刻は単調増加、動画長の内側
- カットは新しい情報のため。微妙な寄りの変化は Push / Zoom にする
- FL2VA は原則1ショット。カットを増やすと先頭-末尾の補間が壊れる

カメラは文末にラベルを並べず、動作として書く。

```text
The camera pushes in with small amplitude at slow speed toward the folded letter in her hands.
```

### 台詞と音の分担

| 置く場所 | 中身 |
| --- | --- |
| 本体 | 台詞、歌、ラジオ、電話、キャラに聞こえる音 |
| `overall_soundscape` | 雨、足音、衣擦れ、呼吸、交通など。1–4文 |
| `non_diegetic_music` | 観客専用BGM。楽器・テンポ・音量変化。ムード語禁止 |

台詞フォーマット:

```text
The young woman with a quiet, breathy voice (S1) says: <d>[Japanese] 次で降ります。</d>
```

- 発話する人だけ `(S1)`。無言キャラに ID を付けない
- `<d>` の中は言語タグと原文だけ。翻訳しない
- ナレーションは `says in an off-screen voiceover` のあと、口が閉じていると書く
- カット越えは `<scenetrans>`、終端切断は `<cutoff>`
- 画面内文字は `"営業中"` のように原文をダブルクォート

BGM が無い、または完全無音を明示されたときだけ `N/A`。

### キーフレーム別の中身

| モード | 本体で書くべきこと |
| --- | --- |
| I2VA | 画像の人物・構図・空間を固定してから、次の動作を前へ進める |
| FL2VA | 2枚の静止説明を繰り返さない。ポーズ・物・光がどう遷移するかを書く |
| L2VA | 末尾画像は最終ショット。そこへ着地する先行状態と経路を逆算する |

## ループ動画（同一画像の FL2VA）

ComfyUI の FL2VA ループは、1枚の画像を `first_frame` と `last_frame` の両方に渡す。プロンプトは通常の FL2VA と同じ定型だが、**末尾が先頭に戻る周期運動**だけを書く。実例は [examples/loop-fl2va-body-sway.txt](../examples/loop-fl2va-body-sway.txt)。ワークフローは [workflows/minimax_h3_loop_fl2va.json](../workflows/minimax_h3_loop_fl2va.json)。

使う動き: sway / bounce / breathe / pulse / drift。使わない動き: 歩き去る、服を脱ぐ、ドアが開く、カット、ズームジャンプ。

書き方:

1. アライメント1行の `S.SS` を Duration と同じ秒にする（ワークフロー初期値は `5.00`）
2. `[Shot 1]` のみ。カメラは `static shot`
3. 経路は **先頭のポーズ → Duration 内の整数回の周期（5秒なら5回など） → 差分が縮小 → Picture 2（= Picture 1）に着地**。1回だけの往復にすると動きが足りない。胸は二次動作で、体の1拍のあとに余韻の揺れを残す
4. `overall_soundscape` は音量一定。始まりも終わりもフェードしない（音の継ぎ目対策）
5. 湯気や粒子を足すなら密度を一定にし、最後に同じ見え方へ戻す

Duration を変えたら、プロンプト先頭の `5.00-second` も必ず書き換える。フレーム数は 24fps の 17k+5 グリッドにスナップするため、5秒指定でも実フレームは 124（約 5.17 秒）になることがある。アライメントは Duration ウィジェットの秒に合わせる。

## Ref2VA（フルリファレンス）

役割未指定の添付ファイルは無視されやすい。先にラベルを定義する。

```text
subject_definitions:
summary:
retention_analysis:
detailed_description:
overall_soundscape:
non_diegetic_music:
```

スタイル宣言は `[Shot 1]` の前に 1–2 文。本体フィールド名は `detailed_description`。

### ラベルの使い分け

- `<Subject N>`: 実際に画面へ出す再利用単位（人、犬、店内、衣装、歩き方）
- `<Picture N>`: その画像自体が先頭/末尾/絵コンテになるときだけ独立行
- `<Video N>`: 元動画の編集・継続・カット構造
- `<Audio N>`: 声色参照や音声コピー

例: 顔は画像、歩きは動画なら1人にまとめる。

```text
<Subject 1> is the woman whose appearance comes from <Picture 1> and whose walking motion comes from <Video 1>.
```

`summary` は `[reference generation]` のようなタスク種別で始める。カメラやリズムだけ借りる参照動画は `video editing` ではなく `reference generation`。

`retention_analysis` はラベル1行。外見定義だけなら、新しい背景で走らせても `fully_preserved`。未定義の動きが無いことを減点しない。

発話する参照人物は視覚ラベルと話者 ID を両方残す。

```text
<Subject 3> (S1) says: <d>[English] Hey! Watch your dog!</d>
```

## 公式の短い実例（T2VA）

```text
integrated_multimodal_description: [Shot 1] Live-action, cinematic, a medium-wide shot frames a baker opening the shutters of a small street bakery before sunrise. The camera pushes in with small amplitude at slow speed as the middle-aged baker with a calm, slightly raspy voice (S1) places a fresh loaf on the wooden counter and says: <d>[English] First batch of the morning.</d> [Shot 2] At 00:05.000, the camera cuts to a close-up of steam rising from the sliced bread while the baker's final words carry over from the previous shot.

overall_soundscape: Wooden shutters scrape open over a quiet street as trays clink softly inside the bakery. The doorbell rings once, followed by light footsteps and the crisp sound of bread being sliced.

non_diegetic_music: A soft acoustic-guitar pattern at a moderate tempo, joined by sparse upright-bass notes and a gentle fade at the end.
```

## よくある崩れ方

- フィールド名を日本語や別名にする
- `[Shot 1]` に時刻を付ける / 2本目以降の時刻が戻る
- カメラを `orbit, cinematic, beautiful` とラベルだけ並べる
- 台詞を `<d>` の外で英訳する
- 足音を `non_diegetic_music` に書く、BGM を `overall_soundscape` に書く
- FL2VA でカットを増やし、2枚の間の経路を書かない
- ループなのに片道の動作や、音のフェードイン/アウトを書く
- Ref2VA で添付の役割を定義せず、ラベルを途中から増やす
- 動画長 8 秒なのに 00:12.000 のカットを書く

## 作業チェック

1. モードは入力に対して正しいか
2. フィールド名と順序は公式どおりか
3. 総尺とカット時刻が一致しているか
4. 各ショットに構図・動作・カメラ・音があるか
5. 台詞・画面文字は原文のままか
6. 参照ラベルは定義と本文で同じ意味か
