# Start here

This is an open-source Kortix project. It runs one agent on a steady job, and the rules that bound that agent are two files in this repository rather than settings in someone's account.

A Kortix project is a git repository with a `kortix.yaml` at its root. An agent is one `agents.<name>` block in that file: its description, mode, prompt file, per-tool permission and grants. `agents/feedback.md` is the prompt alone, with no frontmatter. Every session runs in its own sandbox on its own branch, and work reaches the default branch only as a change request.

## What the agent does

It reads a week of customer feedback, groups the items by the problem behind them, and writes one plan per recurring problem. It commits the plans on its session branch and opens a change request. It cannot merge that change request itself, which is the point of the project. `examples/feedback.md` holds a sample week so a first run has real work to do.

## Run it

```bash
curl -fsSL https://kortix.com/install | bash
kortix login

git clone https://github.com/Kortix-seo/kortix-always-on-agent feedback-agent
cd feedback-agent
kortix ship

kortix sessions new --prompt "Triage examples/feedback.md and write one plan per recurring problem in plans/"
kortix cr ls
kortix cr diff 1
kortix cr merge 1
```

`kortix ship` validates `kortix.yaml`, commits, pushes the current branch and connects the project, creating and managing the repository when the folder has no `origin` and linking it when the folder already has one. `kortix doctor` checks the login, the project and a throwaway session end to end. The job is a prompt, so any scheduler you already run can fire `kortix sessions new` once a week: same agent, same review step, no always-on process to keep alive.

## The grant that decides who merges

An agent merges only when its grants include `project.gitops.merge`. Without it, `kortix cr merge` returns 403 with `agent_scope_insufficient`, and a person merges. This project leaves that permission out on purpose. Here is the agent as `kortix.yaml` declares it, minus the description and mode lines:

```yaml
agents:
  feedback:
    prompt_file: agents/feedback.md
    workspace: { project_repository: true }
    permission:
      edit: allow
      bash:
        "*": ask
        "git status": allow
        "git diff *": allow
        "git log *": allow
        "rm -rf *": deny
    skills: [kortix-cli]
    kortix_permissions: [project.session.read, project.review.submit]
```

An omitted grant resolves to none, so the absent `connectors` and `secrets` keys mean the agent reaches nothing and holds no credential. Add one connector and one secret when the job needs them, in a diff a reviewer sees. The `permission` block is the other half: `bash: "*": ask` waits for a person on every command not listed, `git status`, `git diff` and `git log` run without one, and `rm -rf *` never runs.

## Files

| Path | What it holds |
|---|---|
| `kortix.yaml` | The manifest: the agent block, its permission and its grants |
| `agents/feedback.md` | The system prompt, with no frontmatter |
| `examples/feedback.md` | A sample week of feedback |
| `LICENSE.md` | MIT |

## How this differs from a dots setup

OpenAI's dots are an always-on agent in ChatGPT with its own cloud computer, live since 2026-09-29, and the difference is not capability. A dot belongs to one person and only that person can direct it ([workspace controls](https://help.openai.com/en/articles/20001554)); its memory cannot be viewed or edited item by item ([privacy and safety](https://help.openai.com/en/articles/20001529)); enterprise model settings do not apply to it. The record of its work sits in that person's account.

Here the same job runs from a repository the company owns: the rules are one manifest block and one prompt file, read as a diff, and the merge is a permission with a name. Set `model` in the agent block of `kortix.yaml` and the rules stay put.

The longer comparison, with the licence, pricing and self-hosting rows for each open-source option, is on the Kortix blog: https://kortix.com/blog/openai-dots-alternative

## What this is not

Not a hosted product, and not a copy of the dots interface. It needs a Kortix account, the CLI and a git host. The triage quality is the quality of the prompt in `agents/feedback.md`, which you should edit for your own product vocabulary.

## Sources

- Kortix quickstart, read 2026-10-30: https://kortix.com/docs/developers#quickstart
- Agents and permissions, read 2026-10-30: https://kortix.com/docs/guide#agents
- OpenAI dots launch post: https://openai.com/index/introducing-dots
- OpenAI dots workspace controls: https://help.openai.com/en/articles/20001554
