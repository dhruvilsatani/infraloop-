# InfraLoop

**Plan your infrastructure from your project: what's missing, what it costs per month, what to run it on, and how to scale.**

Live: https://infraloop.space (free to try)

InfraLoop scans a project (a repository or uploaded files), asks a few questions about expected load, and produces a report:

- what is missing for production
- a monthly cost estimate for the load you expect
- which infrastructure fits
- how to scale, and what to do first

Your source code and secrets are never sent to an LLM.

## Why I built it

Every time I wanted to take a project to production, I had to stop and ask the same questions:

- Can I run this on **Vercel** or another free service? Will the **best free tier** be enough?
- If it is not a new project and I need to **scale**, do I need a self-managed **Kubernetes** cluster, a managed one, a **self-hosted server**, or **Lambda / serverless**? Which of these is comparable for my case?
- If I have a **compute-heavy task**, do I need a **GPU**, and how much?
- What happens when there is a **database**? How big, where, and how is it backed up?
- What will it cost per month today, and at 10x the traffic?
- What am I missing: monitoring, CI/CD, backups, secrets handling?
- Do I need more than one region, or is that overkill for now?

The answers are scattered across docs, pricing pages and opinions. Then I thought: a good LLM like Claude can answer this. But is it **safe** to hand it the whole project?

I had spent a lot of time building the logic of my projects. A project can also contain **secrets** such as API keys, tokens and passwords, and I don't want to send those to anyone. Leaked keys are a real and recurring problem. Examples that were widely reported:

- **Uber (2016):** attackers found cloud credentials in a private code repository and used them to reach customer and driver data.
- **Toyota (2022):** a contractor's source code containing an access key was publicly available on GitHub for years.
- **Samsung (2023):** engineers pasted internal source code into a public chatbot, and the company then restricted its use.

So I asked: why not build something that gives the **reasoning of AI** without giving up **security**? That became InfraLoop. A deterministic scanner reads the project locally, plain code does the analysis and cost math, and an LLM is optional and only sees the findings, never the code or secrets.

## How it works

```mermaid
flowchart LR
    A[Project files] --> B[Deterministic scanner]
    B --> C[Gap analysis<br/>rule-based]
    C --> D[Adaptive questionnaire<br/>load, budget, team]
    D --> E[Planner and<br/>price engine]
    E --> F[Base report<br/>written by code]
    F --> G[Optional LLM polish]
    G --> H[Claim check<br/>per sentence]
    H -->|passes| I[Final report]
    H -->|fails| F
```

1. **Scanner**: reads project signals (languages, containers, CI, data stores, config) with plain code.
2. **Gap analysis**: rule-based findings, no LLM.
3. **Questionnaire**: asks only what the scan couldn't answer.
4. **Planner and price engine**: deterministic cost and sizing math.
5. **Base report**: written by code, so it is repeatable.
6. **LLM polish (optional)**: improves wording only. Every sentence is checked against the findings. If a sentence fails, the code-written text is used.

## Deployment

```mermaid
flowchart LR
    U[Visitor] --> CF[Cloudflare DNS]
    CF --> CY[Caddy<br/>automatic HTTPS]
    CY --> APP[App container<br/>Podman]
    APP --> GW[LLM gateway<br/>model failover]
    GW --> M[Model providers]
```

- **Currently deployed on Oracle Cloud.** The next major version is planned on **Azure**.
- Caddy is the only service facing the internet.
- The app and the LLM gateway listen on localhost only.
- Per-visitor rate limits protect the public instance.

## Questions and answers

### What problem does it solve?
Before going live, small teams usually don't know what their project is missing (CI/CD, monitoring, backups, secrets handling), what it will cost per month, or what to run it on. Answers are scattered across blog posts and cloud pricing pages. InfraLoop turns one scan and a few questions into one clear plan.

### How is this different from asking an LLM?
A chatbot guesses from whatever you paste in. InfraLoop does the opposite:
- The scan, the gap analysis and the cost math are **plain code**, so the same project gives the same result.
- An LLM is **optional** and only polishes wording. Every sentence it writes is checked against the findings, and if a sentence fails the check, the code-written text is used.
- Your **source code and secrets never go to the LLM**.

### Who is it for?
Developers, students and small teams planning to deploy a project who want a realistic plan and a monthly cost before they spend money.

### Why is there no code in this repository?
The source is private for now, because the tool is still changing quickly. This repository documents what it does and how it is built, and the live tool is free to use, so you can judge it yourself.

### Is it really free?
Yes. InfraLoop is free and will stay free. The public instance runs on a small server, so there are per-visitor hourly limits.

### What do I see on the live site, and is it finished?
What you see at https://infraloop.space is roughly **30% of the planned work**. It already scans, asks questions and produces a plan with costs, and more is coming soon.

### What is planned next?
- Separate **required** and **optional** findings, so small projects aren't told they have gaps they don't need to fix
- A **free or near-free option** for tiny projects
- **Provider-neutral** recommendations, not tied to one cloud
- **Structured JSON export** (findings, costs, gaps) so reports can be compared in CI
- Better scaling advice and a clearer "what to do first" list

### How can I give feedback?
Open an issue in this repository, or try the tool and tell me where the plan or cost estimate looks wrong. That is the most useful feedback.

## Stack

Python, FastAPI, LangGraph, Podman, Caddy, Cloudflare DNS, Oracle Cloud. 177 automated tests.

Built by Dhruvil Satani.
