# AnimeFix-Workflow

> A modular anime generation and refinement workflow for ComfyUI, focused on SDXL / Illustrious.

[![ComfyUI](https://img.shields.io/badge/ComfyUI-Workflow-111111?style=flat-square)](https://github.com/comfyanonymous/ComfyUI)
[![License](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](LICENSE)
[![Workflow](https://img.shields.io/badge/Workflow-FaceFix%20V2-orange?style=flat-square)](workflow_facefix_v2.json)

AnimeFix-Workflow 是一套面向 **SDXL / Illustrious 二次元生成** 的模块化 ComfyUI 工作流，将文生图、放大、高清重绘、手部修复、面部修复和眼睛细化串成一条可独立调节的处理链。

```text
Txt2Img
   ↓
Base Upscale
   ↓
Upscale Refiner
   ↓
Hand Repair
   ↓
Face Repair
   ↓
Eye Detailer
   ↓
Output
```

当前主工作流：[`workflow_facefix_v2.json`](workflow_facefix_v2.json)

> 本仓库仅提供工作流配置与文档，不包含任何第三方模型、ComfyUI 本体或 Custom Nodes 源码。

---

## Features

- **SDXL / Illustrious oriented**
  - 默认工作流面向二次元 SDXL / Illustrious 模型设计。
- **Multi-stage generation**
  - Txt2Img、放大和高清重绘分阶段执行，可分别调整采样参数。
- **Base Upscale + Refiner**
  - 使用 ESRGAN 放大，再进入低 denoise 高清重绘阶段。
- **Hand Repair**
  - 使用 YOLO 手部检测、SAM / MeshGraphormer mask 与局部 inpaint 修复手部结构。
  - 支持在 **YOLO + SAM** 与 **MeshGraphormer** 两种手部 mask 来源之间切换。
- **Face Repair**
  - 使用 FaceDetailer + face detector + SAM 对面部进行局部修复。
- **Eye Detailer**
  - 使用独立眼睛检测器进一步细化眼睛区域。
- **LoRA Stack**
  - 通过 LoRA Stacker 统一管理多条 LoRA，并复用到不同修复阶段。
- **Modular routing**
  - 大量跨区域连接通过 Set / Get 节点完成，避免工作流出现大量长距离连线。
- **Repair switches**
  - Hand Repair 和 Face Repair 可以独立旁路，方便测试不同组合。
- **Four-stage comparison preview**
  - 工作流内置 4 个前后对比预览：
    1. Txt2Img → Base Upscale
    2. Base Upscale → Upscale Refiner
    3. Hand Repair 前 → Hand Repair 后
    4. Txt2Img 原图 → 最终输出

---

## Workflow Structure

| Group | Purpose |
|---|---|
| `00 USER INPUT` | Positive / Negative Prompt |
| `01 MODEL & SHARED RESOURCES` | Checkpoint、VAE、LoRA、检测器、Upscaler |
| `02 BASE PIPE SETUP` | 基础生成管线 |
| `03 UPSCALE PIPE SETUP` | 放大重绘管线 |
| `04 FACE DETAILER SETUP` | Face Repair 条件与 Detailer Pipe |
| `05 EYE DETAILER SETUP` | Eye Detailer 条件与 Pipe |
| `10 TXT2IMG` | 初始生成 |
| `20 BASE UPSCALE` | 模型放大 |
| `30 UPSCALE REFINER` | 高清重绘 |
| `40 HAND REPAIR` | 手部检测、mask 与局部重绘 |
| `50 FACE REPAIR` | 面部修复 |
| `60 EYE DETAILER` | 眼睛细化 |
| `70 OUTPUT` | 最终输出 |
| `90 PREVIEW ①–④` | 四级前后对比 |

---

## Requirements

### 1. ComfyUI

先安装并确认 ComfyUI 可以正常运行：

- [ComfyUI](https://github.com/comfyanonymous/ComfyUI)

### 2. Custom Nodes

本工作流依赖多个第三方 Custom Nodes，例如：

- ComfyUI Impact Pack
- ComfyUI Impact Subpack
- ComfyUI-KJNodes
- rgthree-comfy
- Comfyroll Studio
- ComfyUI Inspire Pack
- smZNodes
- tinyterraNodes
- ComfyUI Detail Daemon
- efficiency-nodes-comfyui
- ComfyUI AutomaticCFG
- cg-use-everywhere
- comfyui_controlnet_aux

完整节点归属、用途和仓库地址请查看：

**[REQUIREMENTS.md](REQUIREMENTS.md)**

推荐在载入工作流后使用 **ComfyUI Manager → Install Missing Custom Nodes** 检查缺失节点。

### 3. Models

工作流会引用 Checkpoint、LoRA、Upscaler、YOLO Detector、SAM 和 ControlNet 等模型文件。

完整文件名、用途和安装目录请查看：

**[MODELS.md](MODELS.md)**

> 所有模型均需用户自行从原始发布源获取。本仓库不重新分发任何第三方模型。

---

## Quick Start

1. 安装并启动 ComfyUI。
2. 安装 [REQUIREMENTS.md](REQUIREMENTS.md) 中列出的 Custom Nodes。
3. 按照 [MODELS.md](MODELS.md) 准备对应模型。
4. 下载本仓库中的 [`workflow_facefix_v2.json`](workflow_facefix_v2.json)。
5. 将 JSON 拖入 ComfyUI Canvas，或使用 **Workflow → Open**。
6. 如果出现 Missing Nodes，使用 ComfyUI Manager 补齐依赖。
7. 在 `00 USER INPUT` 中填写 Positive / Negative Prompt。
8. 在 `01 MODEL & SHARED RESOURCES` 中选择 Checkpoint、LoRA 和相关模型。
9. 点击 **Queue Prompt**。
10. 在 `90 PREVIEW ①–④` 中检查各阶段前后差异。

---

## Basic Usage

### Prompt

主要提示词位于 `00 USER INPUT`：

- **Positive Prompt**：描述角色、姿态、服装、构图、场景等。
- **Negative Prompt**：排除低质量、畸形结构、文字、水印等问题。

Hand Repair 和 Face Repair 还包含各自的局部正 / 负向提示词，用于修复阶段，不需要把所有局部修复标签都塞进主提示词。

### Checkpoint and LoRA

工作流 JSON 中保存的 Checkpoint / LoRA 只是作者开发时的测试配置，并不是强制要求。

你可以替换为其它兼容的 SDXL / Illustrious 模型，但更换底模或 LoRA 后通常需要重新调整：

- Prompt
- LoRA weight
- CFG
- denoise
- sampler / scheduler
- Detailer 参数

### Repair Switches

| Switch | Behavior |
|---|---|
| `hand repair enable` | OFF 时跳过手部修复，原图直接向后传递 |
| `hand mask source` | OFF = YOLO + SAM；ON = MeshGraphormer |
| `face repair enable` | OFF 时跳过 Face Repair，Hand Repair 结果直接向后传递 |

---

## Default Pipeline Notes

### Txt2Img

默认初始 latent：

```text
832 × 1216
```

### Base Upscale

默认使用：

```text
RealESRGAN_x2.pth
```

然后缩放到：

```text
1248 × 1824
```

### Upscale Refiner

放大图像会重新编码为 latent，再以较低 denoise 进行细节重绘。

### Hand Repair

默认流程：

```text
Hand Detector
   ↓
YOLO bbox
   ↓
SAM / MeshGraphormer mask
   ↓
Mask Expand + Feather
   ↓
MaskDetailer local inpaint
```

局部修复主要负责手掌、手指结构和局部线条。复杂遮挡、交叉手指或极端透视仍可能失败。

### Face Repair

默认使用 `face_yolov8m.pt` 检测面部区域，并通过 FaceDetailer 进行局部重绘。

### Eye Detailer

Face Repair 完成后，Eye Detailer 会再次检测眼睛区域并进行局部细化。

---

## Preview / Debugging

| Preview | A (Left) | B (Right) |
|---|---|---|
| ① Base Upscale | Txt2Img 原图 | Base Upscale |
| ② Upscale Refiner | Base Upscale | 高清重绘后 |
| ③ Hand Repair | 修手前 | HAND FIXED |
| ④ Final vs Original | Txt2Img 原图 | 最终输出 |

如果 A / B 看起来完全相同，常见原因包括：

- 对应修复模块被关闭。
- Detector 没有检测到目标区域。
- mask 为空或范围过小。
- denoise 较低，局部变化不明显。
- 输入图本身已经足够稳定。

---

## Models Used by the Default Workflow

| Type | Default reference |
|---|---|
| Checkpoint | `waiIllustriousSDXL_v170.safetensors` |
| VAE | `taesdxl` |
| LoRA | `USNR STYLE_XL_lokr.safetensors` |
| LoRA | `748cmSDXL.safetensors` |
| Upscaler | `RealESRGAN_x2.pth` |
| Face Detector | `face_yolov8m.pt` |
| Eye Detector | `Eyeful_v2-Individual.pt` |
| Hand Detector | `hand_yolov8s.pt` |
| SAM | `sam_vit_b_01ec64.pth` |
| ControlNet | SDXL Union ControlNet |

具体目录和说明请以 [MODELS.md](MODELS.md) 为准。

> `taesdxl` 是 ComfyUI 自带的近似 VAE，主要用于低成本 latent 预览。若你需要更高质量的最终 VAE 解码，请根据自己的 SDXL 模型配置合适的完整 VAE。

---

## Hardware

本项目不声明固定的最低 GPU / VRAM 要求。

显存需求会受到以下因素影响：

- Checkpoint 精度和大小
- 初始分辨率
- 放大目标分辨率
- 同时启用的修复模块
- Custom Nodes 的实现版本
- VAE / SAM / Detector 的加载方式

如果显存不足，建议先只运行 Txt2Img，再逐步启用 Upscale、Hand Repair、Face Repair 和 Eye Detailer。

---

## Troubleshooting

### Missing Nodes

使用：

```text
ComfyUI Manager → Install Missing Custom Nodes
```

同时对照 [REQUIREMENTS.md](REQUIREMENTS.md)。

### Model not found

确认模型文件名与目录和 [MODELS.md](MODELS.md) 一致。

ComfyUI 中部分 Loader 会保存完整的子目录名称，因此目录层级也可能影响下拉列表中的路径。

### Hand Repair does nothing

检查：

- `hand repair enable` 是否开启
- Hand Detector 是否检测到手
- `hand mask source` 当前选择的分支
- SAM / MeshGraphormer 是否正常工作
- mask 是否覆盖手部
- denoise 是否过低

### Face / Eye repair changes identity too much

尝试降低：

- denoise
- CFG
- repair prompt 权重

并避免在局部修复提示词中加入过多会改变角色身份的描述。

---

## Repository Structure

```text
AnimeFix-Workflow/
├── workflow_facefix_v2.json   # Main ComfyUI workflow
├── README.md                  # Project overview and usage
├── REQUIREMENTS.md            # Required custom nodes
├── MODELS.md                  # Required / referenced models
├── LICENSE                    # Repository license
└── .gitignore
```

---

## Limitations

AnimeFix-Workflow 不是“一键完美修复器”。

以下情况仍可能生成错误：

- 手指复杂交叉或严重遮挡
- 极端透视 / foreshortening
- 手部与面部大面积重叠
- 牙齿、舌头等小尺寸口腔细节
- 过小的人脸或眼睛
- Detector 漏检
- 不同模型 / LoRA 对 Detailer prompt 的理解差异

生成式模型具有随机性。相同提示词和参数并不代表不同环境或不同版本中一定得到完全一致的输出。

---

## Documentation

- [Custom Nodes / Requirements](REQUIREMENTS.md)
- [Models](MODELS.md)
- [Workflow JSON](workflow_facefix_v2.json)
- [License](LICENSE)

---

## License

本仓库作者原创的工作流配置与文档按照 [MIT License](LICENSE) 发布。

该许可证 **不覆盖** 工作流引用的第三方：

- ComfyUI
- Custom Nodes
- Checkpoints
- LoRA
- VAE
- ControlNet
- SAM
- YOLO / Ultralytics Detector
- Upscale models
- MeshGraphormer weights

第三方组件继续遵循各自的许可证和使用条款。

---

## Disclaimer

- 本仓库不提供或重新分发第三方模型。
- 用户应从模型 / 插件的原始发布源获取文件。
- 用户有责任遵守第三方资源对应的许可证。
- 工作流效果会随模型、LoRA、seed、ComfyUI 及 Custom Node 版本发生变化。
- 项目不保证特定修复结果、跨版本兼容性或输出一致性。
- 使用本工作流生成内容时，使用者应自行确认相关内容与资源的许可和合规要求。

---

## English Summary

**AnimeFix-Workflow** is a modular ComfyUI workflow designed for SDXL / Illustrious anime image generation and refinement.

Its pipeline is:

```text
Txt2Img → Base Upscale → Upscale Refiner → Hand Repair → Face Repair → Eye Detailer → Output
```

The workflow provides local hand inpainting, FaceDetailer-based face repair, a dedicated eye-detailing stage, reusable LoRA stacks, optional repair switches, and four before/after comparison views for debugging each major stage.

This repository contains **workflow configuration and documentation only**. Third-party models and custom nodes must be installed separately from their original sources.

See [REQUIREMENTS.md](REQUIREMENTS.md) and [MODELS.md](MODELS.md) before loading the workflow.
