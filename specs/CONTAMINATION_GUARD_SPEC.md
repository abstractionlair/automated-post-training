# Contamination Guard Spec (Clean Base-Model Loading)

Objective
- Guarantee that all base-model work is free of chat-template contamination and unintended special-token injection.

Required Utility
- CleanModelLoader (canonical). Responsibilities:
  - Load tokenizer and set tokenizer.chat_template=None; if present, default_chat_template=None.
  - Ensure add_special_tokens=False during tokenization.
  - Token-ID check: verify no Qwen chat template token IDs in any input position.
  - Delta check: assert len(tokenize(prompt, add_special_tokens=True)) == len(..., False) for sentinel prompts; any delta → abort.
  - Sentinel prompts: run a small suite to confirm base behavior (fails instruction-following sentinels; succeeds at simple completions).
  - Provenance: return loader_version (git SHA), model_name, quantization, template_disabled=True, add_special_tokens=False, strict, sentinel_tests_passed, sentinel_results.

Enforcement (strict mode)
- CleanModelLoader(strict=True) is the default: template-token contamination always aborts; the delta check and instruction-sentinel failures abort with RuntimeError.
- strict=False downgrades the delta check and sentinel failures to warnings; the actual outcomes are still recorded in provenance (sentinel_tests_passed may be False). Template-token contamination aborts regardless.
- The escape hatch exists because the behavioral sentinels are heuristic: the Stage 1 findings (docs/STAGE1_FINDINGS_AND_PIVOT.md) showed modern base models such as Qwen2.5-32B partially follow instructions, so a sentinel failure can reflect genuine base-model capability rather than contamination. Overriding must be a deliberate, recorded decision, not the default.
- Historical note: before 2026-07 the delta check and sentinel failures only warned; runs from that period did not have these aborts enforced.

Forbidden Patterns (CI Grep)
- Any script in active code calling AutoTokenizer/AUTOModel directly for base model without going through CleanModelLoader.
- Any manual setting of chat_template=None outside the utility.
- Any duplicate implementations of next-token A/B logprob scoring (must use instruction_critic).

Logging & Manifests
- Every run logs contamination checks and sentinel outcomes to artifacts and includes them in session manifests.

