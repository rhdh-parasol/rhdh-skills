# Working with fullsend

This repository runs [fullsend](https://github.com/fullsend-ai/fullsend) agents
on its issues and pull requests. You ask an agent to triage, implement, review,
or fix something by commenting a slash command. This page covers the commands,
where the configuration lives, and how agent pull requests get checked. The
upstream [fullsend documentation](https://fullsend.sh/docs) covers the rest.

## Slash commands

Comment one of these on an issue or pull request.

| Command | Works on | What it does |
| ------- | -------- | ------------ |
| `/fs-triage` | Issue or PR | The [triage](https://fullsend.sh/docs/agents/triage) agent categorizes, labels, and assesses the item. |
| `/fs-code` | Issue | The [code](https://fullsend.sh/docs/agents/code) agent implements the issue and opens a pull request. Add text after the command to give it specific guidance. |
| `/fs-review` | PR | The [review](https://fullsend.sh/docs/agents/review) agent reviews the pull request. |
| `/fs-fix` | PR | The [fix](https://fullsend.sh/docs/agents/fix) agent addresses review feedback on the pull request. |
| `/fs-fix-stop` | PR | Adds the `fullsend-no-fix` label so the fix agent stops responding to that pull request. The comment must be exactly `/fs-fix-stop` with no other text, and only the PR author or a collaborator with write access can use it; anyone else's comment is ignored. Remove the label or comment `/fs-fix` to bring it back. |
| `/fs-retro` | Issue or PR | The [retro](https://fullsend.sh/docs/agents/retro) agent analyzes completed work and proposes improvements. |
| `/fs-prioritize` | Issue or PR | The [prioritize](https://fullsend.sh/docs/agents/prioritize) agent scores the issue for project board ranking. |

The usual flow for an issue is `/fs-triage`, then `/fs-code` once triage marks
it ready. The `ready-to-code` label also dispatches the code agent.

## Configuration

[`.fullsend/config.yaml`](../.fullsend/config.yaml) holds the repository
configuration:

- `roles` lists the agents enabled here.
- `runtime` selects the agent runtime (currently `claude`).
- `tracker.github.allowed_targets.repos` lists the repositories agents may act
  on besides this one.
- `allowed_remote_resources` and `inference` set what agents may fetch and
  which inference project they use.

Two workflows run the agents, and both call reusable workflows pinned to a
fullsend release:

- [`.github/workflows/fullsend.yaml`](../.github/workflows/fullsend.yaml)
  routes issue, comment, and pull request events to the agents and handles
  `/fs-fix-stop`.
- [`.github/workflows/prioritize.yml`](../.github/workflows/prioritize.yml)
  runs the prioritize agent.

fullsend manages both workflow files. Do not edit them by hand; change the
configuration or upgrade fullsend instead.

## Agent pull requests

An agent pull request is held to the same bar as a human one. CI runs the same
`uv run pytest`, `uv run ruff check .`, and `uv run ruff format --check .` jobs
described in [CONTRIBUTING.md](../CONTRIBUTING.md), and the same repository
rules apply.

The [review](https://fullsend.sh/docs/agents/review) agent will not approve a
pull request that touches its protected paths, such as `.github/` workflow
files. Nothing in this repository, such as a CODEOWNERS file, adds a required
human approval on top of that, so maintainers should review those pull requests
by hand.
