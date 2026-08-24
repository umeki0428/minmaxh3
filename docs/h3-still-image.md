# MiniMax H3 で静止画を作る（構図参照 × キャラ差し替え）

H3 は動画モデルで、公式の画像生成ワークフローは無い。ここでは「最短の 5 フレームだけ生成して 1 枚を使う」流用で、**構図画像の人物をキャラに差し替えた下絵**を作る。用途は anytest（ControlNet）で描き直す前の構図ラフ。顔の再現度は求めない。

検証は途中（2026-08-24 時点）。結論が出ている項目と、未検証の項目を分けて書く。

## 仕組み

この会話で唯一、動作が確認できている参照構造は、動画の **`[video editing]`**（`<Video 1>` の人物を `<Picture 1>` のキャラに差し替え、それ以外は保つ）。静止画もこれに乗せる。

```
構図画像 ──→ RepeatImageBatch(5) ──→ ref_videos.ref_video_0  = <Video 1>
キャラ画像 ─────────────────────→ ref_images.ref_image_0  = <Picture 1>（任意）
プロンプト（公式 Ref2VA 6セクション、[video editing]）
MiniMaxH3ReferenceToVideo（length = 5）→ VAEDecode → SaveImage（5枚）
```

- 核ノードのみ（ComfyUI 本体 + 公式 R2V テンプレートの配線）。カスタムノード不要
- モデルはモーション転写と同じ：`ref2va_pruned_int8_convrot` / `qwen3vl_32b nvfp4` / video VAE / audio VAE（参照ノードが要求するだけ）
- 出力サイズは構図画像（または余白付きキャンバス）の比率を 768×1344 以内・32 の倍数に収めた値。Megapixels 1.0 で本番、0.4 で試し
- `ref_image_size = max`、Scheduler `beta`、Steps 25、Turbo LoRA なし

## ワークフロー

| ファイル | キャラ画像 | 出力比率 | 状態 |
| --- | --- | --- | --- |
| [workflows/minimax_h3_still_panel_edit_ref2va.json](../workflows/minimax_h3_still_panel_edit_ref2va.json) | あり | 構図画像と同じ | **動作確認済み（基準）**。生成後に anytest サイズへ余白を足す後処理（⑨）付き |
| [workflows/minimax_h3_still_panel_edit_textchar_ref2va.json](../workflows/minimax_h3_still_panel_edit_textchar_ref2va.json) | なし（文章のみ） | 832×1216（余白付き入力） | 検証中 |
| [workflows/minimax_h3_still_panel_edit_832x1216_ref2va.json](../workflows/minimax_h3_still_panel_edit_832x1216_ref2va.json) | あり | 832×1216（余白付き入力） | 検証中 |
| [workflows/minimax_h3_still_composition_ref2va.json](../workflows/minimax_h3_still_composition_ref2va.json) | あり（`<Picture 2>` に構図） | 構図画像 | **不成立**。[MiniMax H3 Image Studio](https://github.com/astropuzzo/ComfyUI-MiniMax-H3-Image-Studio) 版。構図画像を `<Picture 2>` で渡す方式では構図が効かなかった |

プロンプトのテンプレート：

- [examples/ref2va-still-panel-edit-template.txt](../examples/ref2va-still-panel-edit-template.txt) — キャラ画像あり。`[HAIR]` `[EYES]` `[OUTFIT]` を置換
- [examples/ref2va-still-panel-edit-textchar-template.txt](../examples/ref2va-still-panel-edit-textchar-template.txt) — キャラ画像なし。同じ置換

## 検証ログ

| 日付 | 構成 | 結果 |
| --- | --- | --- |
| 08-23 | Image Studio Reference Edit：キャラ=`<Picture 1>`、構図=`<Picture 2>`、fidelity 0.5 | 構図が効かずキャラ画像の丸写し |
| 08-23 | 同上で出力比率をキャラ画像に合わせる | 丸写し |
| 08-23 | 同上で役割を入れ替え（構図=`<Picture 1>`） | 丸写し |
| 08-23 | 同上で `optimize_for_still` off、公式形式プロンプト | 丸写し |
| 08-23 | **核ノード版**：構図=`<Video 1>`（5フレーム）、キャラ=`<Picture 1>`、`[video editing]`、キャラ記述なし | **構図は完全再現**、人物は元キャラのまま（差し替えなし） |
| 08-23 | 同上 + プロンプトにキャラの特徴（髪・目・輪・スーツ）を明記 | **構図維持・差し替え成功**。顔の再現度は低い |
| 08-23 | 同上 + 汎用プロンプト（キャラ記述なし）+ 構図画像を 832×1216 に余白付き配置 | キャラ画像の丸写しに戻る（2 項目同時変更のため原因未確定） |
| 08-23 | 基準構成に戻して再確認 | 構図維持・差し替え成功（基準 OK） |
| 08-23 | 基準構成でプロンプトだけ汎用版 | 差し替わらない → **キャラの見た目は文章が必要** |

## 確定したこと

1. 構図・ポーズは **`<Video 1>` + `[video editing]`** で確実に保持される。画像を `<Picture 2>` として渡しても効かない
2. キャラの見た目は **文章でほぼ決まる**。`<Picture 1>` だけでは差し替わらない。画像は色味・雰囲気の補助
3. Image Studio の `H3ReferenceEditPrepare` は「Picture 1 を保て」という定型文を自動で前置する（`_normalize_prompt`）。構図転写には向かない
4. 参照の「重み」を数値で決めるパラメータは核ノードに無い。効くのは順に：プロンプト ＞ 参照画像の枚数・解像度 ＞ `ref_image_size` ＞ 出力解像度 ＞ サンプリング
5. 公式にもコミュニティにも「キャラ参照＋構図再現」専用の仕組みは無い。動画の人物差し替えを静止画に転用して成立させている

## 未検証

- 構図画像を 832×1216 に余白付きで配置したとき、構図を保てるか（`textchar` / `832x1216` の 2 本で切り分け中）
- 「余白入力 × キャラ画像あり」が丸写しの原因か、汎用プロンプトが原因か
- キャラ画像の有無で色味・質感がどれだけ変わるか（`textchar` と `832x1216` の比較）
- 余白が白い帯のまま残るか、背景として埋まるか

## 運用ルール

- 変更は **1 回に 1 項目**。比較は同じ構図画像・同じ seed（fixed）で行う
- 構図画像：1 コマだけ切り出す。吹き出し・文字は消す。ページ全体を入れない
- キャラ画像：全身立ち絵はポーズごと写されやすい。丸写しが出たらバストアップに切り抜く
- 出力は 5 フレームすべて保存し、良い 1 枚を選ぶ。seed を変えて 3〜4 回
