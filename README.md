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

This issue is about adding support for reasoning token counts in Dynamo's OpenAI-compatible chat completion response usage field. Dynamo can separate reasoning output from normal assistant content, but the `usage.completion_token_details.reasoning_tokens` field is currently not populated. A successful fix would make this field show a non-zero reasoning token count when a reasoning model produces reasoning output.

### Why I Chose This Issue

I chose this issue because it is clearly scoped, labeled as a good first issue, and related to LLM inference observability. The issue is a good fit for the AI301 timeline because the expected behavior is specific: reasoning tokens should be counted and exposed in the response usage field.

This issue also matches my interest in AI infrastructure and OpenAI-compatible APIs. It will help me learn how real open-source AI serving systems track token usage, handle reasoning parser output, and maintain Rust backend logic.

### Initial Plan

In Phase II, I will set up the Dynamo repository locally, inspect the chat completion response generation code, reproduce the current missing reasoning token count behavior, and identify where completion usage is calculated. Then I will update the relevant Rust logic and add a test to verify that reasoning tokens are included in `completion_token_details.reasoning_tokens`.

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

The planned fix is to update Chat Completions so that when the backend provides `completion_usage.completion_tokens_details`, Dynamo copies it into `self.usage.completion_tokens_details`.

This should allow `completion_token_details.reasoning_tokens` to appear correctly in Chat Completions usage output.

### Code Change Drafted

I added the following logic in `chat_completions/delta.rs`:

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

Next, I will look for or add a targeted unit test that verifies Chat Completions preserves `completion_tokens_details.reasoning_tokens`, then prepare the pull request.

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

The main challenge was validating a large Rust-based AI infrastructure project on macOS. I installed Rust, Cargo, and protobuf, but full validation is limited because parts of Dynamo depend on Linux-only APIs. To keep the work scoped, I added a small fix following the existing pattern from the regular Completions API and added a targeted unit test for the Chat Completions path.

### Next Steps

My next step is to prepare for Phase IV by opening a pull request or draft pull request from my pushed branch, describing the implementation clearly, and responding to any maintainer feedback.

---

## Phase IV: Pull Request & Reflection

*To be completed in Phase IV.*
