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
