# kca-sandbox

Manual QA sandbox for [`knowledge-work/command-action`](https://github.com/knowledge-work/command-action) `@v2`,
exercising the **issue**, **pull_request**, and **discussion** comment contexts.

## Purpose

Verify that posting `.test ...` as a comment triggers the action and produces:

1. An :eyes: reaction on the source comment.
2. A bot reply containing a Markdown table with all action outputs.

The reply table is what the automated E2E sweep asserts against; humans can use the same
table for ad-hoc checks with different commands and params.

## Setup prerequisites

- GitHub Discussions feature is enabled (already on for this repo).
- At least one Discussion category exists (this repo has the GitHub defaults: Announcements,
  General, Ideas, Polls, Q&A, Show and tell).
- Repo Settings → Actions → General → **Workflow permissions** is set to
  **Read and write permissions** so the `GITHUB_TOKEN` can write reactions and discussion replies.

## How to test — Issue comment

1. Open an issue (any title / body).
2. Post a comment containing `.test name="alice", count=3, enabled=true`.
3. Wait a few seconds for the `Issue / PR comment` workflow to run.
4. Confirm the comment receives an :eyes: reaction and a bot reply with `| context | issue |`
   in the table.

## How to test — PR comment

1. Open a pull request from any branch.
2. Post a top-level PR comment with `.test name="alice", count=3, enabled=true`.
3. Confirm an :eyes: reaction and a bot reply with `| context | pull_request |`.

Note: line-level review comments (`pull_request_review_comment` event) are **not** handled —
the action only listens to `issue_comment`, which delivers top-level PR comments.

## How to test — Discussion comment

1. Open a discussion in any category.
2. Post a comment with `.test name="alice", count=3, enabled=true`.
3. Confirm an :eyes: reaction and a bot reply with `| context | discussion |`.
4. Confirm the reply table does **not** contain an `issue_number` row — the action does not
   emit `issue_number` for discussion context.

## What to look for

- The reply table renders all action outputs: `context`, `command`, `number`, `comment_id`,
  `actor`, `params`. The deprecated `issue_number` output is still emitted by the action on
  issue/PR contexts but is intentionally omitted from the reply table; check the workflow
  log (`Echo outputs` step) to see its value.
- `params` should be a JSON object reflecting the key/value pairs from the comment.
- Comments that do not start with `.test` (e.g., `.unknown`, `hello world`) should leave the
  workflow run at `continue=false`, with no reaction and no bot reply.

