# Documentation
- Class name: KSamplerSeq2
- Category: sampling
- Output node: False
- Repo Ref: https://github.com/WASasquatch/WAS_Extras

KSamplerSeq2节点旨在促进生成模型中高级采样技术的应用。它管理种子生成和操作的复杂性，允许使用各种采样策略，如增量、递减、随机和固定种子模式。该节点还结合了条件化、潜在插值和噪声注入来微调采样过程，最终生成高质量的潜在表示。

# Input types
## Required
- model
    - 模型参数对于KSamplerSeq2节点至关重要，因为它定义了将用于采样的生成模型。模型的选择对生成样本的质量和类型有显著影响。
    - Comfy dtype: MODEL
    - Python dtype: torch.nn.Module
- seed
    - 种子参数对于初始化随机数生成过程至关重要，确保了采样程序的可重复性。它为基于种子的采样方法设置了起始点。
    - Comfy dtype: INT
    - Python dtype: int
## Optional
- seed_mode_seq
    - seed_mode_seq参数决定了在整个采样序列中种子值是如何调整或选择的，影响生成样本的多样性和连贯性。
    - Comfy dtype: COMBO['increment', 'decrement', 'random', 'fixed']
    - Python dtype: str

# Output types
- samples
    - KSamplerSeq2节点的samples输出代表生成的潜在向量集合。这些向量包含了采样数据的编码信息，可供进一步处理或分析。
    - Comfy dtype: LATENT
    - Python dtype: torch.Tensor

# Usage tips
- Infra type: GPU

# Source code
```
class KSamplerSeq2:

    def __init__(self):
        self.previous_seed = None
        self.current_seed = None

    def initialize_seeds(self, initial_seed):
        self.previous_seed = initial_seed
        self.current_seed = initial_seed

    @classmethod
    def INPUT_TYPES(s):
        return {'required': {'model': ('MODEL',), 'seed': ('INT', {'default': 0, 'min': 0, 'max': 18446744073709551615}), 'seed_mode_seq': (['increment', 'decrement', 'random', 'fixed'],), 'alternate_values': ('BOOLEAN', {'default': True}), 'steps': ('INT', {'default': 20, 'min': 1, 'max': 10000}), 'cfg': ('FLOAT', {'default': 8.0, 'min': 0.0, 'max': 100.0, 'step': 0.5, 'round': 0.01}), 'sampler_name': (comfy.samplers.KSampler.SAMPLERS,), 'scheduler': (comfy.samplers.KSampler.SCHEDULERS,), 'frame_count': ('INT', {'default': 0, 'min': 0, 'max': 1024, 'step': 1}), 'cond_keyframes': ('INT', {'default': 0, 'min': 0, 'max': 1024, 'step': 1}), 'positive_seq': ('CONDITIONING',), 'negative_seq': ('CONDITIONING',), 'use_conditioning_slerp': ('BOOLEAN', {'default': False}), 'cond_slerp_strength': ('FLOAT', {'default': 0.5, 'min': 0.0, 'max': 1.0, 'step': 0.001}), 'latent_image': ('LATENT',), 'use_latent_interpolation': ('BOOLEAN', {'default': False}), 'latent_interpolation_mode': (['Blend', 'Slerp', 'Cosine Interp'],), 'latent_interp_strength': ('FLOAT', {'default': 0.5, 'min': 0.0, 'max': 1.0, 'step': 0.001}), 'denoise_start': ('FLOAT', {'default': 1.0, 'min': 0.0, 'max': 1.0, 'step': 0.01}), 'denoise_seq': ('FLOAT', {'default': 0.5, 'min': 0.0, 'max': 1.0, 'step': 0.01}), 'unsample_latents': ('BOOLEAN', {'default': False}), 'inject_noise': ('BOOLEAN', {'default': True}), 'noise_strength': ('FLOAT', {'default': 0.1, 'max': 1.0, 'min': 0.001, 'step': 0.001}), 'denoise_sine': ('BOOLEAN', {'default': True}), 'denoise_max': ('FLOAT', {'default': 0.9, 'max': 1.0, 'min': 0.0, 'step': 0.001}), 'seed_keying': ('BOOLEAN', {'default': True}), 'seed_keying_mode': (['sine', 'modulo'],), 'seed_divisor': ('INT', {'default': 4, 'max': 1024, 'min': 2, 'step': 1})}}
    RETURN_TYPES = ('LATENT',)
    FUNCTION = 'sample'
    CATEGORY = 'sampling'

    def update_seed(self, seed, seed_mode):
        if seed_mode == 'increment':
            return seed + 1
        elif seed_mode == 'decrement':
            return seed - 1
        elif seed_mode == 'random':
            return random.randint(0, 18446744073709551615)
        elif seed_mode == 'fixed':
            return seed

    def alternate_seed_modulo(self, current, seed, divisor):
        if current % divisor == 0:
            new_seed = (seed + current) % 18446744073709551615
        else:
            new_seed = seed
        return new_seed

    def alternate_seed_sine(self, current, start_seed, divisor):
        seed = 1000 * np.sin(2 * math.pi * current / divisor) + start_seed
        return seed

    def alternate_denoise(self, curent, total, start_denoise=0.5, max_denoise=0.95):
        amplitude = (max_denoise - start_denoise) / 2
        mid_point = (max_denoise + start_denoise) / 2
        cycle_position = math.pi * 2 * curent / total
        current_denoise = amplitude * math.sin(cycle_position) + mid_point
        return current_denoise

    def inject_noise(self, latent_image, noise_strength):
        noise = torch.randn_like(latent_image) * noise_strength
        return latent_image + noise

    def sample(self, model, seed, seed_mode_seq, alternate_values, steps, cfg, sampler_name, scheduler, frame_count, cond_keyframes, positive_seq, negative_seq, cond_slerp_strength, latent_image, use_latent_interpolation, latent_interpolation_mode, latent_interp_strength, denoise_start=1.0, denoise_seq=0.5, use_conditioning_slerp=False, unsample_latents=False, alternate_mode=False, inject_noise=True, noise_strength=0.1, denoise_sine=True, denoise_max=0.9, seed_keying=True, seed_keying_mode='sine', seed_divisor=4):
        if not isinstance(positive_seq, list):
            positive_seq = [positive_seq]
        if not isinstance(negative_seq, list):
            negative_seq = [negative_seq]
        if not isinstance(cond_keyframes, list):
            cond_keyframes = [cond_keyframes]
        cond_keyframes.sort()
        positive_cond_idx = 0
        negative_cond_idx = 0
        results = []
        self.initialize_seeds(seed)
        sequence_loop_count = max(frame_count, len(positive_seq)) if cond_keyframes else len(positive_seq)
        print(f'Starting loop sequence with {sequence_loop_count} frames.')
        print(f'Using {len(positive_seq)} positive conditionings and {len(negative_seq)} negative conditionings')
        print(f"Conditioning keyframe schedule is: {', '.join(map(str, cond_keyframes))}")
        for loop_count in range(sequence_loop_count):
            if loop_count in cond_keyframes:
                positive_cond_idx = min(positive_cond_idx + 1, len(positive_seq) - 1)
                negative_cond_idx = min(negative_cond_idx + 1, len(negative_seq) - 1)
            positive_conditioning = positive_seq[positive_cond_idx]
            negative_conditioning = negative_seq[negative_cond_idx]
            if seed_keying:
                if seed_keying_mode == 'sine':
                    seq_seed = seed if loop_count <= 0 else self.alternate_seed_sine(loop_count, seed, seed_divisor)
                else:
                    seq_seed = seed if loop_count <= 0 else self.alternate_seed_modulo(loop_count, seed, seed_divisor)
            else:
                seq_seed = seed if loop_count <= 0 else self.update_seed(seq_seed, seed_mode_seq)
            seq_seed = seed if loop_count <= 0 else self.update_seed(seq_seed, seed_mode_seq)
            print(f'Loop count: {loop_count}, Seed: {seq_seed}')
            last_positive_conditioning = positive_conditioning if positive_conditioning else None
            last_negative_conditioning = negative_conditioning if negative_conditioning else None
            if use_conditioning_slerp and (last_positive_conditioning and last_negative_conditioning):
                (a, b) = (last_positive_conditioning[0].clone(), positive_conditioning[0].clone())
                (na, nb) = (last_negative_conditioning[0].clone(), negative_conditioning[0].clone())
                (pa, pb) = (last_positive_conditioning[1]['pooled_output'].clone(), positive_conditioning[1]['pooled_output'].clone())
                (npa, npb) = (last_negative_conditioning[1]['pooled_output'].clone(), negative_conditioning[1]['pooled_output'].clone())
                pos_cond = slerp(cond_slerp_strength, a, b)
                pos_pooled = slerp(cond_slerp_strength, pa, pb)
                neg_cond = slerp(cond_slerp_strength, na, nb)
                neg_pooled = slerp(cond_slerp_strength, npa, npb)
                positive_conditioning = [pos_cond, {'pooled_output': pos_pooled}]
                negative_conditioning = [neg_cond, {'pooled_output': neg_pooled}]
            positive_conditioning = [positive_conditioning]
            negative_conditioning = [negative_conditioning]
            end_at_step = steps
            if results and len(results) > 0:
                latent_input = {'samples': results[-1]}
                denoise = self.alternate_denoise(loop_count, sequence_loop_count, denoise_seq, denoise_max) if denoise_sine else denoise_seq
                start_at_step = round((1 - denoise) * steps)
                end_at_step = steps
            else:
                latent_input = latent_image
                denoise = denoise_start
            if unsample_latents and loop_count > 0:
                force_full_denoise = not (loop_count > 0 or loop_count <= steps - 1)
                disable_noise = False
                if seed_keying:
                    if seed_keying_mode == 'modulo' and loop_count % seed_divisor == 0:
                        unsampled_latent = latent_input
                    else:
                        unsampled_latent = unsample(model=model, seed=seq_seed, cfg=cfg, sampler_name=sampler_name, steps=steps, end_at_step=end_at_step, scheduler=scheduler, normalize=False, positive=positive_conditioning, negative=negative_conditioning, latent_image=latent_input)[0]
                else:
                    unsampled_latent = unsample(model=model, seed=seq_seed, cfg=cfg, sampler_name=sampler_name, steps=steps, end_at_step=end_at_step, scheduler=scheduler, normalize=False, positive=positive_conditioning, negative=negative_conditioning, latent_image=latent_input)[0]
                if inject_noise and loop_count > 0:
                    print(f'Injecting noise at {noise_strength} strength.')
                    unsampled_latent['samples'] = self.inject_noise(unsampled_latent['samples'], noise_strength)
                sample = nodes.common_ksampler(model, seq_seed, steps, cfg, sampler_name, scheduler, positive_conditioning, negative_conditioning, unsampled_latent, denoise=denoise, disable_noise=disable_noise, start_step=start_at_step, last_step=end_at_step, force_full_denoise=force_full_denoise)[0]['samples']
            else:
                if inject_noise and loop_count > 0:
                    print(f'Injecting noise at {noise_strength} strength.')
                    latent_input['samples'] = self.inject_noise(latent_input['samples'], noise_strength)
                sample = nodes.common_ksampler(model, seq_seed, steps, cfg, sampler_name, scheduler, positive_conditioning, negative_conditioning, latent_input, denoise=denoise)[0]['samples']
            if use_latent_interpolation and results and (loop_count > 0):
                if latent_interpolation_mode == 'Blend':
                    sample = blend_latents(latent_interp_strength, results[-1], sample)
                elif latent_interpolation_mode == 'Slerp':
                    sample = slerp_latents(latent_interp_strength, results[-1], sample)
                elif latent_interpolation_mode == 'Cosine Interp':
                    sample = cosine_interp_latents(latent_interp_strength, results[-1], sample)
            results.append(sample)
        results = torch.cat(results, dim=0)
        results = {'samples': results}
        return (results,)
```