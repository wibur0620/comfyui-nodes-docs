
# Documentation
- Class name: Empty Images
- Category: List Stuff
- Output node: True
- Repo Ref: https://github.com/comfyanonymous/ComfyUI

Empty Images 节点设计用于生成一批具有指定尺寸和颜色的空白图像。它作为一个基础工具，用于在各种图像处理和生成任务中创建占位符或背景图像。

# Input types
## Optional
- num_images
    - 指定要生成的图像总数。该参数可用于在生成图像时控制整体输出数量。
    - Comfy dtype: INT
    - Python dtype: Optional[List[int]]
- splits
    - 定义如何将图像总数划分为单独的批次或组。这可用于在整体批次内创建不同的图像集。
    - Comfy dtype: INT
    - Python dtype: Optional[List[int]]
- batch_size
    - 确定单个批次中要生成的图像数量。这允许一次创建多个图像，从而优化处理时间。
    - Comfy dtype: INT
    - Python dtype: int

# Output types
- Image
    - 输出是一批具有指定尺寸和颜色的图像。
    - Comfy dtype: IMAGE
    - Python dtype: List[torch.Tensor]


## Usage tips
- Infra type: `GPU`
- Common nodes: unknown


## Source code
```python
class EmptyImages:
    def __init__(self) -> None:
        pass

    @classmethod
    def INPUT_TYPES(s) -> Dict[str, Dict[str, Any]]:
        return {
            "required": {},
            "optional": {
                "num_images": ("INT", {"forceInput": True, "min": 1}),
                "splits": ("INT", {"forceInput": True, "min": 1}),
                "batch_size": ("INT", {"default": 1, "min": 1}),
            }
        }

    RELOAD_INST = True
    RETURN_TYPES = ("IMAGE",)
    RETURN_NAMES = ("Image",)
    INPUT_IS_LIST = (True,)
    OUTPUT_IS_LIST = (True,)
    OUTPUT_NODE = True
    FUNCTION = "generate_empty_images"

    CATEGORY = "List Stuff"

    def generate_empty_images(
            self,
            num_images: Optional[List[int]] = None,
            splits: Optional[List[int]] = None,
            batch_size: Optional[List[int]] = None,
    ) -> Tuple[List[Tensor]]:
        if batch_size is None:
            batch_size = [1]
        else:
            if len(batch_size) != 1:
                raise Exception("Only single batch size supported.")

        if num_images is None and splits is None:
            raise Exception("Must provide either num_images or splits.")

        if num_images is not None and len(num_images) != 1:
            raise Exception("Only single num_images supported.")

        if num_images is not None and splits is None:
            # If splits is None, then all images are in one split
            splits = [num_images[0]]

        if num_images is None and splits is not None:
            # If num_images is None, then it should be the sum of all splits
            num_images = [sum(splits)]

        if num_images is not None and splits is not None:
            if len(splits) == 1:
                # Fill splits with same value enough times to sum to num_images
                fills = int(num_images[0] / splits[0])
                splits = [splits[0]] * fills
                if sum(splits) != num_images[0]:
                    splits.append(num_images[0] - sum(splits))
            else:
                if sum(splits) != num_images[0]:
                    raise Exception("Sum of splits must match number of images.")

        if splits is None:
            raise ValueError("Unexpected error: Splits is None")

        ret_images: List[Tensor] = []
        for split_idx, split in enumerate(splits):
            # Rotate between fully dynamic range of colors
            base_color = (
                50 + (split_idx * 45) % 200,  # Cycle between 50 and 250
                30 + (split_idx * 75) % 200,
                10 + (split_idx * 105) % 200,
            )
            print(f"Splits: {split} | Base Color: {base_color}")

            for _ in range(split):
                batch_tensor = torch.zeros(batch_size[0], 512, 512, 3)
                for batch_idx in range(batch_size[0]):
                    batch_color = (
                        (base_color[0] + int(((255 - base_color[0]) / batch_size[0]) * batch_idx)),
                        (base_color[1] + int(((255 - base_color[1]) / batch_size[0]) * batch_idx)),
                        (base_color[2] + int(((255 - base_color[2]) / batch_size[0]) * batch_idx)),
                    )
                    image = Image.new("RGB", (512, 512), color=batch_color)
                    batch_tensor[batch_idx] = pil2tensor(image)
                ret_images.append(batch_tensor)
        return (ret_images,)

```
