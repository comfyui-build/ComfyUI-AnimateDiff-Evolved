# ComfyUI的AnimateDiff插件

为ComfyUI提供了改进的[AnimateDiff](https://github.com/guoyww/AnimateDiff/)集成，以及被称为Evolved Sampling的先进采样选项，这些选项在AnimateDiff之外也可使用。请阅读AnimateDiff仓库的README和Wiki，以了解更多关于其核心工作原理的信息。

AnimateDiff工作流程通常会用到以下有用的节点包：
- [ComfyUI_FizzNodes](https://github.com/FizzleDorf/ComfyUI_FizzNodes)，提供BatchPromptSchedule节点的提示旅行功能。由FizzleDorf维护。
- [ComfyUI-Advanced-ControlNet](https://github.com/Kosinkadink/ComfyUI-Advanced-ControlNet)，用于使ControlNets与上下文选项配合工作，并控制哪些潜在因素应受到ControlNet输入的影响。包括SparseCtrl支持。由我维护。
- [ComfyUI-VideoHelperSuite](https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite)，用于加载视频，将图像合成为视频，以及进行各种图像/潜在操作，如追加、分割、复制、选择或计数。由AustinMroz和我积极维护。
- [comfyui_controlnet_aux](https://github.com/Fannovel16/comfyui_controlnet_aux)，提供vanilla ComfyUI中不存在的ControlNet预处理器。由Fannovel16维护。
- [ComfyUI_IPAdapter_plus](https://github.com/cubiq/ComfyUI_IPAdapter_plus)，用于IPAdapter支持。由cubiq (matt3o)维护。
- [ComfyUI-KJNodes](https://github.com/kijai/ComfyUI-KJNodes)，提供包括为动画GLIGEN选择坐标在内的杂项节点。由kijai维护。

# 安装

## 如果使用 ComfyUI Manager：

1. 查找 ```AnimateDiff Evolved```，并确保作者是 ```Kosinkadink```。安装它。
![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/2c7f29e1-d024-49e1-9eb0-d38070142584)


## 如果手动安装：
1. 将此仓库克隆到 `custom_nodes` 文件夹中。

# 模型设置：
1. 下载运动模块。您至少需要一个。不同的模块会产生不同的结果。
   - 原始模型 ```mm_sd_v14```, ```mm_sd_v15```, ```mm_sd_v15_v2```, ```v3_sd15_mm```: [HuggingFace](https://huggingface.co/guoyww/animatediff/tree/cd71ae134a27ec6008b968d6419952b0c0494cf2) | [Google Drive](https://drive.google.com/drive/folders/1EqLC65eR1-W-sGD0Im7fkED6c8GkiNFI) | [CivitAI](https://civitai.com/models/108836)
   - ```mm_sd_v14``` 的稳定微调版本，```mm-Stabilized_mid``` 和 ```mm-Stabilized_high```，由 **manshoety** 提供：[HuggingFace](https://huggingface.co/manshoety/AD_Stabilized_Motion/tree/main)
   - ```mm_sd_v15_v2``` 的微调版本，```mm-p_0.5.pth``` 和 ```mm-p_0.75.pth```，由 **manshoety** 提供：[HuggingFace](https://huggingface.co/manshoety/beta_testing_models/tree/main)
   - 更高分辨率的微调版本，```temporaldiff-v1-animatediff``` 由 **CiaraRowles** 提供：[HuggingFace](https://huggingface.co/CiaraRowles/TemporalDiff/tree/main)
   - 原始运动模型的 FP16/safetensor 版本，由 **continue-revolution** 托管（占用较少的存储空间，但使用相同的 VRAM，因为 ComfyUI 默认以 fp16 加载模型）：[HuffingFace](https://huggingface.co/conrevo/AnimateDiff-A1111/tree/main)
2. 将模型放置在这些位置之一（如果需要，可以重命名模型）：
   - ```ComfyUI/custom_nodes/ComfyUI-AnimateDiff-Evolved/models```
   - ```ComfyUI/models/animatediff_models```
3. 可选地，您可以使用 Motion LoRAs 来影响基于 v2 的运动模型，如 mm_sd_v15_v2 的运动。
   - [Google Drive](https://drive.google.com/drive/folders/1EqLC65eR1-W-sGD0Im7fkED6c8GkiNFI?usp=sharing) | [HuggingFace](https://huggingface.co/guoyww/animatediff) | [CivitAI](https://civitai.com/models/108836/animatediff-motion-modules)
   - 将 Motion LoRAs 放置在这些位置之一（如果需要，可以重命名 Motion LoRAs）：
      - ```ComfyUI/custom_nodes/ComfyUI-AnimateDiff-Evolved/motion_lora```
      - ```ComfyUI/models/animatediff_motion_lora```
4. 发挥创意！如果它适用于正常的图像生成，那么它（很可能）也适用于 AnimateDiff 生成。潜在的上采样？去尝试吧。ControlNets，一个或多个堆叠？当然可以。将 ControlNets 的条件掩码仅影响动画的一部分？当然可以。尝试各种方法，你会惊讶于你能做到的事情。下面包含了一些带有工作流程的示例。

注意：您还可以通过使用 ComfyUI 的 ```extra_model_paths.yaml``` 文件来使用自定义模型/motion lora 位置。运动模型文件夹的 ID 是 ```animatediff_models```，motion lora 文件夹的 ID 是 ```animatediff_motion_lora```。


# 功能
- 兼容几乎任何原版或自定义的 KSampler 节点。
- 支持 ControlNet、SparseCtrl 和 IPAdapter
- 通过在整个 unet 上滑动上下文窗口（上下文选项）和/或在运动模块内（视图选项）实现无限动画长度支持
- 调度上下文选项，以在采样过程中的不同点进行更改
- 支持 FreeInit 和 FreeNoise（FreeInit 在迭代选项下，FreeNoise 在 SampleSettings 的 noise_type 下拉菜单中）
- 从 [原始 AnimateDiff 仓库](https://github.com/guoyww/animatediff/) 实现的混合 Motion LoRAs。注意：原始的 LoRAs 实际上只适用于基于 v2 的运动模型，如 ```mm_sd_v15_v2```, ```mm-p_0.5.pth```, 和 ```mm-p_0.75.pth```。
     - 更新：现在可以通过 [AnimateDiff-MotionDirector 仓库](https://github.com/ExponentialML/AnimateDiff-MotionDirector) 训练没有 v2 限制的新 Motion LoRAs。感谢 ExponentialML 为 AnimateDiff 目的实现 MotionDirector！
- 使用 [ComfyUI_FizzNodes](https://github.com/FizzleDorf/ComfyUI_FizzNodes) 中的 BatchPromptSchedule 节点进行提示旅行
- 通过 Scale 和 Effect 多值输入控制运动量和运动模型对生成的影响。
     - 可以是浮点数、浮点数列表或掩码
- 通过噪声类型、噪声层和种子覆盖/偏移/批量偏移在 Sample Settings 和相关节点中进行自定义噪声调度
- 支持 AnimateDiff 模型 v1/v2/v3
- 通过 Gen2 节点同时使用多个运动模型（每个节点都支持）
- 支持 [HotshotXL](https://huggingface.co/hotshotco/Hotshot-XL/tree/main)（一个 SDXL 运动模块架构），```hsxl_temporal_layers.safetensors```。
     - 注意：您需要使用 ```autoselect``` 或 ```linear (HotshotXL/default)``` beta_schedule，上下文长度或总帧数（不使用上下文时）的最佳点是 8 帧，并且您需要使用 SDXL 检查点。
- 支持 [AnimateDiff-SDXL](https://github.com/guoyww/AnimateDiff/)，对应模型。经过几个月的测试，仍处于测试阶段。
     - 注意：您需要使用 ```autoselect``` 或 ```linear (AnimateDiff-SDXL)``` beta_schedule。除此之外，AnimateDiff-SDXL 的规则与 AnimateDiff 相同。
- 支持 [AnimateLCM](https://huggingface.co/wangfuyun/AnimateLCM)
     - 注意：您需要使用 ```autoselect``` 或 ```lcm``` 或 ```lcm[100_ots]``` beta_schedule。要完全使用 LCM，请确保使用适当的 LCM lora，在 KSampler 节点中使用 ```lcm``` sampler_name，并将 cfg 降低到 1.0 到 2.0 之间。不要忘记减少步骤（最小值 = ~4 步），因为 LCM 收敛更快（步骤更少）。根据需要增加步骤数以增加细节。
- 支持 [AnimateLCM-I2V](https://huggingface.co/wangfuyun/AnimateLCM-I2V)，非常感谢 [Fu-Yun Wang](https://github.com/G-U-N) 在我撰写论文期间提供的原始 diffusers 代码
     - 注意：需要与上述 AnimateLCM 相同的设置。需要使用 ```Apply AnimateLCM-I2V Model``` Gen2 节点，以便提供 ```ref_latent```；使用 ```Scale Ref Image and VAE Encode``` 节点预处理输入图像。虽然这原本是一个 img2video 模型，但我发现它在 ```ref_drift=0.0``` 的情况下最适合 vid2vid 目的，并且在切换到其他模型之前至少使用 1 步，通过与其他 Apply AnimateDiff Model (Adv.) 节点串联。```apply_ref_when_disabled``` 可以设置为 True，以允许 img_encoder 在达到 ```end_percent``` 时继续工作。AnimateLCM-I2V 在更高分辨率下保持一致性也非常有用（在激活 ControlNet 和 SD LoRAs 的情况下，我可以轻松地从 512x512 源一次性上采样到 1024x1024）。待办事项：添加示例
- 支持 [CameraCtrl](https://github.com/hehao13/CameraCtrl)，您必须在这里使用的修剪模型：[CameraCtrl_pruned.safetensors](https://huggingface.co/Kosinkadink/CameraCtrl/tree/main)
     - 注意：需要 AnimateDiff SD1.5 模型，并且专门为 v3 模型训练。仅限 Gen2，提供了 Gen2/CameraCtrl 子菜单下的辅助节点。
- 支持 [PIA](https://github.com/open-mmlab/PIA)，模型 [pia.ckpt](https://huggingface.co/Leoxing/PIA/tree/main)
     - 注意：您需要使用 ```autoselect``` 或 ```sqrt_linear (AnimateDiff)``` beta_schedule。如果需要实际提供输入图像，需要使用 ```Apply AnimateDiff-PIA Model``` Gen2 节点。```pia_input``` 可以通过论文预设（```PIA Input [Paper Presets]```）或手动输入值（```PIA Input [Multival]```）提供。
- 在采样过程中的不同点更改 Scale 和 Effect 的 AnimateDiff 关键帧。
- 支持 fp8；需要最新的 ComfyUI 和 torch >= 2.1（减少 VRAM 使用，但改变输出）
- 支持 Mac M1/M2/M3
- 通过 Gen2 Use Evolved Sampling 节点在 AnimateDiff 之外使用上下文选项和采样设置
- 通过 LoRA Hooks 支持可掩码和可调度的 SD LoRA（以及模型作为 LoRA）用于 AnimateDiff 和 StableDiffusion
- 每帧 GLIGEN 坐标控制
     - 目前需要 KJNodes 中的 GLIGENTextBoxApplyBatch 来实现这一点，但我很快会添加原生节点来完成这个功能。
- 采样过程中注入图像

## 即将推出的功能
- 为 AnimateDiff-Evolved 仓库中的**每个功能**提供示例工作流程，并希望制作一个长长的 Youtube 视频展示所有功能（目标：在 Elden Ring DLC 发布之前完成。目前正在努力中。）
- 支持 [UniCtrl](https://github.com/XuweiyiChen/UniCtrl)
- 支持 Unet-Ref，以便可以将大量论文移植过来
- 实现 [StoryDiffusion](https://github.com/HVision-NKU/StoryDiffusion)
- 合并运动模型权重/组件，包括每个块的自定义
- 可掩码的 Motion LoRA
- 可调度时间步的 GLIGEN 坐标
- 动态内存管理，用于在不同的 start/end_percents 加载/卸载运动模型
- 内置提示旅行实现
- 任何其他与 AnimateDiff 相关的新功能


# 基本用法和节点

有两种类型的节点可以用于使用 AnimateDiff/Evolved Sampling - **Gen1** 和 **Gen2**。除了特别标记为 Gen1/Gen2 的节点外，所有其他节点都可以用于 Gen1 和 Gen2。

Gen1 和 Gen2 产生完全相同的结果（后端代码相同），唯一的区别在于使用模式的方式。总的来说，Gen1 是使用基本 AnimateDiff 功能的最简单方式，而 Gen2 将模型加载和应用与 Evolved Sampling 功能分开。这意味着在实践中，Gen2 的 Use Evolved Sampling 节点可以在没有模型的情况下使用，允许在不使用 AnimateDiff 的情况下使用上下文选项和采样设置。

在以下文档中，输入/输出将按以下颜色编码：
- 🟩 - 必需输入
- 🟨 - 可选输入
- 🟦 - 开始作为小部件，可以转换为输入
- 🟪 - 输出

## Gen1/Gen2 节点

| ① Gen1 ① | ② Gen2 ② |
|---|---|
| - 一体化节点<br/> - 如果同一个模型被多个 Gen1 节点加载，会导致 RAM 重复使用。 | - 将模型加载与应用和 Evolved Sampling 分离<br/> - 在没有运动模型的情况下启用 Evolved Sampling 功能<br/> - 通过 Apply AnimateDiff Model (Adv.) 节点启用多个运动模型的使用 |
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/a94029fd-5e74-467b-853c-c3ec4cf8a321) | ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/8c050151-6cfb-4350-932d-a105af78a1ec) |
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/c7ae9ef3-b5cd-4800-b249-da2cb73c4c1e) | ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/cffa21f7-0e33-45d1-9950-ad22eb229134) |


### 输入
- 🟩*model*: StableDiffusion (SD) 模型输入。
- 🟦*model_name*: AnimateDiff (AD) 模型，在采样过程中加载和/或应用。某些运动模型适用于 SD1.5，而其他模型适用于 SDXL。
- 🟦*beta_schedule*: 将选定的 beta_schedule 应用于 SD 模型；```autoselect``` 将自动为选定的运动模型选择推荐的 beta_schedule - 或者如果没有为 Gen2 选择运动模型，则使用现有的 beta_schedule。
- 🟨*context_options*: 来自 context_opts 子菜单的 Context Options 节点 - 当需要回到 AnimateDiff 模型的最佳点时应该使用。也可以在没有运动模型的情况下使用（仅限 Gen2）。
- 🟨*sample_settings*: Sample Settings 节点输入 - 用于应用自定义采样选项，如 FreeNoise（noise_type）、FreeInit（iter_opts）、自定义种子、噪声层等。也可以在没有运动模型的情况下使用（仅限 Gen2）。
- 🟨*motion_lora*: 对于基于 v2 的模型，Motion LoRA 将影响生成的运动。只有少数官方 motion LoRAs 被发布 - 很快，我将与其他社区成员合作，创建训练代码来创建（并测试）新的 Motion LoRAs，这些 LoRAs 可能适用于非 v2 模型。
- 🟨*ad_settings*: 在加载过程中修改运动模型，允许调整位置编码器（PEs）以扩展模型的最佳点或修改整体运动。
- 🟨*ad_keyframes*: 允许在采样时间步长上调度 ```scale_multival``` 和 ```effect_multival``` 输入。
- 🟨*scale_multival*: 使用 ```Multival``` 输入（默认为 ```1.0```）。以前称为 motion_scale，它直接影响模型生成的运动量。通过 Multival 节点，它可以接受浮点数、浮点数列表和/或掩码输入，允许不仅对不同帧，而且对帧的不同区域（包括每帧）应用不同的比例。
- 🟨*effect_multival*: 使用 ```Multival``` 输入（默认为 ```1.0```）。确定运动模型对采样过程的影响。值为 ```0.0``` 相当于没有 AnimateDiff 影响的正常 SD 输出。通过 Multival 节点，它可以接受浮点数、浮点数列表和/或掩码输入，允许不仅对不同帧，而且对帧的不同区域（包括每帧）应用不同的效果量。

#### Gen2 独有输入
- 🟨*motion_model*: 加载的运动模型输入。
- 🟨*m_models*: 从 Apply AnimateDiff Model 节点输出的一个（或多个）运动模型。

#### Gen2 Adv. 独有输入
- 🟨*prev_m_models*: 与当前模型一起使用的先前应用的运动模型。
- 🟨*start_percent*: 确定连接的运动模型何时开始生效（优先于任何 ad_keyframes）。
- 🟨*end_percent*: 确定连接的运动模型何时停止生效（优先于任何 ad_keyframes）。

#### Gen1 (Legacy) 输入
- 🟦*motion_scale*: ```scale_multival``` 的旧版本，只能是浮点数。
- 🟦*apply_v2_models_properly*: 向后兼容的切换，适用于几个月前使用未关闭 v2 模型组归一化hack代码的工作流程。**仅影响 v2 模型，其他模型不受影响。** 所有节点现在默认此值为 ```True```。

### 输出
- 🟪*MODEL*: 注入了 Evolved Sampling/AnimateDiff 的 SD 模型。

#### Gen2 独有输出
- 🟪*MOTION_MODEL*: 加载的运动模型。
- 🟪*M_MODELS*: 一个（或多个）应用的运动模型，可以插入 Use Evolved Sampling 或另一个 Apply AnimateDiff Model (Adv.) 节点。


## Multival 节点

对于 Multival 输入，这些节点允许使用浮点数、浮点数列表和/或掩码作为输入。Scaled Mask 节点允许自定义掩码的暗/亮区域，以确定这些区域对应的值。

| 节点 | 输入 |
|---|---|
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/d4c6a63f-703a-402b-989e-ab4d04141c7a) | 🟨*mask_optional*: 用于浮点数值的掩码 - 黑色表示 0.0，白色表示 1.0（乘以 float_val）。 <br/> 🟦*float_val*: 浮点数乘数。|
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/bc100bec-0407-47c8-aebd-f74f2417711e) | 🟩*mask*: 用于浮点数值的掩码。 <br/> 🟦*min_float_val*: 最小值。 <br/>🟦*max_float_val*: 最大值。 <br/> 🟦*scaling*: 当 ```absolute``` 时，黑色表示 min_float_val，白色表示 max_float_val。当 ```relative``` 时，掩码中最暗的区域（总体）表示 min_float_val，掩码中最亮的区域（总体）表示 max_float_val。 |


## AnimateDiff 关键帧

允许为 scale_multival 和 effect_multival 安排时间步长。

两个设置来确定调度是 ***start_percent*** 和 ***guarantee_steps***。当多个关键帧具有相同的 start_percent 时，它们将按照连接的顺序执行，并在移动到下一个节点之前运行 guarantee_steps 时间步长。

| 节点 |
|---|
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/dca73cdc-157a-47db-bed2-6ba584dceccd) |

### 输入
- 🟨*prev_ad_keyframes*: 链式关键帧以创建调度。
- 🟨*scale_multival*: 此关键帧使用的 scale 值。
- 🟨*effect_multival*: 此关键帧使用的 effect 值。
- 🟦*start_percent*: 开始使用此关键帧的时间步长百分比。如果多个关键帧具有相同的 start_percent，执行顺序由它们的链式顺序决定，并将持续 guarantee_steps 时间步长。
- 🟦*guarantee_steps*: 关键帧将使用的最小步数 - 当设置为 0 时，此关键帧仅在当前时间步长没有更好的匹配关键帧时使用。
- 🟦*inherit_missing*: 当设置为 ```True``` 时，任何缺失的 scale_multival 或 effect_multival 输入将继承前一个关键帧的值 - 如果前一个关键帧也继承缺失，将使用最后一个继承的值。


## 上下文选项和视图选项

这些节点提供了扩展动画长度以克服 AnimateDiff 模型（通常为 16 帧）和 HotshotXL 模型（8 帧）最佳点限制的技术。

上下文选项通过一次扩散动画的部分内容，包括主要的 SD 扩散、ControlNets、IPAdapters 等，有效地将 VRAM 使用限制为相当于 context_length 潜在变量的使用。

相比之下，视图选项通过将运动模型看到的潜在变量分段来工作。这并不会减少 VRAM 使用，但通常比上下文选项更稳定和更快，因为潜在变量不需要通过整个 SD unet。

上下文选项和视图选项可以结合使用，以获得两者的最佳效果 - 可以使用更长的 context_length 来获得更稳定的输出，代价是使用更多的 VRAM（因为 context_length 决定了在 GPU 上同时进行多少 SD 采样）。如果你有足够的 VRAM，你也可以使用仅视图的上下文选项，只使用视图选项（并自动使 context_length 等同于完整的潜在变量），以换取更高的 VRAM 使用来获得速度提升。

有两种类型的上下文/视图选项：***标准*** 和 ***循环***。***标准*** 选项不会导致输出循环。***循环*** 选项，顾名思义，会导致输出从结束到开始循环。在代码重构之前，唯一可用的上下文是循环类型。

***我建议在不需要循环输出时首先使用标准静态。***

在下面的动画中，***绿色*** 显示上下文，***红色*** 显示视图。简而言之，绿色是加载到 VRAM（并采样）的潜在变量数量，而红色是每次传递给运动模型的潜在变量数量。

### 上下文选项◆标准静态
| 行为 |
|---|
| ![anim__00005](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/b26792d6-0f41-4f07-93aa-e5ee83f4d90e) <br/> (潜在变量计数: 64, context_length: 16, context_overlap: 4, 总步数: 20)|

| 节点 | 输入 |
|---|---|
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/a4a5f38e-3a1b-4328-9537-ad17567aed75) | 🟦*context_length*: 一次扩散的潜在变量数量。<br/> 🟦*context_overlap*: 相邻窗口之间的最小公共潜在变量。<br/> 🟦*fuse_method*: 窗口结果平均的方法。<br/> 🟦*use_on_equal_length*: 当为 True 时，允许在潜在变量计数匹配 context_length 时使用上下文。<br/> 🟦*start_percent*: 当多个上下文选项链式连接时，允许调度。<br/> 🟦*guarantee_steps*: 当调度上下文时，确定上下文应使用的*最小*采样步数。<br/> 🟦*context_length*: 一次扩散的潜在变量数量。<br/> 🟨*prev_context*: 允许上下文链式连接。<br/> 🟨*view_options*: 当 context_length > view_length（除非另有指定）时，允许在每个上下文窗口内使用视图选项。|

### 上下文选项◆标准均匀
| 行为 |
|---|
| ![anim__00006](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/69707e3d-f49e-4368-89d5-616af2631594) <br/> (潜在变量计数: 64, context_length: 16, context_overlap: 4, context_stride: 1, 总步数: 20) |
| ![anim__00010](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/7fc083b4-406f-4809-94ca-b389784adcab) <br/> (潜在变量计数: 64, context_length: 16, context_overlap: 4, context_stride: 2, 总步数: 20) |

| 节点 | 输入 |
|---|---|
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/c2c8c7ea-66b6-408d-be46-1d805ecd64d1) | 🟦*context_length*: 一次扩散的潜在变量数量。<br/> 🟦*context_overlap*: 相邻窗口之间的最小公共潜在变量。<br/> 🟦*context_stride*: 相邻潜在变量之间的最大 2^(stride-1) 距离。<br/> 🟦*fuse_method*: 窗口结果平均的方法。<br/> 🟦*use_on_equal_length*: 当为 True 时，允许在潜在变量计数匹配 context_length 时使用上下文。<br/> 🟦*start_percent*: 当多个上下文选项链式连接时，允许调度。<br/> 🟦*guarantee_steps*: 当调度上下文时，确定上下文应使用的*最小*采样步数。<br/> 🟦*context_length*: 一次扩散的潜在变量数量。<br/> 🟨*prev_context*: 允许上下文链式连接。<br/> 🟨*view_options*: 当 context_length > view_length（除非另有指定）时，允许在每个上下文窗口内使用视图选项。|

### 上下文选项◆循环均匀
| 行为 |
|---|
| ![anim__00008](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/d08ac1c9-2cec-4c9e-b257-0a804448d41b) <br/> (潜在变量计数: 64, context_length: 16, context_overlap: 4, context_stride: 1, closed_loop: False, 总步数: 20) |
| ![anim__00009](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/61e0311b-b623-423f-bbcb-eb4eb02e9002) <br/> (潜在变量计数: 64, context_length: 16, context_overlap: 4, context_stride: 1, closed_loop: True, 总步数: 20) |

| 节点 | 输入 |
|---|---|
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/c2c8c7ea-66b6-408d-be46-1d805ecd64d1) | 🟦*context_length*: 一次扩散的潜在变量数量。<br/> 🟦*context_overlap*: 相邻窗口之间的最小公共潜在变量。<br/> 🟦*context_stride*: 相邻潜在变量之间的最大 2^(stride-1) 距离。<br/> 🟦*closed_loop*: 当为 True 时，添加额外的窗口以增强循环。<br/> 🟦*fuse_method*: 窗口结果平均的方法。<br/> 🟦*use_on_equal_length*: 当为 True 时，允许在潜在变量计数匹配 context_length 时使用上下文 - 允许在潜在变量计数 == context_length 时创建循环。<br/> 🟦*start_percent*: 当多个上下文选项链式连接时，允许调度。<br/> 🟦*guarantee_steps*: 当调度上下文时，确定上下文应使用的*最小*采样步数。<br/> 🟦*context_length*: 一次扩散的潜在变量数量。<br/> 🟨*prev_context*: 允许上下文链式连接。<br/> 🟨*view_options*: 当 context_length > view_length（除非另有指定）时，允许在每个上下文窗口内使用视图选项。|

### 上下文选项◆仅视图 [VRAM⇈]
| 行为 |
|---|
| ![anim__00011](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/f2e422a4-c894-4e89-8f35-1964b89f369d) <br/> (潜在变量计数: 64, view_length: 16, view_overlap: 4, 视图选项◆标准静态, 总步数: 20) |

| 节点 | 输入 |
|---|---|
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/8cd6a0a4-ee8a-46c3-b04b-a100f87025b3) | 🟩*view_opts_req*: 用于所有潜在变量的视图选项。 <br/> 🟨*prev_context*: 允许上下文链式连接。<br/> |


这些调度有相应的视图选项：

### 视图选项◆标准静态
| 行为 |
|---|
| ![anim__00012](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/7aee4ccb-b669-42fd-a1b5-2005003d5f8d) <br/> (潜在变量计数: 64, view_length: 16, view_overlap: 4, 上下文选项◆标准静态, context_length: 32, context_overlap: 8, 总步数: 20) |

| 节点 | 输入 |
|---|---|
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/4b22c73f-99cb-4781-bd33-e1b3db848207) | 🟦*view_length*: 一次传递给运动模型的上下文中的潜在变量数量。<br/> 🟦*view_overlap*: 相邻窗口之间的最小公共潜在变量。<br/> 🟦*fuse_method*: 窗口结果平均的方法。<br/> |

### 视图选项◆标准均匀
| 行为 |
|---|
| ![anim__00015](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/faa2cd26-9f94-4fce-90b2-8acec84b444e ) <br/> (潜在变量计数: 64, view_length: 16, view_overlap: 4, view_stride: 1, 上下文选项◆标准静态, context_length: 32, context_overlap: 8, 总步数: 20) |

| 节点 | 输入 |
|---|---|
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/bbf017e6-3545-4043-ba41-fcbe2f54496a) | 🟦*view_length*: 一次传递给运动模型的上下文中的潜在变量数量。<br/> 🟦*view_overlap*: 相邻窗口之间的最小公共潜在变量。<br/> 🟦*view_stride*: 相邻潜在变量之间的最大 2^(stride-1) 距离。<br/> 🟦*fuse_method*: 窗口结果平均的方法。<br/> |

### 视图选项◆循环均匀
| 行为 |
|---|
| ![anim__00016](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/8922b44b-cb19-4b2a-8486-2df8a46bf573) <br/> (潜在变量计数: 64, view_length: 16, view_overlap: 4, view_stride: 1, closed_loop: False, 上下文选项◆标准静态, context_length: 32, context_overlap: 8, 总步数: 20) |
| 注意：除非你有特定的原因使用这个，否则这个可能不会看起来很好。 |

| 节点 | 输入 |
|---|---|
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/c58fe4d4-81a8-436b-8028-9e81c2ace18a) | 🟦*view_length*: 一次传递给运动模型的上下文中的潜在变量数量。<br/> 🟦*view_overlap*: 相邻窗口之间的最小公共潜在变量。<br/> 🟦*view_stride*: 相邻潜在变量之间的最大 2^(stride-1) 距离。<br/> 🟦*closed_loop*: 当为 True 时，添加额外的窗口以增强循环。<br/> 🟦*use_on_equal_length*: 当为 True 时，允许在潜在变量计数匹配 context_length 时使用上下文 - 允许在潜在变量计数 == context_length 时创建循环。<br/> 🟦*fuse_method*: 窗口结果平均的方法。<br/> |

## 采样设置

采样设置节点允许自定义采样过程，超出大多数 KSampler 节点暴露的内容。使用其默认值，它不会有任何效果，并且可以安全地附加而不改变任何行为。

简而言之：要使用 FreeNoise，从 noise_type 下拉菜单中选择 ```FreeNoise```。FreeNoise 不会以任何方式降低性能。要使用 FreeInit，将 FreeInit 迭代选项附加到 iteration_opts 输入。注意：FreeInit，尽管它的名字，通过重新采样潜在变量 ```iterations``` 次来工作 - 这意味着如果你使用 iteration=2，总采样时间将是原来的两倍，因为它将执行两次采样。

具有相同名称（或非常接近相同名称）输入的噪声层具有与采样设置相同的预期行为 - 请参阅下面的输入。

| 节点 |
|---|
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/563a13cf-7aed-4acc-9ce3-1556660a34c2) |

### 输入
- 🟨*noise_layers*: 可自定义的、可堆叠的噪声，添加到/修改初始噪声。
- 🟨*iteration_opts*: 确定是否（以及如何）连续重复采样的选项；如果你想查看 FreeInit，这就是如何使用它。
- 🟨*seed_override*: 接受一个整数以使用种子而不是传递给 KSampler 的种子，或一个整数列表（如通过 FizzNodes 的 BatchedValueSchedule）为批次中的每个潜在变量分配单独的种子。
- 🟦*seed_offset*: 当不设置为 0 时，将值添加到当前种子，可预测地改变它，无论原始种子是什么。
- 🟦*batch_offset*: 当不设置为 0 时，将噪声“偏移”，就好像第一个潜在变量实际上是 batch_offset-nth 潜在变量，将所有噪声移动。
- 🟦*noise_type*: 选择要生成的噪声类型。值包括：
   - **default**: 像往常一样为所有潜在变量生成不同的噪声。
   - **constant**: 为所有潜在变量生成完全相同的噪声（基于种子）。
   - **empty**: 为所有潜在变量生成无噪声（就像噪声被关闭一样）。
   - **repeated_context**: 每 context_length（或 view_length）潜在变量重复噪声；稳定较长的生成，但有非常明显的重复。
   - **FreeNoise**: 重复噪声，使其每 context_length（或 view_length）潜在变量重复，但上下文/视图之间的重叠噪声被洗牌，以减少重复，同时仍实现稳定。
- 🟦*seed_gen*: 允许在 ComfyUI 和 Auto1111 方法之间选择噪声生成。两者都不是更好（噪声分布相同），它们只是不同的方法。
   - **comfy**: 根据提供的种子一次性为整个潜在变量批次张量生成噪声。
   - **auto1111**: 为每个潜在变量单独生成噪声，每个潜在变量接收一个增加的 +1 种子偏移（第一个潜在变量使用种子，第二个潜在变量使用种子+1，等等）。
- 🟦*adapt_denoise_steps*: 当为 True 时，带有 'denoise' 输入的 KSamplers 将自动减少总步数，就像 Auto1111 中的默认选项一样。
   - **True**: 步数将随着较低的去噪减少，例如，20 步，0.5 去噪将执行 10 个总步数，但选择的 sigma 仍将实现 0.5 去噪。以速度换取质量（因为采样步数较少）。
   - **False**: 默认行为；20 步，0.5 去噪将执行 20 步。

## 迭代选项

这些选项允许 KSamplers 重新采样相同的潜在变量，而不需要将多个 KSamplers 链接在一起，并且还允许专门的迭代行为来实现诸如 FreeInit 之类的功能。

### 默认迭代选项

简单地重新运行 KSampler，将前一次迭代的结果输入到下一次迭代中。在默认的 iterations=1 情况下，这与没有插入这个节点没有任何区别。

| 节点 | 输入 |
|---|---|
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/23c5e698-6eff-43cc-92e9-488e9b5ca96a) | 🟦*iterations*: KSampler 应该连续运行的总次数。 <br/> 🟦*iter_batch_offset*: 在每次后续迭代中应用的 batch_offset。 <br/> 🟦*iter_seed_offset*: 在每次后续迭代中应用的 seed_offset。 |

### FreeInit 迭代选项

实现了 [FreeInit](https://github.com/TianxingWu/FreeInit)，这个想法是 AnimateDiff 是在现有视频（具有时间一致性的图像）的潜在变量上训练的，这些潜在变量随后被噪声化，而不是从随机的初始噪声开始，并且当对现有潜在变量进行噪声化时，低频数据仍然保留在噪声化的潜在变量中。它将现有视频（或默认行为，前一次迭代）的低频噪声与随机生成噪声中的高频噪声结合起来，运行后续迭代。***每次迭代都是一次完整的采样 - 2 次迭代意味着运行时间将是 1 次迭代/没有迭代选项连接时的两倍。***

当 apply_to_1st_iter 为 False 时，噪声化/低频/高频组合不会在第一次迭代中发生，假设没有有用的潜在变量传递进来进行噪声组合，因此至少需要 2 次迭代才能使 FreeInit 生效。

如果你有一组现有的潜在变量用于获取低频噪声，你可以将 apply_to_1st_iter 设置为 True，然后即使你设置 iterations=1，FreeInit 仍然会生效。

| 节点 |
|---|
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/21404e4f-ab67-44ed-8bf9-e510bc2571de) |

#### 输入
- 🟦*iterations*: KSampler 应该连续运行的总次数。参考上面的解释，为什么默认是 2（以及何时可以设置为 1）。
- 🟦*init_type*: 应用 FreeInit 的代码实现。
   - ***FreeInit [sampler sigma]***: 可能最接近预期的实现，并且从采样器而不是模型获取噪声化的 sigma（如果可能）。
   - ***FreeInit [model sigma]***: 从模型获取噪声化的 sigma；当使用自定义 KSampler 时，这是将用于两个 FreeInit 选项的方法。
   - ***DinkInit_v1***: 我在弄清楚如何准确复制噪声化行为之前的初始、有缺陷的 FreeInit 实现。纯粹通过运气和试错，我设法用这种方法让它实际上有点工作。现在主要是为了向后兼容，但也可能产生有用的结果。

- 🟦*apply_to_1st_iter*: 当设置为 True 时，即使在第一次迭代中也会进行 FreeInit 低频/高频组合工作。参考上面的 FreeInit 迭代选项部分的解释，了解何时可以设置为 True。
- 🟦*init_type*: 应用 FreeInit 的代码实现。
- 🟦*iter_batch_offset*: 在每次后续迭代中应用的 batch_offset。
- 🟦*iter_seed_offset*: 在每次后续迭代中应用的 seed_offset。默认值为 1，以便每次迭代使用新的随机噪声。

- 🟦*filter*: 确定应用于噪声的低频滤波器。非常技术性，查看代码/在线资源以了解各个滤波器的作用。
- 🟦*d_s*: 滤波器的空间参数（在潜在变量内，我认为）；非常技术性。如果你想知道它到底做了什么，查看代码/在线资源。
- 🟦*d_t*: 滤波器的时间参数（跨潜在变量，我认为）；非常技术性。如果你想知道它到底做了什么，查看代码/在线资源。
- 🟦*n_butterworth*: 仅适用于 ```butterworth``` 滤波器；非常技术性。如果你想知道它到底做了什么，查看代码/在线资源。
- 🟦*sigma_step*: 噪声化潜在变量以获取低频噪声时使用的/模拟的噪声化步骤。999 实际上意味着最后一个（-1），任何小于 999 的数字意味着距离最后一个的距离。除非你知道你想用它做什么，否则保持为 999。

## 噪声层

这些节点允许初始噪声被添加、加权或替换。在不久的将来，我将添加能力，使掩码相对于掩码的运动“移动”噪声，而不是仅仅“剪切和粘贴”噪声。

与采样设置共享的输入具有完全相同的效果 - 唯一的新的选项是 seed_gen_override，默认情况下将使用与采样设置相同的 seed_gen（使用现有的）。你可以随意让噪声层使用不同的 seed_gen 策略，或者使用不同的种子/种子集等。

```mask_optional``` 参数决定了噪声层应该应用于初始噪声的哪个位置。

| 节点 | 行为 + 输入 |
|---|---|
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/66487969-669d-47d3-9742-85ae26606903) | [Add]; 直接在顶部添加噪声。 <br/> 🟦*noise_weight*: 在添加到顶部之前噪声层的乘数。 |
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/52acb25c-9116-4594-b3fb-01b7b15bb79d) | [Add Weighted]; 添加噪声，但在已经存在的噪声和自身之间取加权平均。 <br/> 🟦*noise_weight*: 新噪声在现有噪声的加权平均中的权重。 <br/> 🟦*balance_multipler*: 噪声权重影响现有噪声的尺度；1.0 意味着正常的加权平均，低于 1.0 将减少加权减少的量（例如，如果 balance_multiplier 设置为 0.5 且 noise_weight 为 0.25，现有噪声只会减少 0.125 而不是 0.25，但新噪声将以未修改的 0.25 权重添加）。 |
| ![image](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/4feb586e-9920-4f35-8f92-e2e36fabb2df) | [Replace]; 直接用自身替换下面层的现有噪声。 |


# 示例（下载或拖动工作流程的图像到 ComfyUI 中以立即加载相应的工作流程！）

注意：我已经将 gif 缩小到 0.75 倍大小，以使它们在 README 中占用更少的空间。

### txt2img

| 结果 |
|---|
| ![readme_00006](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/b615a4aa-db3e-4b24-b88f-b694e52f6364) |
| 工作流程 |
| ![t2i_wf](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/6eb47506-b503-482b-9baf-4c238f30a9c2)   |

### txt2img - (提示旅行)

| 结果 |
|---|
| ![readme_00010](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/c27a2029-2c69-4272-b40f-64408e9e2ea6) |
| 工作流程 |
| ![t2i_prompttravel_wf](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/e5a72ea1-628d-423e-98ed-f20e1bcc5320) |



### txt2img - 48 帧动画，16 context_length（上下文选项◆标准静态）+ FreeNoise

| 结果 |
|---|
| ![readme_00012](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/684f6e79-d653-482f-899a-1900dc56cd8f) |
| 工作流程 |
| ![t2i_context_freenoise_wf](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/9d0e53fa-49d6-483d-a660-3f41d7451002) |


# 旧示例（TODO：当我得到休息时更新所有这些 + 添加新的）

### txt2img - 32 帧动画，16 context_length（均匀）- PanLeft 和 ZoomOut 运动 LoRAs

![t2i_context_mlora_wf](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/41ec4141-389c-4ef4-ae3e-a963a0fa841f)

![aaa_readme_00094_](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/14abee9a-5500-4d14-8632-15ac77ba5709)

[aaa_readme_00095_.webm](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/d730ae2e-188c-4a61-8a6d-bd48f60a2d07)


### txt2img w/ 潜在上采样（上采样时部分去噪）

![t2i_lat_ups_wf](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/521991dd-8e39-4fed-9970-514507c75067)

![aaa_readme_up_00001_](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/f4199e25-c839-41ed-8986-fb7dbbe2ac52)

[aaa_readme_up_00002_.webm](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/2f44342f-3fd8-4863-8e3d-360377d608b7)



### 使用潜在放大（放大时部分去噪）的txt2img - PanLeft和ZoomOut运动LoRAs

![t2i_mlora_lat_ups_wf](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/f34882de-7dd4-4264-8f59-e24da350be2a)

![aaa_readme_up_00023_](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/e2ca5c0c-b5d9-42de-b877-4ed29db81eb9)

[aaa_readme_up_00024_.webm](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/414c16d8-231c-422f-8dfc-a93d4b68ffcc)



### 使用潜在放大（放大时部分去噪）的txt2img - 48帧动画，16个上下文长度（均匀）

![t2i_lat_ups_full_wf](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/a1ebc14e-853e-4cda-9cda-9a7553fa3d85)

[aaa_readme_up_00009_.webm](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/f7a45f81-e700-4bfe-9fdd-fbcaa4fa8a4e)



### 使用潜在放大（放大时完全去噪）的txt2img

![t2i_lat_ups_full_wf](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/5058f201-3f52-4c48-ac7e-525c3c8f3df3)

![aaa_readme_up_00010_](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/804610de-18ec-43af-9af2-4a83cf31d16b)

[aaa_readme_up_00012_.webm](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/3eb575cf-92dd-434a-b3db-1a2064ff0033)



### 使用潜在放大（放大时完全去噪）的txt2img - 48帧动画，16个上下文长度（均匀）

![t2i_context_lat_ups_wf](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/7b9ec22b-d4e0-4083-9846-5743ed90583e)

[aaa_readme_up_00014_.webm](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/034aff4c-f814-4b87-b5d1-407b1089af0d)



### 使用ControlNet稳定潜在放大（放大时部分去噪，缩放软控制网权重）的txt2img

![t2i_lat_ups_softcontrol_wf](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/c769c2bd-5aac-48d0-92b7-d73c422d4863)

![aaa_readme_up_00017_](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/221954cc-95df-4e0c-8ec9-266d0108dad4)

[aaa_readme_up_00019_.webm](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/b562251d-a4fb-4141-94dd-9f8bca9f3ce8)



### 使用ControlNet稳定潜在放大（放大时部分去噪，缩放软控制网权重）的txt2img - 48帧动画，16个上下文长度（均匀）

![t2i_context_lat_ups_softcontrol_wf](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/798567a8-4ef0-4814-aeeb-4f770df8d783)

[aaa_readme_up_00003_.webm](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/0f57c949-0af3-4da4-b7c4-5c1fb1549927)



### 使用初始ControlNet输入的txt2img（以在第一个txt2img上使用Normal LineArt预处理器为例）

![t2i_initcn_wf](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/caa7abdf-7ba0-456c-9fa4-547944ea6e72)

![aaa_readme_cn_00002_](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/055ef87c-50c6-4bb9-b35e-dd97916b47cc)

[aaa_readme_cn_00003_.webm](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/9c9d425d-2378-4af0-8464-2c6c0d1a68bf)



### 使用初始ControlNet输入的txt2img（以在第一个txt2img上使用Normal LineArt预处理器为例）- 48帧动画，16个上下文长度（均匀）

![t2i_context_initcn_wf](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/f9de2711-dcfd-4fea-8b3b-31e3794fbff9)

![aaa_readme_cn_00005_](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/6bf14361-5b09-4305-b2a7-f7babad4bd14)

[aaa_readme_cn_00006_.webm](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/5d3665b7-c2da-46a1-88d8-ab43ba8eb0c6)



### 使用初始ControlNet输入（使用OpenPose图像）+ 潜在放大与完全去噪的txt2img

![t2i_openpose_upscale_wf](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/306a40c4-0591-496d-a320-c33f0fc4b3d2)

(使用toyxyz提供的OpenPose图像)

![AA_openpose_cn_gif_00001_](https://github.com/Kosinkadink/ComfyUI-AnimateDiff/assets/7365912/23291941-864d-495a-8ba8-d02e05756396)

![aaa_readme_cn_00032_](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/621a2ca6-2f08-4ed1-96ad-8e6635303173)

[aaa_readme_cn_00033_.webm](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/c5df09a5-8c64-4811-9ecf-57ac73d82377)



### 使用初始ControlNet输入（使用OpenPose图像）+ 潜在放大与完全去噪的txt2img - 48帧动画，16个上下文长度（均匀）

![t2i_context_openpose_upscale_wf](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/a931af6f-bf6a-40d3-bd55-1d7bad32e665)

(使用toyxyz提供的OpenPose图像)

![aaa_readme_preview_00002_](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/028a1e9e-37b5-477d-8665-0e8723306d65)

[aaa_readme_cn_00024_.webm](https://github.com/Kosinkadink/ComfyUI-AnimateDiff-Evolved/assets/7365912/8f4c840c-06a2-4c64-b97e-568dd5ff6f46)



### img2img

TODO: 填充一些有用的方法，有些使用ControlNet Tile。很抱歉目前这里什么都没有，我有很多代码要写。我会尝试逐步填充这部分内容，包括高级ControlNet的使用。



## 已知问题

### 某些运动模型在生成的图像上有可见的水印（尤其是在使用mm_sd_v15时）

AnimateDiff论文作者使用的训练数据包含Shutterstock水印。由于mm_sd_v15是在更精细、不那么剧烈的运动上进行微调的，运动模块试图复制该水印的透明度，并且不像mm_sd_v14那样被模糊掉。使用其他运动模块，或者使用高级KSamplers的组合，应该可以缓解水印问题。
