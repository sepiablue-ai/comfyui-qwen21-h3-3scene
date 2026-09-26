# Qwen Image 2.1 → MiniMax H3: Three-Scene Video Workflow

[日本語](README.md) | [English](README_en.md)

Select one three-view character sheet and paste a seven-field JSON object from an AI chat into a single text box. This ComfyUI workflow generates four keyframes, three video scenes of about three seconds each, and a joined video. Run anime and live-action references **separately**, once per reference sheet.

## Included files

- [`qwen_h3_3scene_workflow.json`](qwen_h3_3scene_workflow.json): the ComfyUI GUI workflow
- [`prompt_guide_ja.md`](prompt_guide_ja.md): instructions for generating the input JSON (Japanese)
- [`prompt_guide_en.md`](prompt_guide_en.md): the same instructions (English)
- [`prompt_guide_zh-CN.md`](prompt_guide_zh-CN.md): the same instructions (Simplified Chinese)

Model weights, custom nodes, reference sheets, and generated results are not included. Check the terms of each model and node at its distribution source.

## How it works

```text
Three-view sheet ──→ Qwen K0 (scene master)
                     ├─ K0 + sheet ──→ K1
                     ├─ K0 + sheet ──→ K2
                     └─ K0 + sheet ──→ K3

MiniMax H3 FL2VA: K0→K1, K1→K2, K2→K3 (73 frames each at 24 fps)
                                  ↓
                          Join the three clips
```

K2 is not an image edit of K1, and K3 is not an image edit of K2. Each of K1–K3 is independently edited from K0 to preserve clothing, fixed background objects, lighting, and framing. Video prompts should also describe the **visible motion** of environmental elements that keep moving, such as waves or leaves.

## Requirements and nodes

The workflow requires Qwen Image 2.1, MiniMax H3, `BlockSparseAttention`, `JsonExtractString`, `StringFormat`, and the other node types recorded in the JSON. The environment checked for this release was **ComfyUI 0.37.0, Python 3.13.13, Windows, and an RTX 4070 with 12 GB VRAM**. Speed and memory use on other systems have not been verified.

Install these custom node packs using their respective instructions, then restart ComfyUI:

| Node pack | Node used by this workflow |
| --- | --- |
| [Comfyui-Spectrum-Qwen2.1](https://github.com/awdqwdasdg/Comfyui-Spectrum-Qwen2.1) | `SpectrumQwenImage21` |
| [ComfyUI-KJNodes](https://github.com/kijai/ComfyUI-KJNodes) | `MiniMaxChunkFeedForward` |
| [ComfyUI-MiniMax-H3-MotionCache-FastVAE](https://github.com/Mozer/ComfyUI-MiniMax-H3-MotionCache-FastVAE) | `MiniMaxH3FastVAEDecode` (MotionCache is not used) |

Git commits at the time of validation: ComfyUI `1568e6c`, Spectrum `9211073`, KJNodes `d3cfe21`, FastVAE `b719329`. If a future version changes node names or inputs, consult the corresponding project's change history.

## Required models

These are the **exact filenames configured in the workflow's model loaders**. Download them from their respective sources and place them under the indicated directories in your ComfyUI `models/` folder. You can also reference an existing shared model store through `extra_model_paths.yaml`. If you use other filenames or quantizations, select them in the loaders and check the results yourself.

| Purpose | Location and filename under `models/` | Source |
| --- | --- | --- |
| Qwen diffusion model | `diffusion_models/qwen_image_2.1_int8_convrot.safetensors` | [Comfy-Org](https://huggingface.co/Comfy-Org/Qwen-Image-2.1/blob/main/diffusion_models/qwen_image_2.1_int8_convrot.safetensors) |
| Qwen text encoder | `text_encoders/qwen3vl_8b_fp8_scaled.safetensors` | [Comfy-Org](https://huggingface.co/Comfy-Org/Qwen3-VL/blob/main/text_encoders/qwen3vl_8b_fp8_scaled.safetensors) |
| Qwen VAE | `vae/qwen_image_2.1_vae_bf16.safetensors` | [Comfy-Org](https://huggingface.co/Comfy-Org/Qwen-Image-2.1/blob/main/vae/qwen_image_2.1_vae_bf16.safetensors) |
| H3 diffusion model (4-step fused) | `diffusion_models/minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors` | [MATLOWAI](https://huggingface.co/MATLOWAI/minimax-h3-fused-turbo-int8-convrot/blob/main/diffusion_models/minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors) |
| H3 text encoder | `text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors` | [Comfy-Org](https://huggingface.co/Comfy-Org/MiniMax-H3/blob/main/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors) |
| H3 video VAE | `vae/minimax_h3_video_vae_int8_convrot.safetensors` | [Comfy-Org](https://huggingface.co/Comfy-Org/MiniMax-H3/blob/main/vae/minimax_h3_video_vae_int8_convrot.safetensors) |
| H3 audio VAE | `vae/minimax_h3_audio_vae_fp32.safetensors` | [Comfy-Org](https://huggingface.co/Comfy-Org/MiniMax-H3/blob/main/vae/minimax_h3_audio_vae_fp32.safetensors) |

The fused H3 model already incorporates the acceleration weights. This JSON does not contain an additional Turbo LoRA node. This repository's license does not grant rights to redistribute the model weights.

## Usage

1. Put **one** sheet showing the same character from the front, side, and back in ComfyUI's `input/` directory. Make sure you have the right to use the image.
2. Load the workflow JSON in ComfyUI. In the `LoadImage` node labeled `01 毎回選択` (select each run), choose your own sheet. The initial value `SELECT_YOUR_THREE_VIEW.png` is only a placeholder; that image is not included.
3. Attach the prompt guide in your preferred language ([Japanese](prompt_guide_ja.md) / [English](prompt_guide_en.md) / [Simplified Chinese](prompt_guide_zh-CN.md)) to a chat with ChatGPT, Claude, Gemini, a vision-capable local LLM, or another suitable AI. If the chat cannot accept file attachments, paste the guide's full text instead. You do not need to edit the guide for each run.
4. If the AI can view images, also attach **the same three-view sheet selected in ComfyUI**. Describe your desired video in ordinary language, for example: “This character lifts a paper airplane in a greenhouse, throws it, and watches it land. Fixed camera, no dialogue.” You can specify three actions or ask the AI to divide one idea into three scenes.
5. Paste **only the full seven-key JSON object** returned by the AI into the workflow's single prompt box labeled `02 毎回ここだけ編集` (edit this field each run). If the response contains a code fence or commentary, remove it and paste from the first `{` through the final `}`. The initial greenhouse and paper-airplane JSON is an example.
6. Confirm that the image and models are selected, then run Queue Prompt in ComfyUI.

Attaching the sheet to a vision-capable AI can help it reflect the face, hairstyle, outfit, visual style, and intended props in its prompts. The AI may still misread the image, so this does not guarantee better final images. **Selecting the sheet in ComfyUI is required whether or not you attach it to the AI chat.** For a text-only local LLM, paste the guide and describe the character's face, hair, outfit, visual style, and required props in words. Review the returned JSON and keyframes visually.

The seven keys are `k0_prompt`, `k1_end`, `scene1_h3`, `k2_end`, `scene2_h3`, `k3_end`, and `scene3_h3`. Changing key names or leaving a value empty prevents the corresponding prompt from being extracted.

Files are saved under ComfyUI's `output/qwen_h3_3scene/`: `keyframe_K0_*` through `keyframe_K3_*` are images, `scene_01_*` through `scene_03_*` are the three clips, and `final_*.mp4` is the joined video.

## Current fixed settings and limitations

- The workflow always makes four keyframes and three scenes. It does not adapt the scene count to the input.
- H3 is set to **1024×1792, 73 frames, 24 fps, and 4 steps per scene**. Generating and joining three clips requires GPU time, sufficient RAM, and disk space.
- The three-view sheet is an identity reference. If the final image contains sheet panels or multiple people, check `k0_prompt` and the input image.
- Instructions to keep the background layout consistent may leave waves or other environmental elements looking still. Describe the motion you want in the **visual part** of every relevant `scene*_h3`. `overall_soundscape` specifies audio.
- Audio is generated by H3 independently for each ~3-second clip, so soundscapes and BGM will abruptly cut off or change at scene boundaries (every ~3 seconds). The joined video simply concatenates the audio tracks without crossfading. For continuous background music or smooth ambient sound, adding an external audio track in post-production is recommended.
- The initial greenhouse and paper-airplane JSON is a generic release example. Its generation results have not been tested. The workflow structure was checked in a local run with a different sheet and prompt.

## License and attribution

The workflow and documentation in this repository are released under the [MIT License](LICENSE). ComfyUI, external nodes, Qwen Image 2.1, MiniMax H3, and derived models remain subject to their respective licenses. Model weights, nodes, and reference images are not included in this repository.
