# Product Management at Vervian

> Owner: Vervian · Last reviewed: 2026-06-05
>
> **Status:** First draft. A distillation of Vervian's product principles brought in from past teams, set up as a starting position for Vervian. Adoption happens through the work, not through a memo.
>
> This guide is **not a methodology**. It is the shared vocabulary and set of mental models Vervian product work runs on. Take what's useful, leave what's not.

---

## Why this document exists

Vervian is a small studio shipping a portfolio of focused products (see `vision.md`). That portfolio only works if we stay disciplined about *what* we build and *why*, not just *how fast*. Discipline about what and why is what keeps quality high when the team is small. This guide captures the product-management posture that makes that discipline real.

It is downstream of `core-beliefs.md` and serves `vision.md`. When this guide contradicts either, this guide is wrong.

---

## Principles

These principles guide how we make product and design decisions at Vervian. They are the starting position Vervian brings in from past product teams. Some are paraphrased from common product/design wisdom; the formulation here is the one Vervian is adopting.

### Know the user's job intimately

Never create designs that don't solve a customer problem. Focus on outcomes. Form follows function. Create for convenience. Help users the way *they* want to be helped, not the way that flatters us. **You are not your user**, and any time we forget that we ship the wrong thing.

### Less, but better

Prioritise simplicity and intuitiveness throughout the product to minimise cognitive load. We hold to a three-click mentality where it makes sense. If the user has to dig, the design has a job to do.

**Concentration is more important than choice.** Configurations are distracting and difficult to support; use them sparingly, and make the choices that best meet the customer's needs so they don't have to.

### Don't reinvent the wheel

Be consistent. Leverage best practices. Think in systems. Where established UX fundamentals exist, we follow them. We save our originality for the places it actually moves the needle. Reinventing the obvious is a tax on the user.

### Quality is not up for negotiation

Shipping beats perfection, **but quality is non-negotiable.** We don't cause unnecessary harm. We leave the product better than we found it. We create **reversible designs** so customers can correct mistakes easily where needed.

### Understand Why, always

Start with Why. If five whys can't explain why we're suggesting to build it, we probably shouldn't build it. (See also: `vision.md` and the commitment to written, fact-based justification for every major product move in `core-beliefs.md`.)

### Great products require great collaboration

There is no I in team. We are not design heroes. We seek feedback from our peers regularly because the work gets better when it's seen by other eyes. **Make product design a priority.** We act as design coaches with the ambition to make Vervian and our craft better, not to build personal portfolios.

### Form follows function, with intent

We create with intent, and we back our decisions with data and insights where we can. We combine **empathetic** and **analytical** approaches; neither alone gets us all the way. Empathy tells us what to solve for; analysis tells us whether we did.

### Speak the customer's language

We use language that's easy to understand. *"Yes, delete this message"* beats *"Yes."* We don't let marketing copy creep into the product. The only copy welcome inside is the copy that clearly communicates situations and options to the user.

### We are storytellers

When asked where we're going, we tell a story rather than point a finger. Storytelling isn't fluff. It's the discipline of making the direction memorable enough that the team can use it to settle tradeoffs without coming back to ask. (Expanded in [Storytelling](#storytelling) below.)

---

> **Design is our superpower.** Used well, pointed at real customer problems, backed by data and feedback, executed with quality, it is the lever a small studio can pull to ship products that punch above their weight.

---

## The five questions every Vervian initiative should answer

Before an initiative moves from "we should do this" to "we are doing this," we can answer each of these, or name which one is still open.

1. **The Why.** One sentence: "We exist to [outcome] for [who] because [why it matters]." If we can't say it, we're just moving tickets.
2. **The job.** What is the customer hiring this product to do? Not the feature; the *job*, the outcome they want, the alternative they'd use instead.
3. **The risks we're taking.** Of the four (Value, Usability, Feasibility, Business Viability, per Cagan), which ones is this initiative most exposed to? Which are addressed; which are still bets?
4. **The driver we're moving.** Which specific node on the revenue/value tree does this work move? Not the metric, the *driver*. "Improving onboarding" is a task; "improving activation among new users in their first session" is a bet.
5. **How we'll know.** What number, observed when, tells us this worked? If we can't answer this, we ship instrumentation before we ship the feature. (See `core-beliefs.md` and `vision.md` on objective measurement.)

If any one of these is genuinely "TBD," that's fine. Name it and treat it as the next thing to resolve.

---

## Mental models we use

A short catalogue. Use these as a shared vocabulary in product reviews and architecture discussions.

### Outcomes over output

Shipping ten features nobody uses is still a failure. Output is what gets done. Outcome is what changes for the customer. The Build Trap (Perri) is when a team optimizes for the first and ignores the second: backlogs full, sprints running, features shipping, nothing moving. Catch it early.

### The PMF Pyramid (Olsen)

Target customer, then underserved needs, then value proposition, then feature set, then UX. Each layer rests on the one below. **When a product isn't working, look down, not up.** The fix is almost always deeper than the layer you're staring at.

### The business equation (Olsen)

Revenue is a tree, not a number. Customers × revenue per customer. New customers vs retained. Each branch is a lever someone can move. Every Vervian product owns a node on the tree. If you can't draw the line from your week's work to a node on the tree, you're guessing.

### The four big risks (Cagan)

| Risk | Question | Where teams skip it |
| :--- | :--- | :--- |
| Value | Will customers buy / switch / care? | Confused with "we like it." |
| Usability | Can the user figure it out without a manual? | Discovered in support, not design. |
| Feasibility | Can we build it with what we have? | The risk most teams *over*-address. |
| Business Viability | Does Legal, Sales, Finance, Support work? | The risk that kills products *after* they ship. |

Cagan's argument: best teams de-risk all four before writing production code. Feasibility alone is not validation.

### Discovery vs Delivery (dual track)

Two parallel workstreams. Discovery is "building the right thing"; delivery is "building the thing right." They run in parallel, not sequentially. Discovery feeds delivery validated ideas; delivery feeds discovery real-world data.

> "Minimum viable everything", the cheapest, fastest way to learn whether an idea has legs. A prototype, a mock, a concierge test, a landing page. Anything that generates evidence before engineering capacity is committed.

### Missionaries, not mercenaries (Cagan)

Hire smart people to solve problems, not follow a roadmap. Give them objectives and constraints, not instructions. A team that understands the Why finds better solutions than a team executing a spec they had no hand in shaping.

Vervian is small. The cost of mercenary culture isn't slower delivery, it's worse decisions. Missionary culture is a structural advantage we can't afford to give up.

---

## How we prioritize

Prioritization frameworks are tools for structuring conversation. **The score is never the decision.**

### RICE, for comparing competing initiatives

Reach × Impact × Confidence ÷ Effort. Useful when staring at a backlog of 15 things that all seem important. The formula produces a score; the score produces a conversation.

### MoSCoW, for triaging a single release

Must / Should / Could / Won't. A triage tool, not a scoring model. Most important bucket is **Won't**. Naming what you're deliberately *not* doing is what makes the rest credible.

### The power of "Not Now"

Prioritization isn't ranking a list of yeses. It's deciding what you're deliberately not doing. **"Not Now" is more honest than "No"**. It acknowledges value while being clear it's not where finite resources are going.

### Alignment is the real test

The top of the backlog should map to the strategic goal in `vision.md`. If the highest-priority item doesn't, the framework is producing noise, not signal. Scores are inputs to judgment, not substitutes for it.

---

## How we measure

Cross-references `vision.md` (objective measurement) and `core-beliefs.md` ("Facts beat opinions").

- **One North Star metric per product.** The number that tells the team whether users are getting value. Not revenue (lagging), not signups (vanity). For Spotify it's time spent listening; for Slack it's messages sent. **What is it for each Vervian product?** If we can't name it, that's the first thing to fix.
- **Actionable, not vanity.** "Total users" only goes up. "Weekly active users" is better. "Retention rate" is the truth. Vanity metrics make you feel good; actionable metrics change behavior.
- **Instrumentation ships with the feature.** Not after. If you can't measure whether it worked, you've shipped a hope, not a hypothesis.

---

## How we work with stakeholders

Even a small studio has real stakeholders: users, partners, and whoever holds the budget. The PM's job isn't to refuse them; it's to redirect when the data and the Why say so.

- **Influence is earned, not granted.** People don't overrule to be difficult, they overrule when they feel unsafe. Earning ownership means earning the trust that handing it over feels safe.
- **Fill the product vacuum.** If the PM doesn't own the strategy, someone else fills the vacuum and the roadmap becomes a grab bag of whatever is loudest or most interesting to build. Strategic alignment isn't optional.
- **The art of "No" is redirection, not refusal.** "We need to build X" gets answered with "here's what the data says, here's what the customer needs, here's why this alternative gets us closer to the goal." Use evidence and the Why, not authority.

---

## Storytelling

> The PM as Chief Meaning Officer.

A product vision that lives in a doc nobody reads isn't a vision, it's a file. The PM's job is to make the vision legible to the team building it, to whoever funds it, and to the customers buying it.

Storytelling isn't spin. It's the ability to take a complex product direction and make it land. The best PMs don't just set direction, they sell it, over and over, to everyone who needs to believe in it.

**When you're debating tradeoffs at 4pm on a Friday, the story is what settles it.** "Does this move us closer to [the thing we said matters]?" That question only works if people remember the answer.

> *"If you genuinely know what you're building and you believe in it, you are a more powerful advocate than any salesperson. Not because you're selling, but because you're not."*

Data gets you credibility. Story gets you alignment. The best product leaders are fluent in both.

---

## What's coming

- **Jobs to Be Done (Christensen).** People don't buy a quarter-inch drill; they want a quarter-inch hole. Vervian will adopt JTBD as the lens for understanding why customers hire our products, especially since a portfolio of focused products (see `vision.md`) means thinking across multiple jobs across multiple products.

---

## Sources

- **Vervian's product principles**, assembled from past product/design teams, brought into Vervian as the starting position. Source for the [Principles](#principles) section.
- **Internal product-management material** (operational sections: five questions, mental models, prioritisation, measurement, stakeholders, storytelling). TBD.
- The thinkers behind the frameworks: Simon Sinek, Melissa Perri, Dan Olsen, Marty Cagan, Clayton Christensen. The "Less, but better" principle traces to Dieter Rams.
- `vision.md`, strategic direction this guide serves.
- `core-beliefs.md`, the non-negotiables this guide operates within.

## Open items

- [ ] Walk through the principles section; capture which land, which need re-shaping, which to add.
- [ ] Decide whether any principle here belongs upstream in `core-beliefs.md` as a Vervian-wide non-negotiable rather than a PM/design principle (candidate: "Quality is not up for negotiation").
- [ ] Name a North Star metric per Vervian product. None is written down here yet.
- [ ] Decide whether the JTBD content becomes its own document or expands a section here.
- [ ] Cross-link this guide from the consumer-CLAUDE.md template once we're confident it's stable.
