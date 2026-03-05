# The EasyWay Manifesto

## We started building a data portal. We ended up somewhere else.

Like every good book, we knew the first chapter but not the ending. Chapter one was simple: build a web portal where small businesses could upload Excel files, connect FattureInCloud, see dashboards, understand their numbers. A data platform for people who aren't data engineers.

That was the plan.

Then something happened.

We started using AI agents — not as a novelty, not as a demo, but as real collaborators. Session after session, day after day, we found ourselves solving problems we didn't expect. How do you give an AI agent a conscience? How do you let it create pull requests without losing control? How do you maintain context across 75 sessions? How do you separate what needs intelligence from what needs precision?

The portal is still being built. But along the way, we accidentally built something bigger: **a methodology for building software with AI agents, documented with the rigor of a lab notebook and the honesty of a diary**.

Every failure is recorded. Every lesson learned has a name and a session number. Every architectural decision has a GEDI case explaining why. Not "best practices" copied from a blog — battle scars from production.

## The uncomfortable truth

The industry tells you this stuff is hard. That you need Kubernetes clusters, enterprise licenses, dedicated DevOps teams, six-figure cloud bills. That schema governance requires Atlas or Liquibase with enterprise support. That ETL needs Airflow on a managed Kubernetes cluster. That secrets scanning needs a $50k/year GitGuardian subscription.

We built all of this on a **free ARM server from Oracle Cloud**. 4 cores, 24GB RAM. With bash scripts, PowerShell, Docker Compose, and common sense.

We're not sending shuttles to the moon. We do simple things wearing a false mask of complexity. The frameworks we build are grandma's good advice — packaged in a way that makes them reusable.

When they tell you it's not simple, they just don't want to tell you it's only a matter of knowledge. Once you know what to do, everything is easy.

This project exists to prove it. **For everyone — especially for those who can't afford the enterprise price tag.**

## What we believe

**The AI doesn't replace human judgment. It amplifies it.** GEDI — our ethical decision framework — never blocks. It illuminates. The agent proposes, the human approves. Always.

**Start private, earn your way public.** Our Three Circles model: first it works for us (Circle 3), then we open the source (Circle 2), then we give it to the world (Circle 1). No project skips levels. Maturity is earned, not declared.

**Deterministic and non-deterministic don't mix.** The brain (AI) and the muscles (engines) stay separate. HALE-BOPP does schema validation, ETL, and policy gating with zero AI. EasyWay adds intelligence on top. Each layer is testable, replaceable, and honest about what it is.

**You don't need permission to build.** A free server, an open-source database, an AI collaborator, and discipline. That's it. The rest is knowledge — and knowledge wants to be shared.

**Documentation is not overhead. It's the product.** Our chronicles, session logs, GEDI casebook, and wiki are not side effects of development — they ARE the development. If you can't recreate a decision from the docs, the decision doesn't exist.

**Non fuffa, evidenze concrete.** Not hype, concrete evidence. Every tool we release has been tested on our own project first. Every example in the docs is a real command we ran on real data. Every lesson learned comes from a real mistake.

## What we actually built

We set out to build one product. We ended up with ten.

Some were planned. Most were accidents — tools we built to solve our own problems, that turned out to be useful for everyone. And all of it running on a server that costs zero.

See [Product Map](product-map.md) for the full catalog, with honest competitive analysis.

## The story so far

- **Sessions 1-20**: Built the portal. Learned Node.js, Docker, SQL Edge, API design.
- **Sessions 20-40**: Built the agents. L2 runners, L3 orchestrators, skills registry, knowledge API.
- **Sessions 40-55**: Built the governance. GEDI, PAT routing, Iron Dome, agentic SDLC.
- **Sessions 53-62**: Built the factory. Polyrepo migration, 8 repos, GitHub mirror, deploy pipeline.
- **Sessions 62-74**: Built the cathedral. ADO backlog restructured, HALE-BOPP launched, connection registry.
- **Session 75**: Stopped to look at what we'd built. Realized it was more than a portal.

The book isn't finished. But the characters have taken over, and they know where they're going.

---

*Started in 2025. Still going. Every session documented.*
*Built by a human and an AI, in honest collaboration.*
