---
title: "Where AI Actually Fits (and What It Means for Software Jobs)"
date: 2026-08-20T18:00:00Z
draft: false
tags:
  - AI
  - Software Engineering
  - Careers
  - Economics
---

There is an old line about the software industry: **cloud eats open source, open source eats software.** Each layer commoditizes the one below it, and value moves up. Open source made writing code less of a differentiator. Cloud made running software into a paid service. Every generation, whatever was the moat becomes the utility.

Naturally people expect AI to be the next step. Something that eats cloud the way cloud ate open source. But sit with it for a minute and the fit is wrong. AI is not quite the next thing on the ladder. It does something stranger. It bends every rung at once.

This post is about what AI actually is in that picture, and then, because I keep getting asked about it, what the labor data really says about software jobs.

---

## The ladder before AI

The tidy version of the last twenty years:

```
                    value moves up the stack
                                 |
                                 v

  BUY SOFTWARE          <-- open source ate this
  HOST SOFTWARE         <-- cloud ate this
  BUILD ON HOSTED SW    <-- where value has been living
```

Each layer commoditized a *category of software*. Open source made the code layer a public good. Cloud made the operations layer a service. In both cases the thing that got eaten was a piece of the stack, and the value moved one step higher.

If AI were the next rung, you would expect it to eat some layer of software the same way. It does not. That is where the analogy strains.

---

## AI does not sit on the ladder. It bends it.

AI is not a new layer on the stack. It is a capability that touches every layer at the same time, in different directions.

```
                          CLOUD
                     (feeds it: chips,
                      GPUs, datacenters,
                      inference demand)
                          ^
                          |
                          |
   OPEN SOURCE   <---    AI    --->   THE APP LAYER
   (eats it: open        |            (collapses it:
    weights repeat       |             "buy it," "host
    "OSS eats SW"        |             it," "fork the
    one level up)        v             OSS version" all
                     THE JOB LEVEL     blur into
                     (chews mid-       "describe what
                      level tasks      you want")
                      inside every
                      job)
```

Look at each direction:

- **AI feeds cloud.** Every inference call, every training run, every long-context prompt is compute someone has to sell you. That is why the hyperscalers were the first clear winners. Whatever else happens, more AI means more cloud.
- **Open source eats AI back.** The same pattern that made open source eat proprietary software is replaying on models. Open weights are catching up on capability faster than most people expected, which pushes value away from the model itself and out toward data, distribution, and the application layer.
- **AI collapses the app layer.** The old choice between "buy the SaaS," "host it yourself," or "fork the open source alternative" blurs when you can just describe what you want and have it built. That is not fully here, but the direction is clear.
- **AI chews the middle of jobs.** Not entire roles, not entire industries, but the mid-level tasks that sit inside almost every knowledge job. This is the part the labor data starts to show up in.

**One honest caveat.** The framing is clean and probably too clean. AI's economics are still unsettled. Whether value accrues most to models, to infrastructure, or to applications is the real open question, and the answer is not in yet. The ladder metaphor bends nicely on the page. Real economies are messier.

---

## What actually gets commoditized

Here is the way I think about it now. The other layers each commoditized a *thing*. Code. Ops. AI commoditizes a *verb*. **The act of producing.** Writing the code. Drafting the memo. Generating the design. Deciding the routine call.

That reframing matters, because it explains why AI does not map neatly onto job titles. It maps onto tasks within jobs. In every profession there is a spectrum from routine production at one end to non-routine judgment at the other. AI pushes hardest on the middle of that spectrum. The routine end shrinks. The judgment end gets leveraged.

Which brings us to the actual question everyone wants an answer to.

---

## What the labor data says (the honest version)

There are a lot of scary headlines. The data underneath is more mixed than the headlines suggest.

**The big picture, through early 2026:**

- Most deployments so far are **augmentative, not autonomous.** Only about **22% of AI projects target a fully autonomous end state.** (S&P Global, June 2026)
- Firms report a **slightly negative** net employment effect in the past year. The balance of firms reporting job losses ran about **5 percentage points higher** than those reporting gains. S&P's prior 2025 report had this near zero, so this is a reversal, though modest.
- Adoption is wide but shallow. Across 38 use cases, current adoption averages ~50% and planned adoption ~37%. Top uses: summarization (71%), translation (62%), data management (61%).
- **No aggregate unemployment spike for AI-exposed workers.** An Anthropic labor study (March 2026) and independent IMF and Stanford analyses all found the change in unemployment gap between AI-exposed and insulated workers **small and not statistically significant** through late 2022 to early 2026.
- Skills demand is repricing ahead of losses. AI skills appear in **~2.5% of US job postings, up 55% year over year.** Agentic-AI skill mentions are up **more than 280% in a year.** (Stanford AI Index 2026)

The rough shape: augmentation dominates, aggregate employment holds up, but real damage is concentrated at the low-skill and routine end. The best causal study I have seen on this (Marguerit, LISER, March 2025) found:

- A 1 standard deviation rise in **automation-AI exposure cuts hourly wages by ~7.7%.**
- A 1 standard deviation rise in **augmentation-AI exposure raises employment by ~3.1%.**
- The wage damage lands hardest on low-skilled occupations. The employment upside lands hardest on high-skilled ones, especially STEM.

The uncomfortable summary: **AI so far has widened inequality without breaking the aggregate labor market.**

**And the load-bearing caveat that has to sit under all of this:** every one of these numbers measures the tools people actually used through 2024 and early 2026. Frontier models and real agents are arriving now, and they target exactly the mid-level "implement this spec" work that has held up best so far. The data we have is the last chapter of the pre-agent era, not the first chapter of what comes next.

---

## The SWE story: seniority is the real dividing line

Now the part that made me sit up. The single most striking data point in the notes I gathered is from the Stanford Digital Economy Lab:

- Software developers aged **22 to 25: employment down ~20%** from the late-2022 peak by mid-2025.
- Developers aged **30 and older, in the same AI-exposed category: employment grew 6 to 12%** over the same period.

Same technology. Opposite outcomes. Sorted almost entirely by seniority.

```
Software engineer employment change, late 2022 to mid 2025
(within the AI-exposed cohort)

Ages 22-25    ###################|                       -20%
                                 | 0
Ages 30+                         |######## +6% to +12%
```

Supporting evidence from the recruiting side:

- Indeed Hiring Lab: senior tech titles down 19% vs five years earlier. **Junior titles down 34%.** Seniors relatively better off, not untouched.
- Junior hiring is down 25 to 35% depending on the source. New grads make up only **7% of Big Tech hires now, versus 32% in 2019.** Bootcamp enrollment is down about **40%.**

The mechanism people keep pointing to is leverage: **one senior engineer with AI tools can do the work that previously required 3 or 4 juniors.** So the same demand for output supports fewer bodies at the bottom of the pyramid.

The role itself is shifting too. Engineers are becoming, in one useful phrase, **"directors of intelligent systems."** Defining the problem, choosing the architecture, writing precise prompts, and rigorously evaluating what the model produced. The manual keystroke part shrinks. The judgment part grows.

---

## The paradox to resolve

Here is the confusing bit. The BLS projects **17% employment growth for software engineers through 2033**, an addition of about 327,900 US jobs. Meanwhile junior employment is falling right now. Both are true.

The way to reconcile it is not "one of these must be wrong." It is that **the nature of the jobs is changing faster than the number of jobs.** The sector keeps growing. What "a software job" means is being redefined in real time.

Two more caveats worth carrying, because they matter:

1. Some of the "senior safety" is macro, not AI. Higher rates and a post-2021 over-hiring correction have layered on top of the technological shift. If rates ease and hiring loosens, junior demand could come back some. So it is not a pure AI signal.
2. Mid-level is not uniformly safe. Junior and mid-level generalist frontend work has been called out as one of the highest-risk categories right now.

---

## Sub-specialty breakdown, most insulated to most exposed

The organizing principle is simple: **the further you are from "generate well-patterned application code," the safer.** That is what models do best, so that is what gets pressed on hardest.

```
MOST INSULATED  <-------------------------------------->  MOST EXPOSED

Infra / systems for AI workloads
Security / DevSecOps
ML / AI engineering
Platform / DevOps / SRE
                                Backend (safe if specialized)
                                Frontend (safe if specialized)
                                Full-stack (safe if AI-native)
                                                    Generalist anything
```

**Holding up strongest:**

- **Infra and systems for AI workloads.** GPU clusters, inference pipelines, distributed training infrastructure. Deep expertise that current AI tools cannot replicate. Cloud engineer is the top-paid slot (~$153K median).
- **Security and DevSecOps.** US listings up 124% year over year in 2025. AI makes threats faster; humans who understand AI attack vectors are in demand.
- **ML and AI engineering.** ~63% talent shortage, roughly 3.4 open AI roles per qualified candidate. ManpowerGroup calls AI skills the hardest to hire for globally, for the first time. Salary premium 20 to 40% above median SWE.
- **Platform, DevOps, SRE.** Hard to fill, fed by the enormous datacenter buildout.

**Middle: safe if specialized, exposed if generalist:**

- **Backend.** Solid but the profile has shifted. Distributed systems and AI integration are in demand. Generic backend without cloud depth faces more competition.
- **Frontend.** The most bifurcated. Performance, accessibility, and design-systems specialists hold ground. Generalists are high-risk.
- **Full-stack.** Softening precisely *because* AI extends individual reach. The survivable version is AI-native product work (streaming APIs, LLM backends).

**Most exposed:**

- **Generalist anything.** This is the real dividing line. Interchangeable generalists are the profile whose output a model reproduces cheapest. Deep specialists get multiple offers within weeks. Generalists send hundreds of applications.

**Meta-pattern in one line:** winners are AI-adjacent (build, deploy, secure the AI), infra-deep (the layer AI cannot abstract away), or specialist-deep (a proof surface a generalist cannot fake).

---

## Company size, and where cuts actually happen

One detail from the S&P Global report that changed how I read the aggregate numbers: the pain is not evenly distributed across firm sizes.

```
Net employment impact forecast for 2026 by firm size (S&P Global)

Large firms (10,000+ employees)   ############|              -13
Medium firms                                   |##            +2
Small firms                                    |###           +3
                                     ---------|0-----------
```

Large firms both cut more and formalize AI more (44% of the 10,000+ employee organizations have a documented AI strategy with dedicated AI career paths). Small firms are, on net, still growing.

The catch is attribution. Of the S&P Global 1200, **83% had lower headcount in January 2026 than in January 2025.** So most cuts genuinely cannot be pinned on AI. Other factors, from cost pressure to rate levels to a general belt-tightening, matter as much or more. The AI-attributable share is a slice of a larger contraction, not the whole thing.

There is also a trust wrinkle. Only **16% of firms completely trust third-party AI models in 2026, down from 24% in 2023.** Only 46% of past-year AI initiatives are on track for ROI within 12 months. Only 37% are live and delivering value. So the AI story is not "everyone finished migrating and now cuts jobs." It is "everyone is still figuring it out, cutting jobs anyway, and hoping the tools land."

---

## What I take from this

1. **AI is not the next rung on the ladder. It bends every rung.** Feeds cloud, gets eaten by open source, collapses the app layer, chews the middle of jobs. Any single-layer explanation misses most of what it is doing.
2. **The thing being commoditized is the act of producing,** not any particular category of software. That is why it does not map onto occupations, it maps onto tasks inside occupations.
3. **The aggregate labor picture is calmer than the headlines.** No big unemployment spike, adoption is real but shallow, most deployments are augmentative. The damage is concentrated, not general.
4. **The concentration is on the routine and the entry-level.** Wages down for low-skill occupations, employment up for high-skill augmented ones. Widening inequality without an aggregate crash.
5. **In software, the real cut runs by seniority, not by role.** Juniors down ~20% since late 2022; seniors up 6 to 12%. That is one of the clearest labor patterns I have seen in years.
6. **The paradox is not a contradiction.** Sector employment still grows ~17% through 2033 per BLS. What a software job means is changing faster than how many software jobs exist.
7. **Specialize or dissolve is the honest advice.** Deep infrastructure, security, ML, or genuine product judgment on top of AI-native tooling. Interchangeable generalist roles are the ones AI reproduces cheapest.
8. **The scariest caveat is that all the data is pre-agent.** Every number in this post measures tools people actually used through early 2026. The next wave targets exactly the "implement this spec" middle that has held up best so far. Come back to this in a year.

The tidy version of the ladder was useful for twenty years. It is not useful for the next ten. AI is not the next thing that eats a layer. It is the thing that changes what a layer even is.

---

*Sources include S&P Global's* AI and Labor Landscape 2026, *Marguerit's LISER working paper* Augmenting or Automating Labor?, *the Stanford AI Index 2026 and Digital Economy Lab, an Anthropic labor study (Massenkoff and McCrory, March 2026), BLS occupational projections, BCG, ManpowerGroup, Indeed Hiring Lab, and Lightcast. This is a personal read of the data. It is not career or investment advice. The picture will look different in a year, in ways nobody can call yet.*
