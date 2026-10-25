# kortix-always-on-agent

This is an open-source Kortix project: one agent kept on a steady job, with the rules that bound it, the connectors it may reach and the record of who let it act all in this repository.

A Kortix project is a git repository with a `kortix.yaml` at its root. An agent's behaviour is a file in `agents/`, its grants sit in the manifest, and every session runs in its own sandbox on its own branch. Work reaches the default branch only as a change request, so the question "may this agent do that, and who said so" has an answer a colleague can read later.

## What the agent does

The agent reads a week of customer feedback, groups the items by the problem behind them, and writes one plan per recurring problem. It commits the plans on its session branch and opens a change request for a person to merge. It cannot merge that change request itself, and that is the point of the project.

`examples/feedback.md` holds a sample week, so a first run has something real to work on.

## Run it

Install the CLI, then bring this repository into a Kortix project:

```bash
curl -fsSL https://kortix.com/install | bash
kortix login

git clone https://github.com/ArtemShatokhin/kortix-always-on-agent feedback-agent
cd feedback-agent
kortix ship
```

`kortix ship` validates `kortix.yaml`, commits, pushes and links the repository to your Kortix account. It creates a Kortix-managed repository when the clone has no origin and links the existing one when it has.

Start a session with the job:

```bash
kortix sessions new --prompt "Triage examples/feedback.md and write one plan per recurring problem in plans/"
kortix sessions ls
```

Read what it produced, then merge it or send it back:

```bash
kortix cr ls
kortix cr diff 1
kortix cr merge 1
# or
kortix cr request-changes 1 --message "Split the billing group by plan tier"
```

`kortix doctor` checks the login, the project and a throwaway session end to end if you want to see the whole path work before trusting it.

The job is a prompt, so any scheduler you already run can fire it. Running `kortix sessions new` once a week from cron is the same agent and the same review step, with no always-on process to keep alive.

## The grant that decides who merges

An agent merges only when its `kortix_permissions` include `project.gitops.merge`. Without it, `kortix cr merge` returns 403 with `agent_scope_insufficient`, and a person merges. This project leaves that permission out on purpose:

```yaml
kortix_version: 2
default_agent: feedback

agents:
  feedback:
    file: agents/feedback.md
    skills: [kortix-cli]
    kortix_permissions: [project.session.read, project.review.submit]
```

An omitted grant resolves to none, so the missing `connectors` and `secrets` keys mean the agent reaches nothing and holds no credential. Add one connector and one secret when the job actually needs them, in a diff a reviewer sees.

The agent file holds the other half of that argument, a tool policy narrower than the default:

```yaml
permission:
  edit: allow
  bash:
    "*": ask
    "git status": allow
    "git diff *": allow
    "git log *": allow
    "rm -rf *": deny
```

A shell command that is not on the allow list waits for a person. That is how a project stays honest about what an agent may do without asking.

## What is in the repository

| Path | What it holds |
|---|---|
| `kortix.yaml` | The manifest: the default agent and its grants |
| `agents/feedback.md` | The agent's description, tool policy and system prompt |
| `examples/feedback.md` | A sample week of feedback to run the job against |
| `LICENSE.md` | MIT |

## How this differs from a dots setup

OpenAI's dots are a serious piece of work: an always-on agent in ChatGPT with its own cloud computer, live since 2026-09-29. The difference is not capability. A dot belongs to one person and only that person can direct it ([workspace controls](https://help.openai.com/en/articles/20001554)), its memory cannot be viewed or edited item by item ([privacy and safety](https://help.openai.com/en/articles/20001529)), and enterprise model settings do not apply to it. The record of its work sits in that person's account.

Here the same job runs from a repository the company owns. The rules are two files, a reviewer reads them as a diff, and the merge is a permission with a name. Swap the model in `agents/feedback.md` and the rules stay put.

The longer comparison, with the licence, pricing and self-hosting rows for each open-source option, is on the Kortix blog: https://kortix.com/blog/openai-dots-alternative

The two side by side, row by row — where the computer runs, who can direct the agent, what happens to its memory, and which step a person keeps — is [Kortix against a dot](https://openaidotsalternative.com/openai-dots-vs-kortix).

When a dot stops taking instructions, [the pause message, the Resume button that returns a conflict and the recovery steps that actually work](https://openaidotsalternative.com/openai-dot-paused-cannot-resume) separates what the banner says from what the account still runs. If you are still working out what a dot actually is, [what a dot is and which parts of one you own](https://openaidotsalternative.com/what-are-openai-dots) covers how one is created, what its Custom Rules can be set to, and how much of it travels with you when you leave. The cost has more moving parts than the plan name suggests, and [the pricing at each Pro tier, set against the credits arithmetic of a paid Kortix plan](https://openaidotsalternative.com/openai-dots-cost) works through both. If the plan is to keep the machine, [what self-hosting an always-on agent actually takes](https://openaidotsalternative.com/self-hosted-always-on-agent) sets out the Linux box, the DNS records and the API key that sessions need before the first night.

## What this is not

It is not a hosted product and it is not a copy of the dots interface. It needs a Kortix account, the CLI and a git host. The agent is a prompt plus grants, so the quality of its triage is the quality of the prompt in `agents/feedback.md`, which you should edit for your own product vocabulary.

## Sources

- Kortix quickstart, read 2026-10-21: https://kortix.com/docs/quickstart
- Agents and permissions, read 2026-10-21: https://kortix.com/docs/project/agents
- OpenAI dots, launch post: https://openai.com/index/introducing-dots
- OpenAI dots workspace controls: https://help.openai.com/en/articles/20001554
