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

### Next Steps

My next step was to prepare for Phase IV by opening a pull request from my pushed branch, describing the implementation clearly, and responding to any maintainer feedback.

---

## Phase IV: Submit & Iterate

### Status

PR submitted / awaiting maintainer review

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

After the PR was opened, Linux CI validated the code path. Rust tests and Rust clippy passed on CI. Some remaining CI issues appear related to fork permissions or repository-wide infrastructure checks, not to this code change.

### Acceptance Criteria

* [x] Tests added for changed behavior
* [x] Relevant Rust tests passed on Linux CI
* [x] Rust clippy passed on Linux CI
* [x] Follows existing code style and mirrors the existing `prompt_tokens_details` propagation pattern
* [x] No unrelated files changed
* [x] No breaking changes introduced
* [x] Documentation update not applicable because this is a small backend usage-field fix

### Maintainer Feedback

No human maintainer feedback has been received yet. The PR is currently open and awaiting maintainer review.

CodeRabbit reviewed the PR and did not leave any actionable inline comments. The PR title check passed, DCO passed, Rust tests passed, and Rust clippy passed. Remaining CI failures appear related to fork permissions and repository-wide external link or infrastructure checks rather than the code change itself.

If maintainers request changes, I will update the branch with follow-up commits and respond clearly to each review comment.

### Next Steps

Monitor the PR for maintainer feedback and CI updates. If no maintainer review is received after several business days, I may leave a polite follow-up comment asking whether anything else is needed to move the PR forward.

---

## Learnings & Reflections

The biggest lesson from this contribution was learning how to trace an existing behavior pattern in a large open-source codebase and apply it consistently. Comparing the Chat Completions implementation with the regular Completions implementation helped me identify the missing `completion_tokens_details` propagation.

I also learned more about the real open-source workflow: choosing a scoped issue, investigating the relevant code path, making a minimal fix, adding a targeted test, rebasing on upstream main, force-pushing safely with `--force-with-lease`, and documenting testing limitations clearly.

Another important learning was understanding how CI behavior can differ between local development and upstream Linux environments. My macOS setup could not complete some checks because Dynamo uses Linux-specific APIs, but the PR's Linux CI helped validate the actual Rust code path.

This phase helped me understand that a good open-source PR is not only about writing code. It is also about keeping the diff small, explaining the reasoning clearly, respecting the project's PR template, and being ready to respond professionally to maintainer feedback.
