---
title: "Building Shuffled: Treating AI as a Team"
date: "2026-02-10"
author: William Mora
assets_url: /assets/building-shuffled-treating-ai-as-a-team
tags:
- ai
- claude code
- codex
- gemini
- side project
- shuffled
---
*Also published on [Substack](https://itcouldalwaysbeworse.substack.com/p/building-shuffled-treating-ai-as).*

I wanted to run a small experiment: could I build and ship a daily word puzzle game in a few days, using AI as my entire team, to get a better sense of building an idea with today’s AI tools?

<!--more-->

[Here’s the result](https://shuffled.app/). Read along for how it went.

## The idea

I’ve had the idea of a simple game for a while, inspired by other word and casual games: something you pick up once a day, solve in a few minutes, pick up a new word, and move on until the next day. Just a small, fun way to clear your mind. [Shuffled](https://shuffled.app) is my take on it, where you unscramble letters in a grid to form words before running out of moves; everyone gets the same puzzles daily.

This was the perfect kind of project for an “AI-assisted sprint”:

- Something I’d actually play myself (see above)
- Small and shippable: a web app, no game engine, no heavy backend (yet), simple mechanics
- A stack I already know, so I can focus on the “AI team” workflow instead of new tech
- Something I could write about at the end (this post!)

## The stack

### Tech stack

Standard stack for a web app: Next.js, TypeScript, Tailwind, deployed on Railway with Cloudflare as DNS provider, [PostHog](https://posthog.com) for analytics.

### Development stack

I had subscriptions to Claude Pro, ChatGPT Plus, and Gemini AI Plus. While I find Opus to be the most capable model for how I work, I do like the experience of using different models for different problems, depending on complexity. Also, I considered this setup as a hiring budget for a side project 🤓.

To control cost and match strengths, I gave each tool a role:

- [Claude Code](https://code.claude.com/docs/en/overview), my pair programmer: used for hands-on keyboard work, planning complex work, and building features that required a tight feedback loop. I think of Claude Code as my most senior developer whom I want to work closely with to build my ideas
- [Codex](https://openai.com/codex/), my task runner: used for delegating relatively low-complexity or well-defined work that I can confidently trust to be done with almost no back-and-forth. Think specific visual polish, documentation upkeep, straightforward bug fixes that I ran into on the go, etc
- [Gemini Code Assist](https://developers.google.com/gemini-code-assist/docs/review-github-code), my code reviewer: since I predominantly use Claude and Codex for coding tasks, I set up Gemini as my sole code reviewer with out-of-the-box settings. I found Gemini was really good at flagging Critical and High issues in PRs (more on addressing PR comments later)
- For everything else, eg: brainstorming, alternate perspectives, or rate-limit moments, I’d rotate between them (including the Codex and Gemini CLIs)

Beyond the models, here’s the rest of the development stack:

- [SonarQube](https://sonarcloud.io) to ensure good overall code quality before shipping any changes. This is a must for any new project past the prototype phase, especially with more AI-generated code
- [GitHub Actions](https://github.com/features/actions) to enforce good CI/CD hygiene, including SonarQube analyses
- [Playwright](https://playwright.dev/) for end-to-end testing
- [Railway environments](https://docs.railway.com/environments) for:

- Production (obviously). What you see when you open [Shuffled](https://shuffled.app)
- [PR environments](https://docs.railway.com/environments#pr-environments-1), for testing changes manually before shipping to production. Railway automatically spins up the environment and posts the URL in the PR. Extremely convenient in my workflow and dead simple to set up. 10/10 would recommend

- [VSCode](https://code.visualstudio.com/) as my IDE

### Planning upfront: Early decisions matter

I spent a significant amount of time upfront crafting specs I was happy with. Previous experience with an AI-assisted project taught me that you need to be opinionated about boundaries: API contracts, the game flow, the rules. AI handles internal logic well enough, but core mechanics and requirements should be nailed down by a human (for now?) who understands the feel they’re going for.

I was also very intentional about the design. I even hand-sketched the layout:

![Hand-drawn sketch of the Shuffled layout]({{ page.assets_url }}/layout-sketch.jpeg){:width="420px"}

*Don’t hire me for art*

This is something I’ll keep doing even for quick prototypes (well, maybe not the drawing). Skipping a good planning step would’ve cost me later and likely slow me down when I actually want to iterate fast.

## Skills and subagents: “A-ha” moments

Through this project, I had a more relatable use for bringing [skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) and [subagents](https://code.claude.com/docs/en/sub-agents) into a project than I had tried in the past. I’m sure I’m only scratching the surface here, but I can definitely appreciate how much more productive they made me.

### Skills

Once I got the first working prototype running, I started a more regular development workflow, which meant I began encountering recurring tasks I could turn into [skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview). The most time-saving skills I had for this project were:

- Commit and open a PR: Commit pending changes, push to the current branch, and open a pull request on GitHub, with some guardrails in between in case there are git conflicts, wrong branch, etc
- Review PR: For every review comment on the PR for the current branch, classify it as **Address** or **Skip** with a brief rationale, plan fixes, wait for my confirmation, and implement them. The core PR loop was essentially: Open PR → wait for Gemini’s review → run this skill

Other housekeeping skills that are less frequently used but still handy to have:

- Refresh documentation and memory files. Tl;dr verify documented claims against the codebase and workflows, and update any stale information
- Debug failing tests
- Fix lint issues

One neat thing about skills is that they are shared among Claude, Codex and Gemini. Another thing: I found that having the models create and update the skills themselves was much more efficient than attempting to do it myself.

### Subagents

I wanted the game to feel like it has an infinite number of puzzles. In order to do that, you need the math and constraints to support it (eg, 5x5 word square puzzles are hard to generate without repetition). I also wanted a scalable way to generate new puzzles that I could still control for the most part. This was a feature that needed to be well thought out; otherwise, I’d run into headaches down the road, including spending a lot of time on content moderation, which was not ideal. This is how I introduced my first subagent, given that they are well-suited to task-specific workflows.

I created (well, Claude did) an *architecture-advisor* agent. In a nutshell, I use it when I need guidance on architectural decisions, system design trade-offs, code organization strategies, or long-term maintainability planning. I won’t go into the details, but this significantly helped me get to a solution and a foundation for puzzle generation that I was satisfied with. More importantly, it made me realize: *whoa,* *I can orchestrate a development team with different disciplines.*

Since then, I added two agents I used heavily during this sprint:

- *game-design-advisor*: used when I’m thinking of improving a game mechanic, expanding on game design, addressing user feedback about gameplay experience, or brainstorming a new game feature
- *test-strategist*: this one I find extremely interesting and one I’ll keep testing (no pun intended), so sharing the current subagent description:

```
You are an elite software testing strategist with deep expertise in test architecture, coverage optimization, and quality engineering. You specialize in TypeScript/React applications and understand that the goal of testing is to maximize development velocity while preserving critical user experiences. You think in terms of risk-based testing: protecting what matters most while avoiding test bloat that slows teams down.
```

### Development workflow

By now, the flow looks somewhat like this:

1. Planning/specs/decisions with Claude (and subagents). I even created a roadmap file broken down into phases, calling out which subagent is needed on each phase
1. Delegate small tasks to Codex asynchronously. My rule of thumb is that if I don’t feel confident that the task can be easily solved in one shot, I’d rather wait until I’m available to pair program
1. Pair program with Claude Code for features, more complex work, ensure test coverage remains solid, and test locally. Then push PRs and test in a PR environment. Small commits to iterate fast and safely are still a golden practice.
1. Once PR is up, Gemini reviews PRs
1. If review risk is low + PR env looks good → merge
1. SonarQube flags, or if the PR comments are signaling moderate risk, get fed back into whichever agent is best suited to address in a pair programming session

### What was my role in all of this?

This is something I wanted to figure out during this experiment. How is my role as a developer evolving given everything available today? By now, I have close to 20 years of experience building and maintaining software, as well as leading Engineering teams of 80 people. While I’m aware of highly autonomous AI workflows and I’m all in on AI, I’m also approaching this as closely as a realistic, team-based development workflow could look, based on my own development approach. Vibe coding is fun; accountability is better.

To answer the question, AI primarily handles execution, while I provide direction, taste, and risk control. While AI’s throughput is excellent, I had an opinionated vision of what I wanted done. I was the orchestrator of everything happening. I needed to be able to follow and understand the code well enough, manually intervene occasionally, and assess PR comments. I also need to understand and evaluate technology choices, etc. The Railway + Cloudflare deployment setup, while straightforward, still required me to serve as the orchestrator and architect (though I’m sure this could be streamlined).

More importantly, I am the one continuously testing the game’s *feel* and its direction. Is it snappy enough? Is it fun? Can I actually read the letters, even if based on good design patterns? Is it what I envisioned? If not, am I still happy with it? Then there are other aspects of building the game beyond coding: I shared the game with friends and family. I am still collecting feedback, triaging it, and deciding what matters. Focus and time are limited, so building with a pragmatic approach remains relevant and is where I find myself most involved.

I shipped a shareable version in 2 days, and it hit ~100 unique users within a week. Not bad!

## What’s next

I was going to stop after the experiment sprint, but the positive feedback and engagement made me keep going and add a few more things to Shuffled. Next up: a Spanish version of the game (Update: [it’s up](https://shuffled.app/es)!).

Ultimately, I ran this experiment to get a better feel of what to expect as I build moving forward. The future is exciting for all the ideas we’ll be able to bring to life. Other people working with similar workflows will find their own way to fully own a service through a team of AI agents. An evolution for development teams could be that a single product engineer owns a much larger scope through their AI team than before. While I'm not a fan of fragmented codebases, orchestrating well-scoped, smaller codebases might also be a safer approach to unlocking velocity through AI. I may explore some of these thoughts in future experiments.

---

**If this resonates with something you’re building or thinking about, I’d love to hear about it.** Got questions about anything here? Happy to chat about it!

And the shameless plug: **[play Shuffled](https://shuffled.app/)** and let me know what you think 🙂
