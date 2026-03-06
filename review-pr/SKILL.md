---
name: review-pr
description: Adversarial code review of a pull request. Verifies claims against actual code, finds bugs, edge cases, missing tests, and security issues. Produces a structured findings document and a ready-to-post PR comment.
allowed-tools: Read, Glob, Grep, Bash(gh *), Bash(git *), Write, Agent
argument-hint: [pr-url-or-number]
---

# Adversarial PR Review

Perform a thorough, adversarial code review of a pull request. The goal is to
**verify every claim against the actual code**, find bugs the author may have
missed, and produce actionable findings.

## Input

The PR to review: `$ARGUMENTS`

If no argument is provided, ask which PR to review.

## Phase 1: Gather Context

1. **Fetch PR metadata** using `gh pr view` (description, linked issues, author, base branch, file list).
2. **Fetch the diff** using `gh pr diff`.
3. **Read linked tickets/issues** if URLs are provided in the PR description.
4. **Understand intent** — What problem is this solving? What's the expected behavior change?

## Phase 2: Scope Check

Before diving into code, assess:

- Is this PR doing one thing or multiple things? Flag if it should be split.
- How many files are changed? What's the blast radius?
- Are there unrelated changes mixed in (formatting, refactoring, dependency bumps)?

## Phase 3: Adversarial Review

Read every changed file in full (not just the diff hunks — read the surrounding
context). For each change, systematically check:

### A. Correctness

- Does the code do what the PR description claims?
- Are there edge cases the author didn't handle?
- Does the logic match the intent described in the linked ticket?
- Are there off-by-one errors, null reference risks, or boundary conditions?

### B. Root Cause Analysis

- If this is a bug fix, does the fix address the **root cause** or just the symptom?
- Could the same class of bug exist elsewhere in the codebase? Search for similar patterns.
- Is the fix in the right layer (data access, business logic, API)?

### C. Safety & Security

- Error handling: What happens when things fail? Are errors swallowed silently?
- Null/empty checks: Are inputs validated at system boundaries?
- Concurrency: Race conditions, stale data, lock ordering?
- Security: Injection, authentication bypass, data exposure?

### D. Side Effects & Blast Radius

- Will this change behavior in production beyond the stated intent?
- Does it need a feature flag, migration, or deploy coordination?
- Could it break callers, consumers, or downstream systems?
- Are there config changes that affect multiple environments?

### E. Test Coverage

- Are the changes tested? Are the tests meaningful (not just happy path)?
- Do tests cover the edge cases and failure modes identified above?
- Are there integration or concurrency scenarios that unit tests can't cover?
- If a new test project was added, does it follow existing conventions?

### F. Architecture Fit

- Does it follow existing patterns in the codebase, or introduce new ones?
- If introducing new patterns, is that intentional and justified?
- Are there existing utilities or abstractions it should use instead of rolling its own?

### G. Cross-Reference Verification

- If the PR description cites specific line numbers, file paths, or behaviors — verify each one against the actual code.
- If the fix references a root cause analysis, verify the causal chain step by step.
- If tests assert specific behavior, verify the assertions match the implementation.

## Phase 4: Write Findings

Write the review to a markdown file in the current working directory. Use the
naming convention: `review-pr-<number>.md`

### Output Structure

```markdown
# PR #<number> Review: <title>

**PR:** #<number> — `<branch>`
**Author:** <author>
**Date reviewed:** <today>
**Status:** <Approve | Approve with concerns | Request changes>

---

## Summary

<2-3 sentence summary of what the PR does and the review verdict>

## Findings

### Blocking Issues
Issues that must be fixed before merge.
[For each: what's wrong, why it matters, file:line reference, suggested fix]

### Concerns
Issues that should be addressed but aren't strictly blocking.
[For each: the concern, risk level, file:line reference]

### Questions
Things that need clarification from the author to complete the review.

### Verified Correct
Important claims or logic that was checked and confirmed accurate.
[Brief list — builds author confidence and shows review thoroughness]

## Validation

### Pre-merge validation
[Queries, scripts, or manual checks to run before merging]

### Post-deploy validation
[How to confirm the change works in production]

## PR Comment

Ready-to-post comment for the PR. Keep it concise and actionable.
Use triple-backtick fence so the reviewer can copy-paste it directly.
```

## Guidelines

- **Read the code, don't trust the description.** The whole point is to catch
  things the author missed or got wrong.
- **Be specific.** Every finding must reference a file path and line number.
- **Distinguish severity.** Not all findings are equal. A missed edge case that
  causes data corruption is more important than a naming convention violation.
- **Don't just nitpick.** Focus on findings that affect correctness, safety, or
  production reliability. Skip style preferences and formatting opinions.
- **Be fair.** If the code is correct, say so in the "Verified Correct" section.
  The goal is accuracy, not finding fault.
- **Search for similar patterns.** If a bug fix addresses a specific pattern,
  grep the codebase for the same pattern elsewhere. Report if found.
- **Think about the deploy.** What validation should happen before and after
  this reaches production?
- **Propose, don't just criticize.** If something is wrong, suggest what the
  fix should look like.
