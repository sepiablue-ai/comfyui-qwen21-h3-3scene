# Qwen Image 2.1 → MiniMax H3：3シーン動画ワークフロー

[日本語](README.md) | [English](README_en.md)

三面図を1枚選び、任意のAIチャットが作った7項目のJSONを1つのテキスト欄に貼ると、4枚のキーフレームと約3秒×3シーンの動画を生成・結合するComfyUIワークフローです。アニメと実写は**別々に1回ずつ**実行します。

## 同梱ファイル

- [`qwen_h3_3scene_workflow.json`](qwen_h3_3scene_workflow.json)：ComfyUIに読み込むGUIワークフロー
- [`prompt_guide_ja.md`](prompt_guide_ja.md)：AIに入力用JSONを作らせる指示文（日本語）
- [`prompt_guide_en.md`](prompt_guide_en.md)：同じ指示文（English）
- [`prompt_guide_zh-CN.md`](prompt_guide_zh-CN.md)：同じ指示文（简体中文）

モデル重み、カスタムノード、三面図、生成結果は同梱していません。各モデルとノードの配布条件は、それぞれの配布元で確認してください。

## 動作の流れ

```text
三面図 ──→ Qwen K0（シーンのマスター）
             ├─ K0 + 三面図 ──→ K1
             ├─ K0 + 三面図 ──→ K2
             └─ K0 + 三面図 ──→ K3

MiniMax H3 FL2VA: K0→K1、K1→K2、K2→K3（各73フレーム、24 fps）
                                   ↓
                             3本を結合
```

K1→K2→K3の逐次画像編集は行いません。服装、固定された背景、照明、構図をK0から維持するため、K1～K3はすべてK0から独立して編集します。動画プロンプトには、波や葉など継続して動く背景要素の**映像上の動き**も書いてください。

## 必要な環境とノード

ComfyUI本体にはQwen Image 2.1、MiniMax H3、`BlockSparseAttention`、`JsonExtractString`、`StringFormat`などのノードが必要です。確認に使った環境は **ComfyUI 0.37.0、Python 3.13.13、Windows、RTX 4070 12 GB** です。ほかの環境で同じ速度やメモリ使用量になることは確認していません。

次のカスタムノードを各配布元の手順で導入し、ComfyUIを再起動してください。

| ノード集 | このワークフローで使うノード |
| --- | --- |
| [Comfyui-Spectrum-Qwen2.1](https://github.com/awdqwdasdg/Comfyui-Spectrum-Qwen2.1) | `SpectrumQwenImage21` |
| [ComfyUI-KJNodes](https://github.com/kijai/ComfyUI-KJNodes) | `MiniMaxChunkFeedForward` |
| [ComfyUI-MiniMax-H3-MotionCache-FastVAE](https://github.com/Mozer/ComfyUI-MiniMax-H3-MotionCache-FastVAE) | `MiniMaxH3FastVAEDecode`（MotionCacheは使いません） |

検証時のGitコミット：ComfyUI `1568e6c`、Spectrum `9211073`、KJNodes `d3cfe21`、FastVAE `b719329`。将来の版でノード名や入力が変わった場合は、各配布元の変更履歴を確認してください。

## 必要なモデル

下表はワークフローのローダーに設定された**正確なファイル名**です。各リンク先から利用者が取得し、自分のComfyUIの `models/` 以下の該当フォルダに置いてください。既存の共有モデルフォルダを `extra_model_paths.yaml` で参照しても構いません。別のモデル名や量子化版を使う場合は、ローダーで選び直し、結果を自分で確認してください。

| 用途 | `models/` 以下の配置先・ファイル名 | 配布元 |
| --- | --- | --- |
| Qwen拡散モデル | `diffusion_models/qwen_image_2.1_int8_convrot.safetensors` | [Comfy-Org](https://huggingface.co/Comfy-Org/Qwen-Image-2.1/blob/main/diffusion_models/qwen_image_2.1_int8_convrot.safetensors) |
| Qwenテキストエンコーダー | `text_encoders/qwen3vl_8b_fp8_scaled.safetensors` | [Comfy-Org](https://huggingface.co/Comfy-Org/Qwen3-VL/blob/main/text_encoders/qwen3vl_8b_fp8_scaled.safetensors) |
| Qwen VAE | `vae/qwen_image_2.1_vae_bf16.safetensors` | [Comfy-Org](https://huggingface.co/Comfy-Org/Qwen-Image-2.1/blob/main/vae/qwen_image_2.1_vae_bf16.safetensors) |
| H3拡散モデル（4-step fused） | `diffusion_models/minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors` | [MATLOWAI](https://huggingface.co/MATLOWAI/minimax-h3-fused-turbo-int8-convrot/blob/main/diffusion_models/minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors) |
| H3テキストエンコーダー | `text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors` | [Comfy-Org](https://huggingface.co/Comfy-Org/MiniMax-H3/blob/main/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors) |
| H3映像VAE | `vae/minimax_h3_video_vae_int8_convrot.safetensors` | [Comfy-Org](https://huggingface.co/Comfy-Org/MiniMax-H3/blob/main/vae/minimax_h3_video_vae_int8_convrot.safetensors) |
| H3音声VAE | `vae/minimax_h3_audio_vae_fp32.safetensors` | [Comfy-Org](https://huggingface.co/Comfy-Org/MiniMax-H3/blob/main/vae/minimax_h3_audio_vae_fp32.safetensors) |

H3のfusedモデルには高速化用の重みが統合されています。このJSONに追加のTurbo LoRAノードはありません。モデル本体をこのリポジトリのライセンスで再配布するものではありません。

## 使い方

1. 同じ人物を正面・側面・背面から示す三面図を**1枚**、ComfyUIの `input/` に入れます。画像の使用権を確認してください。
2. ワークフローJSONをComfyUIに読み込み、左の **「01 毎回選択」** `LoadImage` で自分の三面図を選びます。初期値の `SELECT_YOUR_THREE_VIEW.png` は差し替え用の表示名で、画像ファイルは同梱していません。
3. 使いたい言語の指示ファイル（[日本語](prompt_guide_ja.md) / [English](prompt_guide_en.md) / [简体中文](prompt_guide_zh-CN.md)）を、ChatGPT、Claude、Gemini、画像対応のローカルLLMなどの**チャットに添付**します。ファイル添付に対応していなければ、ファイルの全文をチャットへ貼ります。MDの内容を毎回編集する必要はありません。
4. 画像を読めるAIなら、**ComfyUIで選んだものと同じ三面図**もそのチャットに添付します。そして「この人物が温室で紙飛行機を持ち上げ、飛ばし、着地を見届ける。固定カメラで、台詞なし」のように、作りたい動画を普通の文章で送ります。3つの動作を指定しても、ひとつの案から3シーンへ分けるよう頼んでも構いません。
5. AIが返した**7キーのJSON全文だけ**を、ワークフローの **「02 毎回ここだけ編集」** に貼ります。コードブロックの囲いや説明文が付いた場合は除き、最初の `{` から最後の `}` までを貼ります。初期値の温室と紙飛行機のJSONは入力例です。
6. 画像・モデルの選択に欠落がないことを確認してから、ComfyUIでQueue Promptを実行します。

三面図を画像対応AIにも添付すると、顔・髪型・服装・画風・意図した小物をプロンプトへ反映しやすくなります。ただしAIが画像を読み違えることもあり、最終画像の品質向上を保証するものではありません。**三面図をComfyUIへ入力する手順は、AIへの添付の有無にかかわらず必須**です。画像を読めないローカルLLMでは、指示文の全文に加えて人物の顔、髪型、服装、画風、必要な小物を文章で伝えてください。生成されたJSONとキーフレームは目視で確認してください。

JSONのキーは `k0_prompt`、`k1_end`、`scene1_h3`、`k2_end`、`scene2_h3`、`k3_end`、`scene3_h3` の7個です。キー名を変えたり値を空にしたりすると、対応するプロンプトを抽出できません。

出力はComfyUIの `output/qwen_h3_3scene/` に保存されます。`keyframe_K0_*`～`keyframe_K3_*` が画像、`scene_01_*`～`scene_03_*` が各シーン、`final_*.mp4` が結合済み動画です。

## 現在の固定仕様と注意点

- 4キーフレーム、3シーン固定です。シーン数を入力に応じて増減する機能はありません。
- H3は各シーン **1024×1792、73フレーム、24 fps、4-step** に設定されています。3本の生成と結合にはGPU時間と十分なRAM・ディスク容量が必要です。
- 三面図は人物の同一性の参照です。最終画像に三面図のパネルや複数人が残る場合、`k0_prompt` と入力画像を確認してください。
- 背景の配置を保つ指示だけでは、海の波などが静止して見える場合があります。動かしたい要素は、各 `scene*_h3` の**映像描写**に明記してください。`overall_soundscape` は音の指示です。
- 音声は各シーン（約3秒）ごとにH3が独立して生成するため、シーンの境目（約3秒ごと）で環境音やBGMが途切れたり急変したりします。結合後動画の音声はクリップ同士を単純連結したものであり、シームレスな連続再生には対応していません。一貫したBGMや滑らかな音響が必要な場合は、生成後に外部BGMを重ねるなどの後処理を推奨します。
- 初期の温室・紙飛行機JSONは公開用の汎用例です。この入力例での生成結果は検証していません。ワークフローの構成は、別の三面図とプロンプトを使ったローカル実行で確認しています。

## ライセンスと帰属

このリポジトリに含むワークフローと文書は [MIT License](LICENSE) で公開します。ComfyUI、外部ノード、Qwen Image 2.1、MiniMax H3、および各派生モデルには、それぞれの配布元のライセンスが適用されます。モデル・ノード・参照画像をこのリポジトリに同梱していません。
