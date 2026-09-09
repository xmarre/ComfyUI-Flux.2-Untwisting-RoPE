# ComfyUI Untwisting RoPE v0.2.4

v0.2.4 makes MiniMax-H3 attention preprocessing explicitly composable with numerical attention providers and records the completed Sol-H3 production validation.

## Composable H3 preprocessing contract

- Exposes `attention_preprocess_v1 = (transform, previous_provider)` from the existing H3 attention override.
- `transform(q, k, v, heads, **kwargs)` applies Untwist's existing reference-key scaling without changing tensor shape, dtype or device.
- Sparse or specialized providers can consume that preprocessing exactly once, then dispatch through their own numerical backend while retaining the inherited dense provider for dense-required work.
- Ordinary dense execution is unchanged; consumers that do not implement the optional contract continue through the existing override chain.
- The contract prevents transformed prefix/reference keys from being scaled a second time when a downstream provider handles sparse and dense-owned rows separately.

## Sol-H3 production validation

The companion implementation is released as [ComfyUI-Sol-H3 v0.1.0](https://github.com/xmarre/ComfyUI-Sol-H3/releases/tag/v0.1.0). The real production stack on an RTX PRO 6000 Blackwell executed Untwist together with Sol-H3, VDN, Spectrum, Diff-Aid and Flow.

Final sampler accounting was:

```text
sampler_logical_calls       18
transformer_actual_nfe      14
spectrum_forecast_calls      4

low:    8 actual / 2 forecast
high:   4 actual / 2 forecast
probe:  2 actual / 0 forecast
```

Untwist preprocessing remained single-application in the audited provider chain. Sol-H3's native VDN and Flow mixed routes retained 1:1 requested/kernel Q-row accounting, zero square-Q expansion and finite arithmetic gates around `~1e-3` relative L2. No Untwist-specific numerical fallback or duplicate preprocessing was observed.

The one extra actual call relative to the 13/5 Sol-bypassed control is the intentional first low-stage `dense -> sol` backend transition; it is not an Untwist regression.

## Validation

- The Untwist CPU suite passed 40 tests on the companion PR head.
- Sol-H3's pinned native-interop suite exercised the real Untwist preprocessing contract together with VDN, Spectrum, Diff-Aid and Flow.
- Full production GPU execution then passed on SM120 with the final 18/14/4 Spectrum topology described above.

This release changes interoperability exposure only. Untwist's frequency scaling, reference selection, denoising windows, terminal-PECE capability, Flux/Flux.2 behavior and ordinary dense attention math are unchanged.

---

# ComfyUI Untwisting RoPE v0.2.3

This release adds the versioned MiniMax H3 visual-reference capability used by Spectrum to avoid a redundant terminal SA-Solver PECE predictor evaluation in the narrowly reviewed safe envelope.

## MiniMax H3 terminal PECE capability

- Bumps the namespaced MiniMax H3 visual-reference profile/runtime contract to schema v2.
- Publishes `terminal_pece_exact_corrector_safe=true` only when the Untwist profile is the reviewed weak terminal spatial-only case: `progress_start == 0`, `0.90 <= progress_end < 1.0`, aggregate scale strength `<= 0.05`, `image_only` or `image_and_video`, and no temporal-axis scaling.
- All other profiles publish the capability as `false`.
- The capability is eligibility metadata only. Spectrum still has to prove active PECE topology, an immediate same-outer corrected phase, and an exact persistent corrected endpoint before it may keep the terminal predictor forecasted.
- Untwist's configured denoising window is unchanged. The normalized-sigma lower boundary remains `1 - end_percent`; the usual `end_percent=0.95` case therefore reports `0.05` rather than extending Untwist through the final denoiser call.

## Compatibility and failure policy

- Schema-v1 consumers remain conservative and receive no terminal-PECE capability.
- Unknown, strong, early, temporal, Continuum-reference-inclusive, malformed, or mismatched profiles remain on the ordinary exact hard-boundary path.
- Native Untwist attention math, defaults, reference selection, Continuum exclusion, and Flux/Flux.2 behavior are unchanged.
- The existing Comfy Registry package ID remains unchanged.

## Validation

PR #5 passed its 39-test suite, Ruff critical checks, `compileall`, and GitHub Actions run 22. The merged `main` commit also passed tests run 23.

The companion Spectrum implementation was validated with matched MiniMax H3 decoded-media A/B testing and follow-up Continuum workflows. The shortest exercised progressive high-stage lifetime uses `P0 actual -> P1 forecast -> C1 actual`; both tested high-stage invocations confirmed the exact same-outer corrector with zero terminal fail-safe events and zero Spectrum fallbacks.

Companion consumer: https://github.com/xmarre/ComfyUI-Spectrum-MiniMax-H3/pull/98
