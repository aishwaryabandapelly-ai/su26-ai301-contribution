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

*To be completed in Phase II.*

---

## Phase III: Build & Test

*To be completed in Phase III.*

---

## Phase IV: Pull Request & Reflection

*To be completed in Phase IV.*
