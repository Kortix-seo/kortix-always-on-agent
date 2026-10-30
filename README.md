# kortix-always-on-agent

This is an open-source Kortix project: one agent kept on a steady job. The rules that bound it and the permissions it holds are written into this repository, not into someone's account settings.

A Kortix project is a git repository with a `kortix.yaml` at its root. An agent is one `agents.<name>` block in that file: its description, its prompt file, its per-tool permission and the grants it enables. The prompt itself is a markdown file with no frontmatter, and every session runs in its own sandbox on its own branch. Work reaches the default branch only as a change request, so the question "may this agent do that, and who said so" has an answer a colleague can read later.

## What the agent does

The agent reads a week of customer feedback, groups the items by the problem behind them, and writes one plan per recurring problem. It commits the plans on its session branch and opens a change request for a person to merge. It cannot merge that change request itself, and that is the point of the project. `examples/feedback.md` holds a sample week, so a first run has something real to work on.

## Run it

Install the CLI, sign in, bring this repository into a Kortix project and run the job once. `kortix ship` validates `kortix.yaml`, commits, pushes the current branch and connects the project. With no `origin` it creates and manages the repository. With a GitHub origin it links that repository.

```bash
curl -fsSL https://kortix.com/install | bash
kortix login

git clone https://github.com/Kortix-seo/kortix-always-on-agent feedback-agent
cd feedback-agent
kortix ship

kortix sessions new --prompt "Triage examples/feedback.md and write one plan per recurring problem in plans/"
kortix sessions ls
```

Read what it produced, then merge it or send it back with a note:

```bash
kortix cr ls
kortix cr diff 1
kortix cr merge 1
# or
kortix cr request-changes 1 --message "Split the billing group by plan tier"
```

`kortix doctor` checks the login, the project and a throwaway session end to end if you want to see the whole path work before trusting it. The job is a prompt, so any scheduler you already run can fire it. Run `kortix sessions new` once a week from cron and you get the same agent and the same review step, with no always-on process to keep alive.

## The grant that decides who merges

An agent merges only when its grants include `project.gitops.merge`. This project leaves that name out on purpose, and the whole agent is short enough to check in one screen, quoted from `kortix.yaml`:

```yaml
agents:
  feedback:
    description: >-
      Turns a week of customer feedback into plans a product team can act on,
      one plan per recurring problem, and opens a change request for a person
      to merge. Runs in an open-source Kortix project.
    mode: primary
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

The `kortix_permissions` line is the decision. An omitted grant resolves to none, so with `project.gitops.merge` absent, `kortix cr merge` from this agent's session returns 403 with `agent_scope_insufficient` and a person merges. An admin can grant the capability; this project does not.

The `permission` block is the other half. `bash: "*": ask` holds every shell command that is not listed below it, so `git status`, `git diff` and `git log` run without an answer and everything else waits for a person. `rm -rf *` is denied outright. A tool left out of `permission` is allowed, which is why the ones this job must not do on its own are written down.

The `workspace` line is what puts a checkout of this repository on the session's branch. The absent `connectors` and `secrets` keys mean the agent reaches nothing and holds no credential. Add one connector and one secret when the job actually needs them, in a diff a reviewer sees.

`agents/feedback.md` holds the prompt and nothing else: the job, the order to work in, and the four rules it must not break, one of which is that it commits on its session branch and opens a change request rather than pushing to the default branch.

## What is in the repository

| Path | What it holds |
|---|---|
| `kortix.yaml` | The manifest: the agent, its prompt file, tool permission and grants |
| `agents/feedback.md` | The system prompt, with no frontmatter |
| `examples/feedback.md` | A sample week of feedback to run the job against |
| `LICENSE.md` | MIT |

## How this differs from a dots setup

OpenAI's dots are a serious piece of work: an always-on agent in ChatGPT with its own cloud computer, live since 2026-09-29. The difference is not capability. A dot belongs to one person and only that person can direct it ([workspace controls](https://help.openai.com/en/articles/20001554)), its memory cannot be viewed or edited item by item ([privacy and safety](https://help.openai.com/en/articles/20001529)), and enterprise model settings do not apply to it. The record of its work sits in that person's account.

Here the same job runs from a repository the company owns. The rules are one manifest block and one prompt file, a reviewer reads them as a diff, and the merge is a permission with a name. Set `model` in the agent block of `kortix.yaml` and the rules stay put.

The longer comparison, with the licence, pricing and self-hosting rows for each open-source option, is on the Kortix blog: https://kortix.com/blog/openai-dots-alternative. Where the computer runs, who can direct the agent, what happens to its memory and which step a person keeps are set out side by side in [Kortix against a dot](https://openaidotsalternative.com/openai-dots-vs-kortix).

When a dot stops taking instructions, [the pause message, the Resume button that returns a conflict and the recovery steps that actually work](https://openaidotsalternative.com/openai-dot-paused-cannot-resume) separates what the banner says from what the account still runs. If you are still working out what a dot actually is, [what a dot is and which parts of one you own](https://openaidotsalternative.com/what-are-openai-dots) covers how one is created, what its Custom Rules can be set to, and how much of it travels with you when you leave. The cost has more moving parts than the plan name suggests, and [the pricing at each Pro tier, set against the credits arithmetic of a paid Kortix plan](https://openaidotsalternative.com/openai-dots-cost) works through both. If the plan is to keep the machine, [what self-hosting an always-on agent actually takes](https://openaidotsalternative.com/self-hosted-always-on-agent) sets out the Linux box, the DNS records and the API key that sessions need before the first night.

## What this is not

It is not a hosted product and it is not a copy of the dots interface. It needs a Kortix account, the CLI and a git host. The agent is a prompt plus grants, so the quality of its triage is the quality of the prompt in `agents/feedback.md`, which you should edit for your own product vocabulary.

## Sources

- Kortix quickstart, read 2026-10-30: https://kortix.com/docs/developers#quickstart
- Agents and permissions, read 2026-10-30: https://kortix.com/docs/guide#agents
- OpenAI dots, launch post: https://openai.com/index/introducing-dots
- OpenAI dots workspace controls: https://help.openai.com/en/articles/20001554
