---
name: research-company-brand-book
description: Research a company from its own website and fill its brand book — the company profile, its products & services with their value propositions, the problems they solve and the job titles that own them, its ideal customer profiles as sized TheirStack company search filters, the customers named in its case studies, its competitors and integrations resolved to TheirStack technology slugs, and the firmographic overlay its buyers share. When the company is the workspace's own, it merges the result into the workspace brand book; for any other company it only returns it. The research step behind `find-theirstack-sales-signals` and `find-lookalike-companies`, which hand it to a subagent when the brand book is empty or incomplete. Use when the user says "fill my brand book", "research my company", "build the brand book for <url>", "update the brand book", or pastes their own company URL to profile it.
user-invocable: true
argument-hint: <company-url> [extra URLs or free-text context]
---

# Research Company Brand Book

Given the website of a company, read it in depth and turn it into a **brand book**: who the company is, what it sells, the problems it solves and who owns them, who it sells to (as TheirStack company search filters), who already bought, and who it competes with. The brand book is the shared starting point of the TheirStack sales skills: `find-theirstack-sales-signals` maps it to signals and `find-lookalike-companies` uses it to rank a customer fingerprint. When the company is the workspace's own, the result is saved so every later run starts from it.

## Input

A **company URL**. The input may also include **extra URLs** (a feature page, a case-study page, a docs site) and/or **free-text context** (the seller's notes on their ICP, market, competitors). Read every URL given and fold the free-text context into the research; it overrides your own inference where the two disagree. If no URL is given, ask for one and stop.

When another skill hands you this research, it also tells you whether the user presented the offering as their own ("my company", "our product", "we sell…"); treat that exactly as if the user had said it to you.

**Run end-to-end, without a confirmation step.** State your inferences in the result so the reader can adjust them later.

## Pre-check — verify the TheirStack tools (run FIRST)

1. Check that the TheirStack tools are available — `search_companies`, `find_technologies_and_topics` and `list_industries` (some clients show them with a prefix, e.g. `theirstack__search_companies`).
2. **If they are missing**, STOP and point the user to the setup guide:

   > The TheirStack MCP isn't connected, and this skill needs it to turn the research into real, queryable filters. Follow the setup guide to install it, then re-run: https://theirstack.com/en/docs/mcp

3. **Check for the brand book tools** — `get_brand_book` and `save_brand_book`. They are **optional**: when they are missing (some clients list read tools only), skip Step 0 and Step 4, and return the research without saving it.

## Hard rules

- **Write only to the brand book, and only the workspace's own.** The one write this skill makes is `save_brand_book` in Step 4. It never creates saved searches, lists or anything else.
- **Research the company from its WEBSITE only.** Read the site with your web browsing or fetch tool. Do **not** use `search_companies` or `search_jobs` to research the company itself; `search_companies` is only for sizing ICP filters (Step 3).
- **The catalog is for grounding.** Use `find_technologies_and_topics` (with `keyword_type: "technology"`) only to turn competitor and integration names into real technology slugs, and `list_industries` to turn verticals into industry ids.
- **Trust the connection's own auth.** Always call the tool directly; if a call fails with an authentication error, tell the user to reconnect TheirStack and stop.
- **It spends the user's credits.** Each ICP count costs 3 API credits (nothing when there are no matches). Report the rough total at the end.
- **Never guess customers.** Only list companies the site (or another public page) names, each with the URL of that page.

## Step 0 — Read the current brand book

1. Call `get_brand_book`. It returns the brand book plus `filter_problems` (ideal customer profiles whose stored filters company search no longer accepts).
2. **It applies to this run** when its `company.website` has the same domain as the URL you were given. Then its products, problems, ICPs, customers and competitors are the starting point of Step 1, and anything a person wrote there overrides your own inference, exactly like free-text context. Still read the website to fill what is missing and to check what changed, and fix every ICP named in `filter_problems`.
3. When it is empty, or describes another company, ignore it for the research (the URL may be a prospect's, not the workspace's).

## Step 1 — Research the company (read several pages, not just one)

1. **Read MULTIPLE pages of the site, not only the landing.** From the given URL (plus any extra URLs), discover and read the key pages: the home/landing, **product / features**, **solutions / use-cases**, **pricing**, **integrations**, and **customers / case studies**. If you can only fetch the landing, follow the obvious nav links and read at least 3–5 pages.

2. **Synthesize what the company is**, from everything you read:
   - what it sells (one line) and its **value proposition** (the core promise to its customer, in its own framing),
   - its **products & services** — the offerings _this company_ sells, with a one-line description, value proposition and the URL of the site's own page that explains it (its product or feature page; leave it empty when the site has none) each, and which one the site leads with,
   - the concrete **problems / jobs-to-be-done** each offering solves, the manual **tasks/processes** it removes, and the **roles / job titles** of the people who own that problem — written as companies write them in job postings, since that is how hiring signals search them,
   - its **ICP** (who it's for — size, industry, geography) and the **distinct ICP segments** it could target,
   - the **firmographic overlay** — **inferred, do NOT ask the user**: **target countries** (from the site's language, currencies, named regions/offices, accreditations and case-study locations), **industries to include or exclude** (the verticals it sells to, and any it clearly can't serve) and the **employee-count band** of its buyers,
   - its **customers** — every company named on a customers / case studies page or in a testimonial (open each case study, not only the logo wall): name, domain, industry, the product it uses, the problem it bought it for and the result it reports with its numbers, plus the URL of the page that names it. Logos with no story still count (name and domain only). Read them as evidence for the ICP and the overlay: the industries, sizes and countries of real buyers outrank what the marketing copy claims,
   - its **competitors** — other businesses in the same space (name real examples; if the site doesn't list them, use well-known comparable businesses and mark them illustrative) — and the **integrations** it names, tagging each integration and competitor as **must-use (blocking)** (the offering cannot serve a company that doesn't run it — a hard prerequisite, or the only stack it integrates with) or **nice-to-have** (it correlates with fit but isn't required).

## Step 2 — Ground the research in the TheirStack catalog

3. **Technologies.** For each competitor and integration, call `find_technologies_and_topics` with `q` set to its name and `keyword_type: "technology"` to resolve it to a technology slug. Keep the slug only when it is the right product (not a homonym). List the names with no match separately.
4. **Industries.** Resolve the overlay's verticals to industry ids with `list_industries`.

## Step 3 — Build and size the ideal customer profiles

5. Turn each ICP segment into **company search filters**: the firmographic overlay (`company_country_code_or`, `industry_id_or` / `industry_id_not`, `min_employee_count` / `max_employee_count`, revenue) plus the segment's must-use technologies (`company_technology_slug_or`). An ICP says **who fits**, not who is ready to buy now: timing signals (a recent hire, a pain in a job posting, a competitor to displace) do not go in it, since with a short date window the profile would hold different companies every week. The one exception is a segment that only its hiring identifies, because no firmographic or technology filter isolates it (e.g. organizations that employ strength & conditioning coaches): put that hiring in `job_filters` with a wide window (`posted_at_max_age_days` of 180 or more). An ICP describes companies, never a buyer persona.
6. **Size each ICP** with `search_companies` as a [count](https://theirstack.com/en/docs/api-reference/features/count-results) — the filters plus `include_total_results: true` and `limit: 1` — and read `metadata.total_companies`. When a profile returns nothing or a number that makes no sense for the market (an industry filter that drops most real matches is the usual cause), loosen it and size again. Keep only profiles that returned companies.

## Step 4 — Save the brand book (only the workspace's own company)

Save only when `save_brand_book` is available **and** one of these holds: the brand book from Step 0 already describes this company (same website domain), or it is empty and the user presents the company as their own. Otherwise do not save; say so in the result and offer it ("Say the word and I'll save this as your workspace brand book").

7. **Build the whole brand book, merging into what is there.** `save_brand_book` replaces the brand book with the body you send (the previous version stays in the history), so start from what `get_brand_book` returned and change it in place: never drop a product, problem, ICP, customer or competitor that is already there; update entries by `name` and append new ones. Fill:
   - `company`: `name`, `website`, `linkedin_url`, `industry`, `employee_range`, `headquarters`, `founded_year`, `overview` (the value proposition and what sets it apart).
   - `products`: one per offering (`name`, `description`, `value_proposition`, `url` = its product page, `is_main` on the one the site leads with).
   - `problems`: one per problem (`name`, `description`, `manual_process` = the task it removes, `product_name` = the offering that solves it, `job_titles` = the roles that own it).
   - `ideal_customer_profiles`: one per sized ICP (`name`, `description`, `product_name` = the offering it is pitched, `filters` = the company search filters from Step 3 without `limit`, `page` or `include_total_results`, and `job_filters` only for the hiring-identified exception).
   - `customers`: one per customer (`name`, `domain`, `industry`, `product_name` = the offering it uses, `use_case`, `outcome`, `case_study_url`). Save only customers with a public source in `case_study_url`.
   - `competitors`: `name`, `domain`, `technology_slug` (from Step 2, when there is one) and `differentiator`.
8. Call `save_brand_book` with `{"brand_book": <the brand book>, "source": "agent"}`. If it rejects an ICP's filters, fix that ICP (the error names it and the filter) and retry once; if it still fails, drop that ICP from the save and say so in the result.

## Output

Return, in the user's language, professional tone, no emojis:

1. **The brand book** — company, products & services, problems (with their job titles), ideal customer profiles (each with its filters and its company count), customers (with their case-study URLs) and competitors (with their technology slugs), in the shape of Step 4 whether or not it was saved.
2. **Integrations** — each with its technology slug and whether it is must-use or nice-to-have. The brand book doesn't hold them, so the skills that hand you this research need them here.
3. **Firmographic overlay** — countries, industries to include or exclude (names and ids) and employee band, with one line on what you inferred it from.
4. **Closing notes** — whether the workspace brand book was saved, merged or left untouched (and why), the names with no catalog match ("request these as new technologies"), the competitors marked illustrative, and the rough credit cost.

When another skill handed you this research, return exactly these four parts and nothing else: that skill writes the report.
