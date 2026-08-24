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

### 動作転写（画像の人物に参照動画のダンスを踊らせる）

画像1枚を外見、動画1本を振付として渡す Ref2VA。実例は [examples/ref2va-dance-seg1-4s.txt](../examples/ref2va-dance-seg1-4s.txt)（4秒テスト）。ローカルは ComfyUI の公式テンプレート `video_minimax_h3_r2v.json` と `minimax_h3_ref2va_pruned_int8_convrot.safetensors` を使う。ラベルは接続順に `<Picture 1>` `<Video 1>` と振られる。

1. `<Subject 1>` を「デザインは `<Picture 1>`、ポーズ・動き・構図は `<Video 1>`」の1行で定義し、画像の独立 `<Picture N>` 行は作らない（先頭フレーム固定にするとタッチを変えられない）。画像のポーズ・構図・背景は使わないと明記し、開始ポーズは参照動画の先頭フレームに合わせる。画面に映らない要素（靴など）は書かない
2. タッチを参照動画に寄せるなら、動画の画風を `<Subject 2>` として線・塗り・ハイライトの有無まで具体的に定義し、`retention_analysis` で `<Subject 1>` を `partially_preserved`（デザイン維持・描画変更）、`<Subject 2>` を `attribute_transfer` にする
3. `<Video 1>` は振付とタイミングの出典として独立行にし、衣装・背景・画面内文字・落書きは使わないと明記する
4. 参照動画は目的の振りだけに切り出す（キャラ不在の導入や衣装切替を含めない）。周期運動なら先頭を末尾に継ぎ足して動画長ちょうどにできる
5. ComfyUI の R2V は参照動画の音声を自動で使う。動きだけ借りるなら `-an` で音を外し、BGM は `non_diegetic_music` で指定する。元曲を乗せるなら音を残し `<Audio 1>` を `fully_copy` で定義する
6. 動きは 0.5 秒刻みでは見誤る（拍手の往復が「手を組む」に見えた）。5fps 以上で見て、周期・片側あたりの秒数・左右交互を数えて書く
7. 表情を参照と変えるとき（目を開ける・口を閉じる）は「固定で、〜しない」と否定形まで書く
8. **ローカル R2V の参照動画は「動きの制御信号」ではなく文脈参照**（ComfyUI `nodes_minimax_h3.py`）。骨格抽出やフレーム単位の条件付けはなく、Kling Motion Control / Wan-Animate のような主役差し替え機能ではない。テキストエンコーダは参照動画を 2fps でしか見ない（0.2 秒周期の動きは言語側に見えない）。参照動画は生成フレーム数（17k+5）で切り詰められるので、クリップ長を生成フレーム数に合わせる（Duration 4 → 107 フレーム）。参照重視なら `ref_image_size: max`、Scheduler `beta`/`normal`、Steps 25–30。速い動きは参照を半速にして生成し、出力を 2 倍速にする手もある
9. **ただし「動画の人物をキャラ画像に差し替える」用途は同じ pruned int8 重みで実績がある**（実写ダンサー、公式テンプレート、画像1枚＋動画1本、LoRA なし）。コミュニティの成功例の型: キャラ画像は正面・背面・顔アップを1枚にまとめたキャラシート、プロンプトは `[video editing]` で「<Video 1> の人物を <Picture 1> のキャラに置き換え、それ以外はすべてそのまま」、`<Video 1>` は `partially_preserved`（人物の同一性だけ捨てる）、`<Subject 1>` は `fully_preserved`。編集は1回に1つ（背景・表情・画風の変更を同時に頼まない）。megapixels はテンプレート既定 0.4 ではなく 1.0 前後。結果は確率的で「同じプロンプトでも当たり外れがある」と報告されている。2D 限定アニメ素材・0.1 秒保持の高速動作での成功報告は見つかっていない。最小編集版は [examples/ref2va-dance-seg1-4s-swaponly.txt](../examples/ref2va-dance-seg1-4s-swaponly.txt)
10. **静止画（構図転写）**も同じ構造で作れる。構図画像を 5 フレームの静止動画にして `<Video 1>` に渡し、`[video editing]` で人物だけ差し替える。キャラの見た目は `<Picture 1>` だけでは変わらず、文章の明記が必要。詳細と検証ログは [h3-still-image.md](h3-still-image.md)
11. 参照動画が 8K の横長キャンバスに縦動画を置いたような書き出しのときは、`cropdetect` で内容領域を切り出し、1080 幅程度・24fps・音声なしに変換してから入れる（8K×99 フレームをそのまま LoadVideo に入れるとメモリを使い切る）。フレーム数が 17k+5 に合わないときは、末尾フレームを `tpad=stop_mode=clone` でホールドして切り上げ、出力を繋ぐときに切る
12. 動きを**完全トレース**したいなら `reference generation` ではなく `[video editing]` にする。`summary` を `The target video is an edited version of <Video 1>.` で始め、`<Video 1>` を「編集元」として `fully_preserved`、変える点（人物デザイン・背景・表情）だけを差分で書く。動きを拍ごとに再記述すると、その文が参照動画より強く効いて別の動きになる
13. **キャラ画像は、その動画で再現したい表情まで含めたキャラクターシート**にする（正面・側面・複数アングル、中立姿勢、白背景）。逆に「参照動画の構図に寄せて1枚だけポーズを作った画像」は、そのポーズが実際の開始フレームとして出やすく、参照動画の1コマ目とずれる。大きさも画像側に引っ張られ、参照動画上の女の子と一致しなくなることがある
14. 参照動画に人物以外の固有の特徴（尻尾・眼鏡・アクセサリ・体型など）があると、キャラ画像にそれが無くても**新キャラに乗り移ることがある**。モーション転写では、参照動画から要らない要素を `<Subject 1>` の定義に**否定形で個別に**書く（例: `she has no tail`, `she does not wear glasses`）。「保持する要素」を書くだけでは防げない
15. 表情も同様に、参照動画側の表情（照れ・赤面・特定の口の形など）を使いたくない場合は、`<Video 1>` の画風説明に含めた「頬の赤み」のような**画風レベルの記述**が新キャラにも及ぶことがある。個別の区間だけ変えたいときは、その区間のプロンプトでだけ画風の該当箇所を外し、`<Subject 1>` 側にも否定形（`no blush`, `no bashfulness` 等）を重ねて書く
16. **参照クリップが短すぎると、開始・終了の両方でキャラ画像自身の静止ポーズに戻ってしまう**ことがある。ComfyUI の `length` 入力のツールチップに「学習された長さの範囲はおよそ124〜362フレーム（24fpsで約5.2〜15秒）、それより短いのは未検証」とある。3〜4秒（90〜107フレーム）の短い区間ではこの下限を割り込み、動きへの追従が弱くなる。周期運動なら**同じクリップ内の動きをもう一周期ぶん継ぎ足して124フレーム以上に延ばす**と改善する（`ffmpeg` で `trim` + `concat` を使い、継ぎ目が自然かコマ送りで確認する）。あわせてプロンプトの `subject_definitions`・`summary`・`detailed_description` に「1コマ目からすでに動きの途中で、キャラ画像の静止ポーズは冒頭にも末尾にも一切出ない」と否定形で明記する（実例: [examples/ref2va-dance-seg1-4s.txt](../examples/ref2va-dance-seg1-4s.txt)）

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
- キャラ画像を参照動画の構図に合わせて1枚だけ作る（開始フレームと大きさが画像に引っ張られる）
- 参照動画固有の要素（尻尾・眼鏡・赤面・効果線・キラキラ等の落書き）を「保持する」と書くだけで、新キャラ側での否定を書かない（そのまま新キャラに乗り移る）
- 参照クリップが124フレーム（24fpsで約5.2秒）未満で、開始・終了フレームがキャラ画像自身の静止ポーズに戻ってしまう

## 作業チェック

1. モードは入力に対して正しいか
2. フィールド名と順序は公式どおりか
3. 総尺とカット時刻が一致しているか
4. 各ショットに構図・動作・カメラ・音があるか
5. 台詞・画面文字は原文のままか
6. 参照ラベルは定義と本文で同じ意味か
