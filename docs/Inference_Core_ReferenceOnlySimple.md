
# Documentation
- Class name: Inference_Core_ReferenceOnlySimple
- Category: custom_node_experiments
- Output node: False

该节点旨在作为ComfyUI推理核心框架中的参考实现。它提供了一个简化示例，展示了如何构建和实现推理节点，重点演示核心概念和实践，而不涉及功能齐全节点的复杂性。

# Input types
## Required
- model
    - 指定用于推理的模型，作为节点处理的主要输入。
    - Comfy dtype: MODEL
    - Python dtype: torch.nn.Module
- reference
    - 为推理过程提供参考或上下文，辅助输出的生成或转换。
    - Comfy dtype: LATENT
    - Python dtype: Dict[str, torch.Tensor]
- batch_size
    - 确定单个批次中要处理的项目数量，影响推理的吞吐量和性能。
    - Comfy dtype: INT
    - Python dtype: int

# Output types
- model
    - 表示推理后的处理模型数据，以结构化格式封装结果。
    - Comfy dtype: MODEL
    - Python dtype: torch.nn.Module
- latent
    - 包含从推理过程中派生的潜在表示，基于模型输出提供洞察或修改。
    - Comfy dtype: LATENT
    - Python dtype: Dict[str, torch.Tensor]


## Usage tips
- Infra type: `CPU`
- Common nodes: unknown


## Source code
```python
class ReferenceOnlySimple:
    @classmethod
    def INPUT_TYPES(s):
        return {"required": { "model": ("MODEL",),
                              "reference": ("LATENT",),
                              "batch_size": ("INT", {"default": 1, "min": 1, "max": 64})
                              }}

    RETURN_TYPES = ("MODEL", "LATENT")
    FUNCTION = "reference_only"

    CATEGORY = "custom_node_experiments"

    def reference_only(self, model, reference, batch_size):
        model_reference = model.clone()
        size_latent = list(reference["samples"].shape)
        size_latent[0] = batch_size
        latent = {}
        latent["samples"] = torch.zeros(size_latent)

        batch = latent["samples"].shape[0] + reference["samples"].shape[0]
        def reference_apply(q, k, v, extra_options):
            k = k.clone().repeat(1, 2, 1)
            offset = 0
            if q.shape[0] > batch:
                offset = batch

            for o in range(0, q.shape[0], batch):
                for x in range(1, batch):
                    k[x + o, q.shape[1]:] = q[o,:]

            return q, k, k

        model_reference.set_model_attn1_patch(reference_apply)
        out_latent = torch.cat((reference["samples"], latent["samples"]))
        if "noise_mask" in latent:
            mask = latent["noise_mask"]
        else:
            mask = torch.ones((64,64), dtype=torch.float32, device="cpu")

        if len(mask.shape) < 3:
            mask = mask.unsqueeze(0)
        if mask.shape[0] < latent["samples"].shape[0]:
            print(latent["samples"].shape, mask.shape)
            mask = mask.repeat(latent["samples"].shape[0], 1, 1)

        out_mask = torch.zeros((1,mask.shape[1],mask.shape[2]), dtype=torch.float32, device="cpu")
        return (model_reference, {"samples": out_latent, "noise_mask": torch.cat((out_mask, mask))})

```
