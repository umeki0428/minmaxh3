---
name: h3-prompt-writing
description: Write MiniMax H3 video generation prompts for T2VA, I2VA, FL2VA, L2VA, and Ref2VA. Use when rewriting requests into H3 prompt structures, composing integrated_multimodal_description, overall_soundscape, and non_diegetic_music, aligning keyframes, or defining reference labels for images, videos, and audio.
---

# MiniMax H3 動画プロンプト

H3 は映像と同期ステレオ音声を同時に生成する。プロンプトは自由文ではなく、公式フィールド名・ショット記法・参照ラベルを厳密に守る。本文は英語。台詞・歌詞・画面内テキストだけ原文言語を保持する。

詳細はモードに応じて必ず読む。

- テキスト / キーフレーム: [references/base-en.txt](references/base-en.txt)
- フルリファレンス Ref2VA: [references/ref-en.txt](references/ref-en.txt)

出典: [MiniMax-AI/MiniMax-H3 skills/h3-prompt-writing](https://github.com/MiniMax-AI/MiniMax-H3/tree/main/skills/h3-prompt-writing)

## 1. モードを決める

| モード | 入力 | 使うガイド |
| --- | --- | --- |
| T2VA | テキストのみ | `base-en.txt` |
| I2VA | 先頭フレーム画像 | `base-en.txt` |
| FL2VA | 先頭 + 末尾フレーム | `base-en.txt` |
| L2VA | 末尾フレーム画像 | `base-en.txt` |
| Ref2VA | 画像・動画・音声の役割参照（最大画像9・動画3・音声3、合計12） | `ref-en.txt` |

キーフレーム（先頭/末尾）は I2VA / FL2VA / L2VA。キャラ・動き・声・スタイルなどの役割参照は Ref2VA。混在したら Ref2VA。

動画長は 4–15 秒の整数。タイムラインとカット時刻をその長さに合わせる。プロンプト上限は 7000 文字。

## 2. Base モードの最終形

フィールド名はこの順で固定。リネーム・省略しない。

T2VA はコア3フィールドから始める。キーフレーム付きは1行目にアライメント指示、空行1行、それからコア3フィールド。

### I2VA

```text
For the target video, at 0.00 seconds into the target video, <Picture 1> (from [Shot 1]) is fully referenced.
```

### FL2VA

`N` は最終ショット番号。`S.SS` は動画長（小数2桁固定）。

```text
How the reference pictures align with the target video — Picture 1 (from Shot 1) aligns with the 0.00-second mark of the target video; Picture 2 (from Shot N) aligns with the S.SS-second mark of the target video.
```

### L2VA

```text
How the reference pictures align with the target video — <Picture 1> (from [Shot N]) aligns with the S.SS-second mark of the target video.
```

### コア3フィールド

```text
integrated_multimodal_description: [Shot 1] ...

overall_soundscape: ...

non_diegetic_music: ...
```

- `integrated_multimodal_description`: 映像・動作・ショット・話者・台詞・歌唱・劇中音を再生順に書く本体
- `overall_soundscape`: 環境音・物理音・非言語の人の音を 1–4 文。台詞・歌唱・劇中音楽は繰り返さない。完全無音指定時のみ `N/A`
- `non_diegetic_music`: キャラには聞こえず視聴者だけに聞こえる BGM を 1–3 文。楽器・テンポ・ダイナミクスを書く。ムード語は使わない。無いときは `N/A`

キーフレームの書き方:

- I2VA: 先頭フレーム固定 → 動作開始 → 連続展開 → 結果/反応
- FL2VA: 原則シングルショット。静止画の再説明ではなく、2枚をつなぐ動きの経路。先頭状態 → 観察可能な中間変化 → 差分の縮小 → 末尾状態
- 同一画像ループ: 同じ画像を `first_frame` と `last_frame` に渡す FL2VA。往復・周期運動だけ書く。細かい揺れは Duration 内の整数周期（5秒なら5回）にする。1回だけの往復は禁止。片道の動作、カット、フェードイン/アウトは禁止。アライメントの `S.SS` は Duration と同じ秒数（小数2桁）にする。音量は一定で両端を切らない
- L2VA: 末尾フレームは最終 `[Shot N]`。先行状態を推定 → 遷移経路 → 最終ショットで漸近 → 着地

## 3. ショット・カメラ・台詞

`[Shot 1]` にタイムスタンプは付けない。冒頭でスタイルと初期構図を宣言する。例: `Live-action`, `cinematic`, `2D-animated`, `3D CG`, `claymation`, `watercolor`, `vintage film`。キーフレーム時は参照画像からスタイルを決める。

2本目以降:

```text
[Shot 2] At 00:03.500, the camera cuts to...
```

カット時刻は単調増加し、動画長内に収める。通常カットは `the camera cuts to` / `the shot cuts to` / `the shot transitions to`。カットは被写体・空間・状態・視点・時間の新しい情報用。距離や角度の微小変化はカメラモーションにする。

カメラは「種類 + 必要なら振幅 + 必要なら速度」を自然な英文にする。ラベルの羅列はしない。

| 種類 | 意味 |
| --- | --- |
| Zoom In / Zoom Out | 位置固定で焦点距離 |
| Push In / Pull Out | カメラ自体が前進/後退 |
| Pan Left / Pan Right | 位置固定で水平振り |
| Truck Left / Truck Right | 水平移動 |
| Tilt Up / Tilt Down | 位置固定で垂直振り |
| Pedestal Up / Pedestal Down | カメラ全体の上下 |
| Arc Shot | 円弧 |
| Tracking Shot | 追従 |
| Static Shot | 静止 |
| Shake Slightly / Shake Strongly | 手ブレ |
| POV | 主観 |
| Roll Clockwise / Roll Counterclockwise | ロール |

振幅: `with small amplitude` / `with large amplitude`。速度: `at slow speed` / `at fast speed`。中程度は省略。

発話する人物だけ `(S1)` `(S2)`。同時発話は `(S1,S2)`。ショットをまたいでも同一人物は同じ ID。無発声キャラに ID は付けない。初出で年齢・性別・声質など識別情報を添える。

台詞は `<d>[Language] ...</d>`。中身は言語タグと原文のみ。翻訳・言い換え禁止。

```text
The young woman with a quiet, breathy voice (S1) says: <d>[English] I get off at the next station.</d>
```

ボイスオーバーは必ず `says in an off-screen voiceover`。`<d>` の直後に口が閉じたままであることを書く。

カットをまたぐ台詞は接続点の両方で `<scenetrans>` を使い、音声が継続することを明示する。動画終端で切れる台詞は `<cutoff>`。

画面内テキストは英語ダブルクォートで原文のまま。

```text
A red neon sign reading "営業中" glows above the doorway.
```

## 4. Ref2VA の最終形

6セクションをこの順で英語で書く。

```text
subject_definitions:
summary:
retention_analysis:
detailed_description:
overall_soundscape:
non_diegetic_music:
```

### ラベル

| ラベル | 用途 |
| --- | --- |
| `<Subject N>` | 再利用する可視コンテンツ（人物・物・場所・衣装・スタイル・動作） |
| `<Picture N>` | 具体的なフレーム/構図アンカー |
| `<Video N>` | 編集元・継続起点・時間構造 |
| `<Audio N>` | コピーまたは参照する音声 |

一度付けたラベルは全セクションで同じ意味。画像がキャラ定義だけなら独立の `<Picture N>` 行は作らず、`<Subject N>` 内で出典引用する。`<Video N>` と `<Audio N>` は独立採番。動画に音があっても自動では `<Audio N>` を作らない。

### summary のタスク種別

角括弧プレフィックスから始める。複数は ` + ` で結合。

| 種別 | 条件 |
| --- | --- |
| `keyframe completion` | 画像が先頭/キー/末尾など具体フレーム |
| `reference generation` | キャラ・シーン・スタイル・動作などの生成ガイド |
| `video editing` | 既存動画を直接改変 |
| `video continuation` | 既存動画から継続 |
| `audio reuse` | 音声信号を全部または一部コピー |
| `audio reference` | 信号はコピーせず声質・リズム等のみ |

### retention_analysis

可視: `fully_preserved` / `partially_preserved` / `attribute_transfer` / `weak_reference`

音声: `fully_copy` / `partially_copy` / `reference` / `weak_reference`

対象動画で追加した新しい動作や背景を、参照忠実度の低下として扱わない。定義された外見が保たれていれば `fully_preserved`。

### detailed_description

スタイルは `[Shot 1]` の前に 1–2 文。生成タスクは目安 350–500 語。台詞が多いときは語数より発話タイムラインの完全性を優先。参照ラベルは初出と役割が効く箇所に入れる。発話する Subject は `<Subject N> (Sx)`。

## 5. 出力ルール

- フィールド名・セクション順・ラベル・時刻記法をガイドどおりに保つ
- 各ショットで構図・被写体・環境・動作・カメラ・音・参照の効きどころを書く
- あらすじ要約、未解決ラベル、動画長と合わない時刻は禁止
- 抽象語（beautiful 等）より具体的な視覚・聴覚ディテール
- キャラに聞こえる音（ラジオ、劇中歌、足音の描写の同期）は本体側。BGM は `non_diegetic_music`
- 同一画像ループは往復・周期運動だけ書く。片道の動作、カット、音のフェードは継ぎ目になる
