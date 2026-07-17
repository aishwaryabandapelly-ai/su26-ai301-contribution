# AI301 Open Source Contribution Log

## Student Information

Name: Aishwarya B  
Course: AI301 Open Source Capstone  
Program: CodePath Summer 2026

---

## Phase I: Issue Selection

### Status

Phase I Complete

### Chosen Issue

[FEATURE]: Support reasoning tokens in response usage field  
https://github.com/ai-dynamo/dynamo/issues/2941

### Project Repository

https://github.com/ai-dynamo/dynamo

### Problem Summary

This issue is about adding support for reasoning token counts in Dynamo's OpenAI-compatible Chat Completions response usage field. Dynamo can separate reasoning output from normal assistant content, but the `usage.completion_tokens_details.reasoning_tokens` field was not being populated in the final response.

A successful fix would make this field preserve and return the reasoning token count when the backend provides reasoning token usage metadata.

### Why I Chose This Issue

I chose this issue because it is clearly scoped, labeled as a good first issue, and related to LLM inference observability. The issue is a good fit for the AI301 timeline because the expected behavior is specific: reasoning tokens should be counted and exposed in the response usage field.

This issue also matches my interest in AI infrastructure and OpenAI-compatible APIs. It helped me learn how real open-source AI serving systems track token usage, handle response metadata, and maintain Rust backend logic.

### Initial Plan

In Phase II, I planned to set up the Dynamo repository locally, inspect the Chat Completions response generation code, reproduce or investigate the missing reasoning token count behavior, and identify where completion usage is calculated. Then I planned to update the relevant Rust logic and add a test to verify that reasoning tokens are included in `completion_tokens_details.reasoning_tokens`.

---

## Phase II: Reproduce & Plan

### Status

Phase II Complete

### Local Setup

I cloned my fork of the Dynamo repository, added the original repository as upstream, and created a Phase II branch.

```bash
git clone https://github.com/aishwaryabandapelly-ai/dynamo.git
cd dynamo
git remote add upstream https://github.com/ai-dynamo/dynamo.git
git checkout -b phase2-reasoning-token-plan
```

I also installed Rust and Cargo using rustup.

```bash
rustc --version
cargo --version
```

### Issue

[FEATURE]: Support reasoning tokens in response usage field  
https://github.com/ai-dynamo/dynamo/issues/2941

### Reproduction / Investigation

I investigated the Chat Completions code path under:

```text
lib/llm/src/protocols/openai/chat_completions
```

The main file involved is:

```text
lib/llm/src/protocols/openai/chat_completions/delta.rs
```

I found that `delta.rs` already tracks usage and increments `completion_tokens` using `delta.token_ids.len()`. It also copies `prompt_tokens` and `prompt_tokens_details` from backend-provided `completion_usage`.

However, Chat Completions was not copying `completion_tokens_details`, which means fields like `reasoning_tokens` may not be preserved in the final usage response.

I compared this with:

```text
lib/llm/src/protocols/openai/completions/delta.rs
```

and found that the regular Completions API already propagates `completion_tokens_details`.

### Planned Fix

The planned fix was to update Chat Completions so that when the backend provides `completion_usage.completion_tokens_details`, Dynamo copies it into `self.usage.completion_tokens_details`.

This should allow `completion_tokens_details.reasoning_tokens` to appear correctly in Chat Completions usage output.

### Code Change Drafted

I drafted the following logic in `chat_completions/delta.rs`:

```rust
// Propagate completion token details if provided, including reasoning tokens.
if let Some(completion_details) = completion_usage.completion_tokens_details.as_ref() {
    self.usage.completion_tokens_details = Some(completion_details.clone());
}
```

### Validation

I ran:

```bash
cargo fmt
git diff --check
```

Both completed successfully.

I also ran:

```bash
cargo check -p dynamo-llm
```

The build progressed into the `dynamo-llm` crate but failed on macOS due to Linux-specific APIs related to NUMA, disk storage, `fallocate`, and `O_DIRECT`. This appears to be a platform/environment limitation, not an error caused by my code change.

### Next Steps

Next, I planned to add a targeted unit test that verifies Chat Completions preserves `completion_tokens_details.reasoning_tokens`, then prepare the pull request.

---

## Phase III: Build & Test

### Status

Phase III Complete

### Implementation Notes

For Phase III, I implemented the planned fix for the Dynamo issue:

**Issue:** [FEATURE]: Support reasoning tokens in response usage field  
**Issue Link:** https://github.com/ai-dynamo/dynamo/issues/2941

The main file modified was:

```text
lib/llm/src/protocols/openai/chat_completions/delta.rs
```

During Phase II, I found that Chat Completions already copied `prompt_tokens` and `prompt_tokens_details` from backend-provided `completion_usage`, but it did not copy `completion_tokens_details`. This meant fields such as `reasoning_tokens` could be lost from the final OpenAI-compatible usage response.

For Phase III, I updated the Chat Completions delta generator to propagate `completion_tokens_details` from backend usage metadata:

```rust
// Propagate completion token details if provided, including reasoning tokens.
if let Some(completion_details) = completion_usage.completion_tokens_details.as_ref() {
    self.usage.completion_tokens_details = Some(completion_details.clone());
}
```

I also added a targeted unit test to verify that completion token details are preserved from backend usage:

```text
test_completion_token_details_are_propagated_from_backend_usage
```

### Code Changes

Active development branch:

```text
https://github.com/aishwaryabandapelly-ai/dynamo/tree/phase2-reasoning-token-plan
```

Commits completed:

```text
fix: propagate chat completion token details
test: cover chat completion token details propagation
```

Latest signed commit hashes after fixing the GPG signing requirement:

```text
3ffcd85 fix: propagate chat completion token details
f0c5b15 test: cover chat completion token details propagation
```

### Testing Strategy

I ran:

```bash
cargo fmt
git diff --check
```

Both completed successfully.

I also attempted:

```bash
cargo check -p dynamo-llm
cargo test -p dynamo-llm test_completion_token_details_are_propagated_from_backend_usage --lib
```

Both commands progressed into the `dynamo-llm` crate but failed on macOS because of Linux-specific APIs related to NUMA, disk storage, `fallocate`, and `O_DIRECT`. This appears to be a local platform/environment limitation rather than an error caused by my code change.

### Challenges Faced

The main challenge was validating a large Rust-based AI infrastructure project on macOS. I installed Rust, Cargo, and protobuf, but full validation was limited because parts of Dynamo depend on Linux-only APIs. To keep the work scoped, I added a small fix following the existing pattern from the regular Completions API and added a targeted unit test for the Chat Completions path.

Another challenge was learning how to handle open-source CI requirements for external contributors. After the PR was opened, Dynamo required my commits to be GPG-signed before some CI workflows could run. I installed GPG, created a GPG key, added the public key to GitHub, configured Git commit signing, signed both commits, rebased onto the latest upstream main, and force-pushed safely with `--force-with-lease`.

### Next Steps

My next step was to prepare for Phase IV by opening a pull request from my pushed branch, describing the implementation clearly, and responding to any maintainer feedback.

---

## Phase IV: Submit & Iterate

### Status

PR submitted / GPG signing blocker fixed / CI running / awaiting human code-owner review

### Pull Request

**PR Title:** `fix(llm): propagate chat completion token details`  
**PR Link:** https://github.com/ai-dynamo/dynamo/pull/11027  
**Issue Link:** https://github.com/ai-dynamo/dynamo/issues/2941

### PR Description

**What does this PR do?**

This PR updates Dynamo's Chat Completions delta generator so that backend-provided `completion_tokens_details`, including `reasoning_tokens`, are propagated into the final OpenAI-compatible `usage` response.

**Why was this PR needed?**

Issue #2941 reported that reasoning token usage was not being surfaced in the response usage field. During investigation, I found that Chat Completions copied `prompt_tokens` and `prompt_tokens_details` from backend usage metadata, but did not copy `completion_tokens_details`. The regular Completions API already had this propagation pattern, so I mirrored that behavior in the Chat Completions path.

**Relevant issue:**

Closes #2941

### Summary of Changes

I opened a pull request against `ai-dynamo/dynamo` that makes the Chat Completions API propagate `completion_tokens_details` from backend-provided usage metadata into the final OpenAI-compatible `usage` response.

The change is in:

```text
lib/llm/src/protocols/openai/chat_completions/delta.rs
```

Previously, Chat Completions copied `prompt_tokens` and `prompt_tokens_details` from the backend `completion_usage` but did not copy `completion_tokens_details`, so fields such as `reasoning_tokens` were dropped from the response. The fix mirrors the pattern already used by the regular Completions API:

```rust
// Propagate completion token details if provided, including reasoning tokens.
if let Some(completion_details) = completion_usage.completion_tokens_details.as_ref() {
    self.usage.completion_tokens_details = Some(completion_details.clone());
}
```

I also added a targeted unit test that feeds a backend `CompletionUsage` with `reasoning_tokens: Some(3)` through the delta generator and asserts the value is propagated to the response usage:

```text
test_completion_token_details_are_propagated_from_backend_usage
```

Before opening the PR, I rebased my branch onto the latest `upstream/main` so the change applies cleanly on top of current main.

After a CI bot detected that my commits were unsigned, I created a GPG key, added it to GitHub, signed both commits, rebased onto the latest upstream main again, and force-pushed the signed commits. The unsigned commit warning is now resolved.

### Testing Notes

I ran locally:

```bash
cargo fmt
git diff --check
```

Both passed.

I also attempted locally:

```bash
cargo check -p dynamo-llm
cargo test -p dynamo-llm test_completion_token_details_are_propagated_from_backend_usage --lib
```

Both commands progressed into the `dynamo-llm` crate but could not complete on macOS because Dynamo depends on Linux-specific APIs such as NUMA, `DiskStorage`, `fallocate`, and `O_DIRECT`. This is a local platform/environment limitation, not an error caused by my code change.

After the PR was opened, Linux CI validated the code path. Rust tests and Rust clippy passed on CI before the branch was updated for GPG signing. After I signed the commits and force-pushed, CI started running again.

### Acceptance Criteria

* [x] Tests added for changed behavior
* [x] Rust code formatted
* [x] No unrelated files changed
* [x] No breaking changes introduced
* [x] Follows existing code style and mirrors the existing `prompt_tokens_details` propagation pattern
* [x] Commits are GPG-signed after CI requested signed commits
* [x] Documentation update not applicable because this is a small backend usage-field fix
* [ ] Final CI completion after signed commit push
* [ ] Human code-owner review

### Maintainer Feedback

No human maintainer feedback requiring code changes has been received yet. The PR is currently open and awaiting human code-owner / maintainer review.

Automated review status has been positive. CodeRabbit reviewed the PR and summarized the change as a small/trivial update. The dynamo-review-agent approved the changes, and Devin Review reported no issues.

A Dynamo bot later detected that the original commits were not GPG-signed. I addressed this by:

```text
1. Installing GPG and pinentry-mac
2. Creating a GPG key
3. Adding the public GPG key to GitHub
4. Configuring Git to sign commits
5. Signing both PR commits
6. Rebasing onto latest upstream/main
7. Force-pushing the signed commits with --force-with-lease
```

After this, the unsigned-commit warning disappeared and GitHub Actions started running again.

The PR is currently blocked by required CI checks still completing and by required code-owner review. The current GitHub merge message says that code-owner review is required before the PR can be merged.

If maintainers request changes, I will update the branch with follow-up commits and respond clearly to each review comment.

### Next Steps

Monitor the PR for CI updates and maintainer feedback. I will not push again or update the branch unless a maintainer requests it. If no maintainer review is received after several business days, I may leave a polite follow-up comment asking whether anything else is needed to move the PR forward.

---

## Learnings & Reflections

The biggest lesson from this contribution was learning how to trace an existing behavior pattern in a large open-source codebase and apply it consistently. Comparing the Chat Completions implementation with the regular Completions implementation helped me identify the missing `completion_tokens_details` propagation.

I also learned more about the real open-source workflow: choosing a scoped issue, investigating the relevant code path, making a minimal fix, adding a targeted test, rebasing on upstream main, force-pushing safely with `--force-with-lease`, and documenting testing limitations clearly.

Another important learning was understanding how CI behavior can differ between local development and upstream Linux environments. My macOS setup could not complete some checks because Dynamo uses Linux-specific APIs, but the PR's Linux CI helped validate the actual Rust code path.

I also learned how signed commits work in open-source repositories. Dynamo required my commits to be GPG-signed before CI could fully run. I created and configured a GPG key, added it to GitHub, signed my commits, and force-pushed the corrected branch. This helped me understand an important part of contributor security and verification in professional open-source workflows.

This phase helped me understand that a good open-source PR is not only about writing code. It is also about keeping the diff small, explaining the reasoning clearly, respecting the project's PR template, handling CI requirements carefully, and being ready to respond professionally to maintainer feedback.

---

## Cycle 2 Open Source Contribution — vllm-omni

### Repository

- Upstream repository: https://github.com/vllm-project/vllm-omni
- My fork: https://github.com/aishwaryabandapelly-ai/vllm-omni

### Selected Issue

- Issue: https://github.com/vllm-project/vllm-omni/issues/2462
- Title: [New Model]: LongCat-AudioDiT (Meituan) — Waveform Latent Space Diffusion TTS

### Current Status

I selected this issue as my second open-source contribution and commented on the issue to express interest in working on it.

I forked the repository, cloned it locally, added the upstream remote, and created a new investigation branch:

```text
cycle2-longcat-audiodit-investigation
```

### Investigation Completed So Far

I started by exploring how `vllm-omni` supports existing TTS and audio/diffusion models. The closest existing references I found are:

- `qwen3_tts` — TTS pipeline structure
- `cosyvoice3` — Talker → Code2Wav pipeline with DiT / flow-matching decoder
- `stable_audio` — audio diffusion model pattern
- `longcat_image` — existing LongCat diffusion model naming and pipeline pattern

Important files identified:

- `vllm_omni/config/pipeline_registry.py`
- `vllm_omni/model_executor/models/registry.py`
- `vllm_omni/model_executor/models/qwen3_tts/pipeline.py`
- `vllm_omni/model_executor/models/cosyvoice3/pipeline.py`
- `vllm_omni/diffusion/models/stable_audio/`
- `vllm_omni/diffusion/models/longcat_image/`
- `docs/contributing/model/adding_tts_model.md`

### Initial Understanding

LongCat-AudioDiT appears to be larger than a simple one-file change because it is a waveform latent-space diffusion TTS model. Based on the repository’s TTS contribution guide, the correct next step is to study the official LongCat-AudioDiT reference implementation before making code changes.

The actual goal of the issue is to add support for LongCat-AudioDiT inside `vllm-omni`, so users can run it as a text-to-speech model. This likely requires understanding how LongCat-AudioDiT handles text input, reference audio or voice cloning, Wav-VAE encoding/decoding, diffusion sampling, scheduler behavior, and final audio output.

### Next Steps

- Clone and inspect the official LongCat-AudioDiT reference repository
- Identify its model components, config structure, scheduler, audio/VAE flow, and inference path
- Compare it with existing `vllm-omni` patterns
- Decide the smallest safe PR scope, likely either:
  - documentation/investigation notes,
  - model registration/pipeline skeleton,
  - or a scoped first implementation step

### Notes

No code changes have been made yet for Cycle 2. I am intentionally doing investigation first to avoid making an incomplete or incorrect model integration.

The first Dynamo PR remains my completed primary contribution, while the vllm-omni issue is my second-cycle investigation and planning work.
