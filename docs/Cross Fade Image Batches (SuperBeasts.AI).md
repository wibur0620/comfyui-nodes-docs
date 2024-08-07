
# Documentation
- Class name: Cross Fade Image Batches (SuperBeasts.AI)
- Category: SuperBeastsAI/Animation
- Output node: False
- Repo Ref: https://github.com/superbeasts/SuperBeastsAI

Cross Fade Image Batches (SuperBeasts.AI)节点专门用于在指定的帧数内将两组图像序列（批次）通过交叉淡入淡出效果混合在一起。该节点能够创建图像序列之间的平滑过渡，通过根据指定的缓动函数和淡化强度对第一批次末尾和第二批次开头的图像像素值进行插值计算来实现。

# Input types
## Required
- batch1
    - 第一批要进行交叉淡化的图像。它代表了过渡效果的起始序列。
    - Comfy dtype: IMAGE
    - Python dtype: torch.Tensor
- batch2
    - 第二批要进行交叉淡化的图像。它代表了过渡效果的结束序列。
    - Comfy dtype: IMAGE
    - Python dtype: torch.Tensor
- fade_frames
    - 指定交叉淡化效果应该持续的帧数。这决定了过渡效果的持续时长。
    - Comfy dtype: INT
    - Python dtype: int
- fade_strength
    - 淡化效果的强度，控制两个批次的图像混合程度。
    - Comfy dtype: FLOAT
    - Python dtype: float
- ease_type
    - 用于淡化效果的缓动函数类型，如线性（linear）、缓入（ease_in）、缓出（ease_out）或缓入缓出（ease_in_out），这会影响过渡的平滑度。
    - Comfy dtype: COMBO[STRING]
    - Python dtype: str

# Output types
- image
    - 应用交叉淡化效果后得到的图像批次，包括未淡化、已淡化和剩余的帧。
    - Comfy dtype: IMAGE
    - Python dtype: torch.Tensor


## Usage tips
- Infra type: `GPU`
- Common nodes: unknown


## Source code
```python
class CrossFadeImageBatches:
    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "batch1": ("IMAGE",),
                "batch2": ("IMAGE",),
                "fade_frames": ("INT", {"default": 10, "min": 1, "max": 100, "step": 1}),
                "fade_strength": ("FLOAT", {"default": 0.5, "min": 0.0, "max": 1.0, "step": 0.01}),
                "ease_type": (["linear", "ease_in", "ease_out", "ease_in_out"], {"default": "linear"}),
            }
        }

    RETURN_TYPES = ("IMAGE",)
    FUNCTION = "crossfade"
    CATEGORY = "SuperBeastsAI/Animation"

    def crossfade(self, batch1, batch2, fade_frames=10, fade_strength=0.5, ease_type="linear"):
        num_frames_batch1 = batch1.shape[0]
        num_frames_batch2 = batch2.shape[0]

        if num_frames_batch1 < fade_frames or num_frames_batch2 < fade_frames:
            raise ValueError("The number of fade frames must be less than or equal to the number of frames in each batch.")

        if batch1.shape[2:] != batch2.shape[2:]:
            raise ValueError("The spatial dimensions of batch1 and batch2 must match. Please resize the batches to the same size before using this node.")

        # Extract the frames to be faded from each batch
        fade_frames_batch1 = batch1[-fade_frames:]
        fade_frames_batch2 = batch2[:fade_frames]

        # Generate the interpolation weights based on the selected easing function
        if ease_type == "linear":
            weights = torch.linspace(0, 1, fade_frames)
        elif ease_type == "ease_in":
            weights = torch.pow(torch.linspace(0, 1, fade_frames), 2)
        elif ease_type == "ease_out":
            weights = 1 - torch.pow(torch.linspace(1, 0, fade_frames), 2)
        elif ease_type == "ease_in_out":
            weights = 0.5 * (1 - torch.cos(torch.linspace(0, torch.pi, fade_frames)))

        weights = weights.view(-1, 1, 1, 1)  # Reshape weights to match the dimensions of the frames

        # Perform the cross-fade between the fade frames of each batch
        faded_frames = fade_frames_batch1 * (1 - weights * fade_strength) + fade_frames_batch2 * (weights * fade_strength)

        # Combine the non-faded frames from batch1, the faded frames, and the remaining frames from batch2
        output_batch = torch.cat((batch1[:-fade_frames], faded_frames, batch2[fade_frames:]), dim=0)

        return (output_batch,)

```
