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

PR merged

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

After a CI bot detected that my commits were unsigned, I created a GPG key, added it to GitHub, signed both commits, rebased onto the latest upstream main again, and force-pushed the signed commits. The unsigned commit warning was resolved and GitHub Actions started running again.

The PR was later merged, completing my first open-source contribution for AI301.

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

The PR was eventually accepted and merged upstream.

### Acceptance Criteria

* [x] Tests added for changed behavior
* [x] Rust code formatted
* [x] No unrelated files changed
* [x] No breaking changes introduced
* [x] Follows existing code style and mirrors the existing `prompt_tokens_details` propagation pattern
* [x] Commits are GPG-signed after CI requested signed commits
* [x] Documentation update not applicable because this is a small backend usage-field fix
* [x] Final CI completed successfully
* [x] Human maintainer / code-owner review completed
* [x] PR merged

### Maintainer Feedback

No major human maintainer feedback requiring code changes was received on the implementation itself. Automated review status was positive. CodeRabbit reviewed the PR and summarized the change as a small/trivial update. The dynamo-review-agent approved the changes, and Devin Review reported no issues.

A Dynamo bot detected that the original commits were not GPG-signed. I addressed this by:

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

The PR was later merged, which means the required project checks and review process were completed successfully.

### Final Outcome

My first AI301 open-source contribution was successfully merged into the upstream Dynamo repository.

This contribution fixed missing propagation of `completion_tokens_details` in Dynamo's Chat Completions usage response, allowing reasoning token details from backend usage metadata to be preserved in the final OpenAI-compatible response.

---

## Learnings & Reflections

The biggest lesson from this contribution was learning how to trace an existing behavior pattern in a large open-source codebase and apply it consistently. Comparing the Chat Completions implementation with the regular Completions implementation helped me identify the missing `completion_tokens_details` propagation.

I also learned more about the real open-source workflow: choosing a scoped issue, investigating the relevant code path, making a minimal fix, adding a targeted test, rebasing on upstream main, force-pushing safely with `--force-with-lease`, and documenting testing limitations clearly.

Another important learning was understanding how CI behavior can differ between local development and upstream Linux environments. My macOS setup could not complete some checks because Dynamo uses Linux-specific APIs, but the PR's Linux CI helped validate the actual Rust code path.

I also learned how signed commits work in open-source repositories. Dynamo required my commits to be GPG-signed before CI could fully run. I created and configured a GPG key, added it to GitHub, signed my commits, and force-pushed the corrected branch. This helped me understand an important part of contributor security and verification in professional open-source workflows.

This phase helped me understand that a good open-source PR is not only about writing code. It is also about keeping the diff small, explaining the reasoning clearly, respecting the project's PR template, handling CI requirements carefully, and being ready to respond professionally to maintainer feedback.

Most importantly, this contribution taught me that getting a PR merged requires both technical correctness and professional collaboration. The final merge gave me confidence that I can contribute to real AI infrastructure projects.

---

## Cycle 2 Open Source Contribution — Investigation and Issue Re-selection

### Current Status

For my second open-source contribution cycle, I first selected a `vllm-omni` issue related to LongCat-AudioDiT support. After investigation, I found that another pull request was already actively tracking the same model support work. Because of that, I decided not to open a duplicate PR and instead started looking for a different issue with a cleaner contribution path.

I then selected an OpenTelemetry Weaver issue related to CI build performance, commented on the issue, implemented a scoped Docker cache improvement, and opened a pull request.

---

## Cycle 2 Attempt 1 — vllm-omni LongCat-AudioDiT Investigation

### Repository

- Upstream repository: https://github.com/vllm-project/vllm-omni
- My fork: https://github.com/aishwaryabandapelly-ai/vllm-omni

### Issue Investigated

- Issue: https://github.com/vllm-project/vllm-omni/issues/2462
- Title: [New Model]: LongCat-AudioDiT (Meituan) — Waveform Latent Space Diffusion TTS

### Work Completed

I selected this issue as my second open-source contribution candidate and commented on the issue to express interest in working on it.

I forked the repository, cloned it locally, added the upstream remote, and created an investigation branch:

```text
cycle2-longcat-audiodit-investigation
```

I explored how `vllm-omni` supports existing TTS and audio/diffusion models. The closest references I found were:

- `qwen3_tts` — TTS pipeline structure
- `cosyvoice3` — Talker → Code2Wav pipeline with DiT / flow-matching decoder
- `stable_audio` — audio diffusion model pattern
- `longcat_image` — existing LongCat diffusion model naming and pipeline pattern

Important files investigated:

```text
vllm_omni/config/pipeline_registry.py
vllm_omni/model_executor/models/registry.py
vllm_omni/model_executor/models/qwen3_tts/pipeline.py
vllm_omni/model_executor/models/cosyvoice3/pipeline.py
vllm_omni/diffusion/models/stable_audio/
vllm_omni/diffusion/models/longcat_image/
docs/contributing/model/adding_tts_model.md
```

I also inspected the official LongCat-AudioDiT reference implementation to understand the model structure. From that investigation, I learned that LongCat-AudioDiT is not a simple TTS model integration. It is a waveform latent-space diffusion TTS model involving:

- UMT5 text encoder
- AudioDiT diffusion transformer
- Wav-VAE encoder/decoder
- CFG/APG guidance
- duration and latent-hop handling
- optional prompt audio / voice cloning flow
- final waveform generation at 24 kHz

### Local Groundwork Attempt

To understand the smallest possible contribution, I created a small local implementation branch:

```text
feat-audiodit-config-registration
```

On that branch, I added local groundwork for HuggingFace-style config recognition:

- `AudioDiTConfig`
- `AudioDiTVaeConfig`
- `AutoConfig.register("audiodit", AudioDiTConfig)`
- `AutoConfig.register("audiodit_vae", AudioDiTVaeConfig)`
- config exposure through `vllm_omni.transformers_utils.configs`
- a unit test for config defaults, nested config conversion, and AutoConfig registration

I also created a signed local commit:

```text
feat: add AudioDiT config registration
```

During validation, I learned that running full pytest locally was blocked because the repository’s shared pytest fixtures import `torch`, which was not installed in my lightweight local environment. I created a Python 3.12 virtual environment using `uv` because the project requires Python `>=3.10, <3.14`, while my system Python versions were either too new or too old.

I successfully ran lightweight validation:

```bash
PYTHONPATH=. python -m py_compile \
  vllm_omni/transformers_utils/configs/audiodit.py \
  vllm_omni/transformers_utils/configs/__init__.py \
  tests/unit/audiodit/test_audiodit_config.py
```

I also directly validated config construction and `AutoConfig.for_model()` registration locally.

### Why I Did Not Open a PR

Before opening the PR, I checked the issue discussion more carefully and found that LongCat-AudioDiT support was already being tracked in an existing PR:

```text
https://github.com/vllm-project/vllm-omni/pull/2387
```

That PR already included broader LongCat-AudioDiT model support, including transformer and pipeline work, and it was linked to the same issue. Because of that, opening my config-only PR would likely be a duplicate or create conflicts with ongoing maintainer work.

I decided not to open a pull request for my local branch. This was an important open-source workflow lesson: even if the code works locally, it is better to avoid duplicate PRs when an existing PR already covers the same feature.

### What I Learned From This Attempt

This investigation still helped me learn a lot:

- How `vllm-omni` organizes model pipelines
- How TTS models are staged in the repository
- How custom model configs are registered through `transformers_utils/configs`
- How `AutoConfig.register()` works for unsupported HuggingFace model types
- How to inspect an official reference implementation before coding
- How to avoid duplicate PRs by checking linked PRs and maintainer comments
- How local validation can be limited by large ML repository dependencies
- Why small config work can still overlap with a larger model integration PR

Even though I did not submit this PR, the investigation improved my ability to read large AI infrastructure repositories and helped me choose a better next issue.

---

## Cycle 2 Attempt 2 — OpenTelemetry Weaver Build Performance

### Repository

- Upstream repository: https://github.com/open-telemetry/weaver
- My fork: https://github.com/aishwaryabandapelly-ai/weaver

### Issue Selected

- Issue: https://github.com/open-telemetry/weaver/issues/791
- Title: Increase build performance

### Why I Chose This Issue

After deciding not to open a duplicate PR for the `vllm-omni` LongCat-AudioDiT issue, I looked for a cleaner issue with:

- recent activity
- no obvious active PR already solving it
- a clear improvement goal
- a smaller contribution surface
- a `good first issue` label

The OpenTelemetry Weaver issue is about improving CI build performance by using Docker cache and potentially Rust crate cache. This was a better fit for a scoped contribution because it focused on GitHub Actions / CI optimization rather than a large model integration.

### Comment Left

I left a comment on the issue expressing interest in working on it. My plan was to first review the existing GitHub Actions workflows, compare them with the Docker cache example linked in the issue, and identify the smallest safe improvement before opening a PR.

### Local Setup

I forked the repository, cloned it locally, added the upstream remote, and created a new branch:

```bash
cd ~/Desktop/project
git clone https://github.com/aishwaryabandapelly-ai/weaver.git
cd weaver
git remote add upstream https://github.com/open-telemetry/weaver.git
git fetch upstream
git checkout -b cycle2-build-performance
```

The working tree was clean before I started making changes.

### Investigation

I inspected the repository’s GitHub Actions workflows and found the Docker publishing workflow:

```text
.github/workflows/publish-docker.yml
```

I found that this workflow already used:

```text
docker/setup-buildx-action
docker/build-push-action
```

but it did not include Docker BuildKit cache settings such as:

```text
cache-from
cache-to
```

I also noticed that many Rust workflows already used `Swatinem/rust-cache`, so I decided to keep this first PR focused on Docker build caching only. This matched the issue’s request while keeping the contribution small and safe.

### Pull Request Opened

- PR: https://github.com/open-telemetry/weaver/pull/1647
- PR Title: `ci: add Docker build cache`
- Issue: https://github.com/open-telemetry/weaver/issues/791
- Status: PR opened / CLA signed / waiting for maintainer workflow approval, CI, and review

### Summary of Changes

I opened a pull request that adds GitHub Actions BuildKit cache support to the Docker build steps in:

```text
.github/workflows/publish-docker.yml
```

The workflow builds Docker images for two platforms:

```text
linux/amd64
linux/arm64
```

To keep caches separated by platform, I added a `cache-scope` value to each Docker build matrix entry:

```yaml
- platform: linux/amd64
  runner: ubuntu-latest
  cache-scope: linux-amd64

- platform: linux/arm64
  runner: ubuntu-24.04-arm
  cache-scope: linux-arm64
```

Then I added BuildKit GitHub Actions cache settings to the existing Docker build steps:

```yaml
cache-from: type=gha,scope=weaver-docker-${{ matrix.cache-scope }}
cache-to: type=gha,mode=max,scope=weaver-docker-${{ matrix.cache-scope }}
```

This change should allow Docker layers to be reused across workflow runs, improving build performance for the Docker image build process.

### Code Changes

Active development branch:

```text
https://github.com/aishwaryabandapelly-ai/weaver/tree/cycle2-build-performance
```

Commit completed:

```text
e3fb8dd3 ci: add Docker build cache
```

The commit was GPG-signed successfully before pushing.

### Testing

I ran:

```bash
git diff --check
```

This passed successfully.

Because this is a GitHub Actions workflow-only change, the actual Docker cache behavior must be validated by GitHub Actions when the PR workflows run.

### CLA Requirement

After opening the PR, the Linux Foundation EasyCLA bot reported that my commit was not authorized under a signed CLA.

I completed the EasyCLA process by:

```text
1. Signing in with my GitHub account
2. Creating / connecting a Linux Foundation ID
3. Proceeding as an Individual Contributor
4. Signing the Individual Contributor CLA
5. Returning to the PR and confirming that EasyCLA turned green
```

This helped me learn that some large open-source foundations require a Contributor License Agreement before they can accept external contributions.

### Current Status

The PR is open and the CLA check is green.

The PR is currently waiting for:

```text
1. Maintainer approval to run workflows
2. CI results
3. Reviewer feedback
4. Final merge decision
```

This means the Docker caching part of the issue has been implemented in a PR, but the issue is not officially resolved yet. It will only be fully resolved if the maintainers approve the workflow run, CI passes, reviewers approve the change, and the PR gets merged.

### Current Cycle 2 Status

The `vllm-omni` issue remains documented as an investigation attempt, but I moved forward with OpenTelemetry Weaver issue #791 as the active Cycle 2 contribution.

The first Dynamo PR is merged and remains my completed primary contribution. The OpenTelemetry Weaver PR is now my active second contribution and is waiting for maintainer review and CI.
