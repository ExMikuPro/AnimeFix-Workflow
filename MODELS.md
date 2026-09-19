# Models

本文件由 `workflow_facefix_v2.json` 的 `widgets_values` **实际扫描**生成。

**This repository does not redistribute any third-party models.**
**Users must download required models from their original / official sources and comply with their respective licenses.**

所有模型文件都**不在本仓库中**。下表只记录工作流中写入的文件名、用途与应放置的位置。

---

## Checkpoint

| | |
|---|---|
| **Filename** | `waiIllustriousSDXL_v170.safetensors` |
| **Purpose** | 主生成模型（SDXL / Illustrious 系二次元底模） |
| **Install location** | `ComfyUI/models/checkpoints/` |

工作流中唯一的 `CheckpointLoaderSimple` 节点（#88）使用此文件。放大重绘、手部修复、面部修复、眼睛修复**全部复用同一个 Checkpoint**，不会加载第二个底模。

Please obtain this model from its original distribution source.

---

## VAE

| | |
|---|---|
| **Filename** | `taesdxl` |
| **Purpose** | VAE 解码 / 编码 |
| **Actual location** | `ComfyUI/models/vae_approx/`（ComfyUI 自带的预览用近似 VAE） |

`VAELoader` 节点（#91）中的值为 `taesdxl`。

> **注意：** `taesdxl` 是 ComfyUI 自带的**预览用近似 VAE**，位于 `ComfyUI/models/vae_approx/`（由 ComfyUI 内置提供，用于低成本 latent 预览），并非完整质量的 SDXL VAE，**不需要你额外下载**。
>
> 若你希望获得完整解码质量，请自备一个 SDXL VAE（例如常见的 `sdxl_vae.safetensors`），放入 `ComfyUI/models/vae/`，并把它的文件名填进 `VAELoader` 节点（注意不要带扩展名以外的路径）。
>
> 本仓库不提供该 VAE 替代文件，也不指定任何具体来源。

---

## LoRA

工作流通过 `LoRA Stacker` 节点（#116）配置，当前**启用的** LoRA 共 2 条：

| # | Filename | Weight | Install location |
|---|---|---|---|
| 1 | `USNR STYLE_XL_lokr.safetensors` | `0.70` | `ComfyUI/models/loras/` |
| 2 | `748cmSDXL.safetensors` | `0.80` | `ComfyUI/models/loras/` |

两条 LoRA 通过 5 个 `CR Apply LoRA Stack` 节点分别应用到基础模型、手部修复、面部修复等各阶段（使用同一组权重）。

`LoRA Stacker` 节点中还预留了第 3–50 号槽位，当前全部为 `None`（未启用），**不需要下载任何对应文件**。

> ⚠️ 当前的 Checkpoint 与 LoRA 只是**作者开发测试时使用的配置**。仓库不包含这些文件，用户需要自行获取。
> 你可以替换为其它兼容的 SDXL / Illustrious 模型与 LoRA；不同模型版本会导致不同效果，也可能需要重新调整权重、提示词与 denoise。

Please obtain these models from their original distribution sources.

---

## Upscaler

| | |
|---|---|
| **Filename** | `RealESRGAN_x2.pth` |
| **Purpose** | Upscale Refiner 阶段的模型放大（2×） |
| **Install location** | `ComfyUI/models/upscale_models/` |

由 `UpscaleModelLoader`（#120）加载，随后经 `ImageScale` 缩放到 1248×1824 再进行低 denoise 重绘。

Please obtain this model from its original distribution source.

---

## Face Detector

| | |
|---|---|
| **Filename** | `bbox/face_yolov8m.pt` |
| **Purpose** | 脸部 bbox 检测（FaceDetailerPipe） |
| **Install location** | `ComfyUI/models/ultralytics/bbox/` |

由 `UltralyticsDetectorProvider`（#213）加载。工作流中该检测器同时供 **Face Repair** 与 **Eye Detailer** 阶段使用（通过 `FACE DETECTOR` 广播）。

> 请把文件放在 `ComfyUI/models/ultralytics/bbox/` 下。`UltralyticsDetectorProvider` 的下拉列表会自动扫描 `ComfyUI/models/ultralytics/` 目录树；若你的插件版本使用其它扫描路径，请以你实际安装的版本为准。

---

## Eye Detector

| | |
|---|---|
| **Filename** | `bbox/Eyeful_v2-Individual.pt` |
| **Purpose** | 眼睛检测（Eye Detailer 阶段） |
| **Install location** | `ComfyUI/models/ultralytics/bbox/` |

由 `UltralyticsDetectorProvider`（#125）加载，结果通过 `EYE DETECTOR` 广播给 Eye Detailer。眼部 bbox 会以负值 padding 收缩，聚焦眼周区域。

Please obtain this model from its original distribution source.

---

## Hand Detector

| | |
|---|---|
| **Filename** | `bbox/hand_yolov8s.pt` |
| **Purpose** | 手部 bbox 检测（HAND REPAIR 阶段） |
| **Install location** | `ComfyUI/models/ultralytics/bbox/` |

由 `UltralyticsDetectorProvider`（#261）加载。检测出的 bbox 会作为 SAM 的提示，生成手部轮廓 mask。

Please obtain this model from its original distribution source.

---

## SAM

| | |
|---|---|
| **Filename** | `sam_vit_b_01ec64.pth` |
| **Purpose** | 由 bbox 提示生成精确分割 mask（手部 / 脸部轮廓） |
| **Install location** | `ComfyUI/models/sams/` |

由 2 个 `SAMLoader` 节点加载（#206 用于脸部/眼睛链路， #262 用于手部链路），`device_mode` 均为 `AUTO`。

Please obtain this model from its original distribution source.

---

## ControlNet

| | |
|---|---|
| **Filename** | `SDXL/controlnet-union-sdxl-1.0/diffusion_pytorch_model.safetensors` |
| **Purpose** | SDXL Union ControlNet（`canny/lineart/anime_lineart/mlsd` 模式） |
| **Install location** | `ComfyUI/models/controlnet/SDXL/controlnet-union-sdxl-1.0/` |

由 `ControlNetLoader`（#128）加载，配合 `SetUnionControlNetType`（#129）设置为 `canny/lineart/anime_lineart/mlsd`，经 `ControlNetApply`（#146）以 strength `0.30` 应用。

> ⚠️ 请注意完整路径中包含子目录 `SDXL/controlnet-union-sdxl-1.0/`，这是 `ControlNetLoader` 的下拉选择值。请保持相同的目录层级，否则该节点会找不到文件。
>
> `ControlNet` 的根目录为 `ComfyUI/models/controlnet/`，子目录层级由 `ControlNetLoader` 的下拉选择值决定，请保持一致。

Please obtain this model from its original distribution source.

---

## Optional

### MeshGraphormer hand mask branch

`hand mask source` 开关（`LazySwitchKJ` #266）可以切换到 **MeshGraphormer** 分支：

| | |
|---|---|
| **Node** | `MeshGraphormer+ImpactDetector-DepthMapPreprocessor`（#265） |
| **Provided by** | [comfyui_controlnet_aux](https://github.com/Fannovel16/comfyui_controlnet_aux)（**必须安装该插件**） |
| **Extra weights** | 该预处理器首次使用时会尝试下载 MeshGraphormer 权重；工作流 JSON 中**不包含**任何 MeshGraphormer 权重文件名 |
| **Output usage** | 该分支的输出**仅用于生成 mask**，工作流中**未连接到任何 ControlNet** |

此路径属于**可选**功能：

- 开关处于默认状态时使用的是 **YOLO + SAM** 路径，不需要 MeshGraphormer。
- 若你启用该分支，需要保证 `comfyui_controlnet_aux` 已安装且其依赖可用。
- 作者**未在本仓库中验证或分发**任何 MeshGraphormer 权重。

Please obtain any required MeshGraphormer weights from their original distribution source.

---

## Summary

| Category | Count |
|---|---|
| Checkpoint | 1 |
| VAE | 1（`taesdxl`，ComfyUI 自带预览用近似 VAE） |
| LoRA (enabled) | 2 |
| Upscaler | 1 |
| Face Detector | 1 |
| Eye Detector | 1 |
| Hand Detector | 1 |
| SAM | 1 |
| ControlNet | 1 |
| Optional (MeshGraphormer) | 权重文件名未在 JSON 中指定 |

**全部文件均需用户自行获取。本仓库不包含、也不重新分发任何第三方模型。**

---

## Third-party licenses

模型文件的许可证由各自的作者 / 发布方决定。用户需要自行确认第三方资源的使用与再分发许可。

Please obtain every model from its original distribution source and comply with its respective license.
