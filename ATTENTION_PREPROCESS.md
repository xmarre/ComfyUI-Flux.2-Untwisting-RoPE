# Composable H3 attention preprocessing

The H3 attention override exposes `attention_preprocess_v1 = (transform, previous_provider)` for optional numerical backends such as [ComfyUI-Sol-H3 v0.1.0](https://github.com/xmarre/ComfyUI-Sol-H3/releases/tag/v0.1.0).

`transform(q, k, v, heads, **kwargs)` is pure with respect to its inputs and returns Q/K/V with unchanged shape, dtype and device. It performs the same reference-key scaling as ordinary Untwist execution. The inherited provider is the next operation in the chain, or None for Comfy's default.

A sparse consumer applies transforms exactly once, then dispatches sparse QKV and dense-required rows through the remaining dense provider. Calling the complete transform override again on transformed prefix keys would incorrectly scale them twice. Consumers without this optional protocol keep the existing override behavior unchanged.

CPU suite: 40 passed. The combined Sol-H3 v0.1.0 production stack subsequently passed real SM120 GPU execution with Untwist preprocessing consumed exactly once and Spectrum retaining a 14-actual / 4-forecast schedule over 18 logical calls.
