# Automated Post-Training Experiment

## Background and Motivation

I found Constitutional AI really interesting when I read about it. But over time, as it sat in my mind, it changed my understanding of base models; what they already "know", and are capable of; what's latent in them. I don't mean that I was seeing anything people in the field didn't already know, nor that Constitutional AI was what initially demonstrated these things. It was just the trigger for me. (For all I know it is what introduced the idea into the field. Oh, I know who I could ask about that.)

At some point this bumped into my reading about all sorts of fine tunes and other derivations from small, open models that people were creating and posting to HuggingFace. And I wondered what I could do if I wanted a _really_ customized-to-my-liking model. That led to the Constitutional AI-inspired ideas and experiment below.

This wasn't successful (but it was educational). I do think with what I know now, and help from newer generations of models, I could carry it off. But that would still mean making a toy/demo model. I don't have the data or the compute to do more. And that's failing to properly express the degree to which the most I could get access to would fall short. Every now and then I ask Claude or ChatGPT what's the absolute best that could be done using public knowledge of methods, publicly available data (not necessarily free, just that anyone could get, possibly expensively), and compute that doesn't require a personal relationship with Jensen Huang. Here, since I'm just imagining things, I ask about doing it end to end, pre-training included. The answers are both huge from my individual perspective (unless my systematic prediction markets trading system works _really_ _really_ well), and surprisingly small from an organizational perspective. And then I wonder if it is actually a good thing that an organization could do this with that budget.

The goal I set was to fully post-train a model starting with only a base model. No other model to distill from, etc. Though the "no other model" restriction is nuanced. I'd allow the help of current, frontier models for writing code and defining procedures. But then I wanted it to be possible to take just that code, the procedures, and a base model and from that produce a real (small) helpful and harmless assistant.

In some conversations about this recently, Claude indicated that if it had worked, at the time I started it, it might have been interesting research. But it's well-known now. (And if it _would_ have been interesting, I'm inclined to believe that would only have been to people who only knew about publicized research. Not to people in the top labs.)

## Introduction

Below, Claude describes the project based on the code, docs, and maybe logs. The work was done mostly in Claude Code, maybe with some help from Codex, and I think the logs were available.

This project explores: **How much post-training can we automate using that observation?** Beyond just safety.

Can a base model generate its own training data, critique itself using principles it already understands, and bootstrap its way to aligned behavior—with minimal human intervention?

---

## The Approach

Instead of jumping straight to full Constitutional AI, we teach capabilities progressively:

**Stage 1**: Explicit instruction following (foundation) ← *Currently here*
**Stage 2**: Implicit instructions (questions & context)
**Stage 3**: Generation tasks (create examples)
**Stage 4**: Evaluation tasks (judge quality)
**Stage 5**: Revision tasks (improve text)
**Stage 6**: Constitutional integration (full CAI)

Each stage produces a functional model that helps generate training data for the next stage. True bootstrapping.

---

## Goals

- **Maximum automation** in the training pipeline
- **Progressive capability building** (each stage enables the next)
- **Reproducible methodology** (all steps scriptable and documented)
- **Budget-friendly research** (~$150 total)
- **Publication-quality results** with comprehensive documentation


---

## Navigation

**New to this project?**

1. **This file** - Project goals and approach
2. **[/specs/PROGRAM_SPEC.md](/specs/PROGRAM_SPEC.md)** - Top-level invariants and gates
3. **[/specs/stage_1_explicit_instructions.md](/specs/stage_1_explicit_instructions.md)** - Current stage methodology
4. **[/specs/complete_pipeline.md](/specs/complete_pipeline.md)** - Full pipeline architecture
5. **[/specs/stage1_data_generation_spec.md](/specs/stage1_data_generation_spec.md)**, **[/specs/stage1_sft_spec.md](/specs/stage1_sft_spec.md)**, **[/specs/stage1_evaluation_spec.md](/specs/stage1_evaluation_spec.md)**
6. **[/runbooks/AGENT_RUNBOOK_STAGE1.md](/runbooks/AGENT_RUNBOOK_STAGE1.md)** and **[/runbooks/POD_OPERATIONS.md](/runbooks/POD_OPERATIONS.md)**
7. **[/docs/TECHNICAL_SETUP.md](/docs/TECHNICAL_SETUP.md)** - Model, hardware, training details
8. **[/docs/STANDARDS.md](/docs/STANDARDS.md)** - How we work

**Critical lessons learned (v1 → v2):**

- **[/docs/BASE_MODEL_TRUTH.md](/docs/BASE_MODEL_TRUTH.md)** - Chat template contamination (critical!)
- **[/docs/KNOWN_BUGS_AND_FIXES.md](/docs/KNOWN_BUGS_AND_FIXES.md)** - Bugs we've fixed
- **[/docs/POST_TRAINING_APPROACHES.md](/docs/POST_TRAINING_APPROACHES.md)** - Methodology references

**For AI agents:**

- **Claude Code** (implementer): [CLAUDE.md](CLAUDE.md)
- **Gemini** (technical review): [GEMINI.md](GEMINI.md)
- **Codex** (methodology review): [codex.md](codex.md)

---

## Current Status

**V2 Clean Restart**: Building Stage 1 from scratch via autonomous agentic sessions.

- V1 implementation archived to `archive/v1-implementation/` (28 methodology discrepancies found)
- V2 builds clean implementation from specs via session-based approach
- GPT-5 writing high-level session specifications
- Claude Code executing autonomous implementation sessions on pod

> **Note on `constitution.yaml`**: This file is aspirational — it documents the Constitutional AI principles that will guide future stages, but no active script loads it yet. All built work is SFT-only (Stage 1 of the planned program).

---

## About

This is a personal research project exploring automated post-training methods. It's heavily AI-assisted (Claude Code for implementation, Gemini and Codex for review), which is itself an experiment in human-AI collaboration for research.

To be more specific about the split: the thesis — generating post-training data by smart-prompting base models — and the orchestration and document-DAG design are mine. Codex served as the designated methodologist under a role charter I wrote (see [codex.md](codex.md)); the single-token logprob critic was its recommendation, not mine. The code is model-written under my direction, and my engagement shows up in validity catches rather than diffs — spotting the Claude-to-Qwen chat-template contamination ([/docs/BASE_MODEL_TRUTH.md](/docs/BASE_MODEL_TRUTH.md)) is the representative example, and catches like it drove the v1-to-v2 pivots. The specs, role charters, and reviews are committed in the repo, so the collaboration is inspectable rather than just asserted.

**Architecture**: V2 uses autonomous agentic sessions where Claude Code builds implementation from specs with minimal human intervention. See [CLAUDE.md](CLAUDE.md) for details.

---

**Questions?** See `/docs/STANDARDS.md` for how things work, or [CLAUDE.md](CLAUDE.md) for autonomous session architecture.

## Note on the reported result vs. later code fixes

The Stage 1 null result documented here — that the base model already followed
instructions, so SFT gave no improvement — was measured under the training
configuration *as it ran in October 2025*, which (as a later review found)
computed loss over the full sequence rather than response tokens only. The
loss-masking has since been corrected in `scripts/train_stage1_sft.py`, but the
experiment has **not been re-run** under the corrected code. The reported
conclusion should therefore be read as provisional with respect to that fix:
response-only masking could change the SFT outcome, and revisiting it is listed
future work. The finding that mattered — *verify the capability gap exists
before training to close it* — is unaffected either way.
