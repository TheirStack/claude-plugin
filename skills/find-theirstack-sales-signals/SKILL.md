---
name: find-theirstack-sales-signals
description: Find the TheirStack outbound/sales signals for any product or service offering, not just SaaS. Reads the offering's OWN website to understand the problems it solves, then maps them to TheirStack signals (jobs, technographics, buying intent) a GTM team can run for outbound. Runs end-to-end (no confirmation step) and returns a self-contained report — company summary, the company's products & services, ICP options, competitors, a firmographic overlay, and signals grouped by the company's product/service — each signal a bullet that starts with a live company count linking to a ready-made, pre-filtered search URL, under the offering it is used to pitch. Use when the user says "find sales signals", "outbound signals for <product>", "what TheirStack signals should <product> use", "which signals can this offering use", or pastes a product or service URL for signal discovery.
user-invocable: true
argument-hint: <product-url> [extra URLs or free-text context]
---

# Find TheirStack Sales Signals

Given the website of any product or service offering — not just SaaS — produce the TheirStack signals its GTM team should use for outbound: read the offering → derive the tasks, roles, competitors and integrations behind each problem → map them to TheirStack's three data types.

## Input

A **product or service URL** (the offering you want signals for). The input may also include **extra URLs** (e.g. a specific feature page, a comparison page, a docs site) and/or **free-text context** (the seller's notes on their ICP, target market, competitors, the offering to push). Treat everything passed in as context: read every URL given, and fold any free-text guidance into the research — it overrides your own inference where the two disagree. If no URL is given at all, ask for one and stop.

**Run end-to-end — do NOT stop to confirm the research.** There is no human checkpoint: research the product, ground the signals, and produce the full report in one pass. State your firmographic inference inside the report (in the overlay header) so the reader can adjust later, but never pause to ask them to validate the input first.

## Pre-check — verify the TheirStack tools and the companion skill (run FIRST, before anything else)

This skill cannot run without the **TheirStack MCP** connected. Before reading any website or running any step:

1. Check whether the TheirStack tools are available in this session — look for `search_companies`, `search_jobs`, `get_catalog_keywords` and the URL builder `generate_app_url` (some clients show them with a prefix, e.g. `theirstack__search_companies`).
2. **If they are present**, continue to the Steps.
3. **If they are missing**, STOP and point the user to the setup guide, then wait — do not attempt the rest of the skill:

   > The TheirStack MCP isn't connected, and this skill needs it to ground signals in real, queryable filters. Follow the setup guide to install it, then re-run: https://theirstack.com/en/docs/mcp

This skill also grounds its **buying-intent / pain** signals (family C) by reusing the iteration loop of the **`job-description-keyword-research`** skill — it reads real job postings and mines the exact phrasings companies use, instead of guessing pain keywords once. That skill is a **hard requirement**:

4. Confirm the `job-description-keyword-research` skill is available in this session.
5. **If it is present**, continue to the Steps.
6. **If it is missing**, STOP and tell the user to install it, then wait — do **not** fall back to catalog-only pains:

   > This skill grounds its buying-intent signals with the `job-description-keyword-research` skill, which isn't installed. It ships with this one — install it from https://theirstack.com/en/docs/mcp/skills/job-description-keyword-research, then re-run.

## Hard rules

- **Research the product from its WEBSITE only.** Read the site with your web browsing or fetch tool. Do **not** use `search_companies`/`search_jobs` to research the product itself.
- **Trust the connection's own auth.** The TheirStack MCP authenticates itself. Always call the tool directly; if a call fails with an authentication error, tell the user to reconnect TheirStack and stop.
- **The catalog is for grounding, not prospect research.** Use `get_catalog_keywords` (with `keyword_type: "technology"` for technologies) only to turn human terms ("Pleo", "expense reporting") into real, queryable slugs.
- **It spends the user's credits.** Every count and every sample read is billed to their TheirStack workspace. Say so in one line before you start, and report the rough total at the end.
- **Infer firmographics, don't ask for them.** Derive the firmographic overlay (countries, industries, employee band) from the site (and any context passed in) and apply it to every signal so the counts are addressable market, not whole-database noise. State your inference inside the report's overlay header so the reader can adjust later — but do **not** pause to confirm it.
- **Must-use vs nice-to-have drives scoring.** For every technology, decide whether the product _can't_ serve a company without it (blocking) or it merely correlates with fit. Must-use signals outrank nice-to-have ones.
- **Pains come from the companion loop, not a one-shot guess.** Discover and ground every buying-intent / pain signal by running the `job-description-keyword-research` procedure (its Steps 2–3) **to saturation**, reusing the product↔pain map from Step 1 here (do not re-fetch the site). The output is grounded `job_description_pattern_or` groups, each validated against real postings and tied to one product feature. A catalog keyword slug (`company_keyword_slug_or`) is a coarse fallback only for a pain with no usable job-description phrasing.
- **Stack pains with tech via job-search, not company-keyword OR.** To combine a pain with a must-use technology, use `search_jobs` with `company_technology_slug_or` AND `job_description_pattern_or` — these AND across dimensions. Do **not** combine `company_keyword_slug_or` + `company_technology_slug_or` in `search_companies` expecting an intersection: those **union**, which inflates the count above either filter alone.
- **Ground or drop.** A signal that isn't grounded in real, queryable data isn't actionable — drop it and list it separately as "request missing". Technographic signals ground to a real `technology_slug`; pain / buying-intent signals ground to an iterated `job_description_pattern_or` group (see the rule above), not a single catalog-keyword guess.
- **Don't trim the catalog.** Output all signal families; mark each high/med/low.

**Run the whole thing in one pass — there is no human checkpoint.** Research the product, ground the signals, and write the full report without stopping to ask the user to validate anything.

## Step 1 — Research the product (read several pages, not just one)

1. **Read MULTIPLE pages of the product site, not only the landing.** From the given URL (plus any extra URLs passed in), discover and read the key pages: the home/landing, **product / features**, **solutions / use-cases**, **pricing**, **integrations**, and **customers / case studies**. If you can only fetch the landing, follow the obvious nav links and read at least 3–5 pages. The whole point is to understand the product in depth, not from a single page. Fold any free-text context passed in into the research (it overrides your inference where they disagree).

2. **Synthesize what the product is**, from everything you read:
   - what it sells (one line) and its **value proposition** (the core promise to its customer, in its own framing),
   - its **products & services** — the offerings _this company_ sells (used later in the report's "Products & services" section), with a one-line description each,
   - its **ICP** (who it's for — size, industry, persona), and the **distinct ICP segments/options** it could target,
   - the **firmographic overlay** to filter every signal by — **inferred, do NOT ask the user**: **target countries** (from the site's language, currencies, named regions/offices, accreditations and case-study locations), **industries to include or exclude** (the verticals it sells to, and any it clearly can't serve), and the **employee-count band** of the ICP. These become the `company_country_code_or`, `industry_id_or` / `industry_id_not` and `min_employee_count` / `max_employee_count` filters applied to every count.
   - the concrete **problems / jobs-to-be-done** it solves,
   - its **competitors** — other businesses in the same space (name real examples; if the site doesn't list them, use well-known comparable businesses and mark them illustrative) — and the **integrations** it names, tagging each integration/competitor as **must-use (blocking)** (the product cannot serve a company that doesn't run it — a hard prerequisite, or the only stack it integrates with) or **nice-to-have** (it correlates with fit but isn't required). This split drives lead scoring.
   - for each problem: the manual **tasks/processes** it removes and the **roles / job titles** of the people who own that problem.

## Step 2 — Map to TheirStack signals

3. **Load the signal catalog.** Read the **Sales signals library** — `https://theirstack.com/en/docs/data/sales-signals.md` — and use it as your checklist. Your **signal sources are the three it defines** (A Technographics, B Jobs and hiring, C Buying intent topics) — every signal you output comes from one of them. **Firmographic and temporal are overlays you apply on top of a signal, not sources**: TheirStack is not a firmographics source, so infer the firmographic overlay only to narrow the counts (never pitch industry/size as a signal), and use temporal filters (first-seen, recency, `min_jobs`, surges — a real TheirStack strength) to prioritize by timing. "Trigger plays" are ready-made combinations. This is the single source of truth for the signal list and the exact filter behind each; do not invent signals outside it.

4. **Ground technographics in the TheirStack catalog** — call `get_catalog_keywords` with `q` set to each competitor or integration name and `keyword_type: "technology"` to resolve it to a technology slug. Confirm each slug really exists (and is the right one) by checking it returns a non-zero count in the count-mode search in step 6. Drop unmatched terms (list them).

   **Ground pains with the `job-description-keyword-research` loop (family C).** Do **not** one-shot pains to catalog keyword slugs. Instead run that skill's procedure (its Steps 2–3) **to saturation**, reusing the product↔pain map from Step 1 here — no re-fetch of the site: for each pain, brainstorm a wide set of real-world phrasings (verb/noun forms, regional spellings, tool-agnostic descriptions, the user's other target languages), size each with a count request (`include_total_results: true`, `limit: 1`, ~1 credit each), **read small samples of real postings** (5–10 jobs, ~1 credit/job) to mine the phrasings companies actually use and exclude false positives via `job_description_pattern_not`, then re-run until a round adds nothing new (~3–4 rounds). The output is a set of grounded `job_description_pattern_or` groups, each tied to one product feature — these ARE your family-C signals. A catalog keyword slug is a coarse fallback only for a pain with no usable job-description phrasing.

5. **Apply the firmographic overlay, fill every signal, and score by fit.** Apply the inferred firmographic overlay — countries, industries, employee band — to **every** signal so the counts are addressable market, not whole-database noise. Fill every signal in the library with this product's confirmed values (competitors, integrations, roles, keywords). Then **score each signal by fit, not raw count**: a signal built on a **must-use** technology (a hard prerequisite) outranks one built on a **nice-to-have**; an explicit buying-intent/pain signal outranks a passive technographic one; and a stacked signal (must-use tech + pain + firmographics) outranks any single filter. This fit score — not the company count — decides the ordering of signals **within each product subsection** (highest-intent first), and which ones are the secondary / marginal / ceiling ones that go last. Keep this scoring internal — it drives the ordering only; do **not** print it as an inline fit tag on the bullets. Don't trim families.

6. **Size every signal (one record per count).** For each grounded signal, get the company count with `search_companies` / `search_jobs` as a [count](https://theirstack.com/en/docs/api-reference/features/count-results) — `include_total_results: true`, `limit: 1` — and read `metadata.total_companies`. Size each signal **with the firmographic overlay applied**, so the headline is the addressable number under the ICP; only when you deliberately show a whole-database **ceiling** (e.g. all users of a competitor before geo/size) do you drop the overlay, and you must label that number a ceiling. Hiring counts are a flow within the posted-date window — say so. Always also build and size one combined **core list** — the correct cross is a `search_jobs` query with `company_technology_slug_or` (must-use entry tech) AND `job_description_pattern_or` (a converged pain group from step 4) AND geo + size, since these AND across dimensions. Do **not** build it as `company_keyword_slug_or` + `company_technology_slug_or` in `search_companies` — those union and inflate the count above either filter alone.

7. **Generate one deep-link search URL per signal** so each count opens the live, pre-filtered search in the app — **do NOT create saved searches**. Call `generate_app_url` with `type: "company_search"` (or `"job_search"` for hiring signals) and `filters` set to the same filters you sized with — **minus the count-only flags** (`include_total_results`, `limit`), which don't belong in a user-facing search. It returns a URL of the form `https://app.theirstack.com/search/companies/new?query=<encoded>` (or `/search/jobs/new?…`) that opens the search and runs it automatically for whoever opens it. Put that URL on the company count.

   If `generate_app_url` isn't available, show the counts without links and say so in the closing notes.

## Output

Write the output **in the user's language** (match the conversation; re-run per market since pain phrasing and role titles don't translate 1:1). It is **professional tone, no emojis**. The signals are **grouped by the product/service of the analyzed company that each one is used to pitch** — there is **one subsection per offering**, not a global `High intent` / `Other` split. Inside each product, write the signals as a **bullet list**, ordered by fit (highest-intent first).

The report has this exact section order: **company summary → Products & services → ICP → Competitors → Firmographic overlay → Signals by product/service → closing notes.** Build it as a self-contained report.

**Company summary** — open with a short heading carrying the company name, then a one-line **What they sell** and a one-line **Value proposition** (the core promise to their customer, in their own framing). Draw both from Step 1; keep them tight, one sentence each.

**Products & services** — the offerings of **the analyzed company itself** (what it sells to its customers), drawn from Step 1 — **never** TheirStack's products/features. Use a short heading and one entry per offering: a bold name plus a one-line description in the company's own framing.

**ICP** — the distinct ICP segments/options the company could target, one short paragraph each (e.g. by stack, by vertical, by buying trigger). Make them genuinely different options, not one ICP reworded; tie each to the signals below where natural.

**Competitors** — examples of other businesses in the same space as the analyzed company (its competitors), one short line each. Prefer competitors named on the site; otherwise give well-known comparable businesses and mark them illustrative ("confirm with the seller"). Note that, once confirmed, these become a competitor-displacement signal.

Then add the **structured firmographic-overlay header** — a short heading (e.g. `Firmographic overlay (applied to every count)`) followed by **bullet points**, one per dimension, so the reader knows the ICP behind the numbers. Write it for the GTM reader, **not** as TheirStack query syntax: describe each dimension in plain business language and **never expose filter/parameter names** (no `company_country_code_or`, `industry_id`, `min_employee_count`, etc.). Use these bullets, in this order, dropping any that don't apply:

> ### Firmographic overlay (applied to every count)
>
> - **Countries (HQ):** `<list>`
> - **Size:** `<min–max>` employees
> - **ICP / vertical:** `<one line>` — identified by the stack they run (`<key tools>`) and their hiring activity
> - **Why no industry filter:** `<one line>` — only include this bullet if you deliberately skipped an industry filter (e.g. LinkedIn's industry tag doesn't expand to sub-industries and dropped most real matches); cite the before/after count
> - **How to read the counts:** company counts are addressable market under this overlay; hiring counts are a flow within the stated window (e.g. 30/90 days), not a cumulative total

**Signals by product/service** — this is the core of the report. Create **one subsection per offering** from the Products & services list, in the same order, headed by the offering name plus a short italic descriptor of what it is (e.g. `## Freshservice / InvGate Service Management — *ITSM / Service Desk*`). Under each, list the signals used to pitch **that** offering as a **bullet list**, ordered by fit (the combined core list and explicit buying-intent signals first, then must-use displacement, then nice-to-have/secondary, then marginal/ceiling).

Each signal is **one bullet** that **starts with the company count, and that count is the only hyperlink in the bullet**, followed directly by a **verbose, clear explanation** — what the filter finds, why it matters for _this_ offering, the pitch angle, and any nuance or flow caveat (2–4 sentences, not one terse line). Do **not** prepend an italic fit tag (e.g. `— _high intent · displacement_.`); go straight from the count into the explanation:

> - **[383 companies already running Zendesk](https://app.theirstack.com/workspace/213/search/companies/new?query=eyJxdWVyeSI6…)** — companies of 50+ employees already using Zendesk for customer support. It is the largest CX displacement pool and the pitch is clear: move to a cheaper Freshworks stack with chat and telephony already integrated. Direct fit, shorter cycle than enterprise ITSM.

Output rules:

- **Group by offering, not by tier.** Every signal lives under the product it is used to pitch (the subsection heading is that offering — one of the analyzed company's real offerings, **never** a TheirStack product). Do **not** add global `High intent signals` / `Other signals` headings, and do **not** use an inline `Recommended product:` — the heading already names the product.
- Lead each bullet with the linked count; put the link **only** on the count, nowhere else. Go straight from the count into the explanation — do **not** add an italic fit tag (no `_high intent · displacement_`, `_secondary_`, `_marginal / ceiling_`, etc.). Fit still drives the ordering of bullets; it is just not labelled inline.
- Make each description **verbose and clear** (2–4 sentences): what the signal finds, why it matters for that specific offering, the angle to pitch, and any caveat — including whether the count is addressable market or a hiring **flow** within the window.
- Order signals **within each product by fit**, not raw count: the combined core list (must-use tech + pain + firmographics) and explicit buying-intent/hiring signals come first; nice-to-have technographics and whole-database ceilings come last. The ordering conveys fit — do **not** label it with an inline tag.
- If one signal genuinely feeds two offerings, place the full bullet under its primary offering and add a short note there that it also feeds the other — don't duplicate the bullet.
- Put offerings that have **no grounded signal** last, grouped together under one line (e.g. `## Ringover · Applivery — no grounded signal (request missing)`), with a bullet explaining why (no catalog match / pain too noisy) and the request-missing action.
- Note any nuance inline (e.g. "ServiceNow is the biggest pool but a heavy enterprise rip-and-replace; treat as long-cycle").

End with: a one-line note that each link opens the live, pre-filtered search in the app (nothing is saved to the account), then a one-line note of any signals dropped for lack of grounding (a technographic with no catalog match — "request these as new keywords/technologies" — or a pain whose sampled postings were mostly irrelevant), and a rough **credit cost** for the run (the pain loop's sample reads cost ~1 credit/job; sizing costs credits too). If the user wants any of these monitored over time, offer to turn one into a saved search with email alerts — but only on request.

## Notes

- **Firmographics are inferred, not asked.** Derive countries, industries and employee band from the site (and any context passed in), state them in the report's overlay header, and filter every signal count to that overlay. Don't pause to confirm — the user can adjust after seeing the report.
- **Lead scoring follows must-use.** A blocking technology (the product can't work without it) scores a prospect higher than a nice-to-have; stacked signals (must-use tech + pain + firmographics) score highest of all.
- Combine signals for precision: _competitor + recently funded_, _target role + pain keyword_. Single-signal lists are broader and noisier.
- **Pains are grounded, not guessed.** Family C reuses the `job-description-keyword-research` loop — iterate against real postings until phrasings saturate. Reading postings to mine phrasings costs ~1 credit/job, so keep samples small (5–10) and report the rough credit spend; sizing costs credits too (see below).
- **AND vs OR.** Stack pain + tech via `search_jobs` (`company_technology_slug_or` AND `job_description_pattern_or`). In `search_companies`, `company_keyword_slug_or` and `company_technology_slug_or` **union**, not intersect — never build the core list that way.
- Sizing is not free: each count (`include_total_results` + `limit: 1`) is charged for the one record it returns (3 API credits per company search, 1 per job search, nothing when there are no matches). Tell the user roughly how many credits the run used, and only spend more if they explicitly ask to pull the actual companies, or for the pain-loop sample reads above.
- Re-run per market — pain phrasing and role titles don't translate 1:1 (FR / EN / ES passes).
