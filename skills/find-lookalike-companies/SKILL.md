---
name: find-lookalike-companies
description: >-
  Find look-alike companies on TheirStack from won customers. Profiles closed/won customers on TheirStack (what they ran, hired for, and mentioned before they bought), extracts the signals they share — the "customer fingerprint" — and uses it to find new companies currently showing similar signals. Optionally uses each customer's purchase dates and purchase context — the use cases, products, modules, workflows, problems, goals, or solutions they were interested in or bought — so multi-product offerings match against relevant signals. Reads the product's own website only to know which shared signals matter for the pitch. Outputs the fingerprint plus prioritized look-alike paths, each starting with a live company count that links to a ready-made, pre-filtered search. Use when the user says "find lookalike companies", "look-alike companies", "companies like my customers", "empresas parecidas a mis clientes", "find companies similar to these customers", or pastes a product URL plus customer names.
user-invocable: true
argument-hint: <product-url> <won-customer companies> [purchase dates] [purchase context]
---

# Find Look-Alike Companies

Given **won customers** and the use cases, products, modules, workflows, problems, goals, or solutions they were interested in or bought, find new companies that look just like them. The won customers are ground truth: instead of guessing which signals predict a sale (the `find-theirstack-sales-signals` approach, which reasons from the product website), this skill reads the signals off companies that **already bought** and finds the pattern they shared immediately before they bought. That shared pattern — the **customer fingerprint** — becomes the search for look-alikes currently showing similar signals.

## Input

1. A **product or service URL** (the offering these customers bought).
2. **Won/closed customers** — company names or, better, domains. More customers are better when they share the same purchase context: use cases, products, modules, workflows, problems, goals, or solutions. Use all relevant customers the user provides. Fewer than 3 for a coherent segment is thin, but continue if the user cannot provide more and label the fingerprint as directional.
3. Highly recommended per-customer context:
   - the date each company became a customer, signed, first paid, or expanded;
   - the use cases, products, modules, workflows, problems, goals, or solutions they were interested in or bought;

If the product URL or customer list is missing, ask for it and stop. If dates or purchase context are missing, ask for them once as optional enrichment and continue with best-effort assumptions if the user does not have them.

## Pre-check — verify the TheirStack tools are available (run FIRST, before anything else)

This skill cannot run without the **TheirStack MCP** connected. Before reading any website or profiling any customer:

1. Check whether the TheirStack tools are available in this session — look for `search_jobs`, `search_companies`, `get_company_technographics`, `get_catalog_keywords`, `get_keyword_categories`, `get_keyword_subcategories` and `generate_app_url` (some clients show them with a prefix, e.g. `theirstack__search_companies`).
2. **If they are present**, continue to the Steps.
3. **If they are missing**, STOP and point the user to the setup guide, then wait — do not attempt the rest of the skill:

   > The TheirStack MCP isn't connected, and this skill needs it to read the signals off your customers and find look-alikes. Follow the setup guide to install it, then re-run: https://theirstack.com/en/docs/mcp

## Hard rules

- **The customers are the ground truth, not the product website.** The fingerprint comes from what the won customers actually run/hire/mention in TheirStack. The product website is read **only** to know which of the shared signals matter for the pitch (so you lead with the relevant ones).
- **Purchase context matters.** If the vendor sells multiple products or modules, build the fingerprint for the specific use cases, products, modules, workflows, problems, goals, or solutions the customer was interested in or bought. Do not treat signals for unrelated areas as positive evidence. Example: a CRM buyer does not automatically validate email-marketing signals; an error-monitoring buyer does not automatically validate logging or observability-platform signals.
- **Timing matters.** When purchase dates are available, profile job postings and buying-intent keywords in the window immediately before each relevant purchase. Use a consistent pre-close window by default (e.g. the 180 days before the purchase date), widen it only when volume is too low, and state the window used. If a customer has multiple purchase dates for different use cases, products, modules, workflows, or solutions, profile each relevant window separately. For customers without dates, use the best recent window and mark those signals as weaker.
- **A signal counts only if it's shared.** A signal is only part of the fingerprint if it appears across **2+ reference customers** (ideally a majority). One customer using a tool is noise; three of five is a pattern. State the support (e.g. "4/5") for every signal you keep.
- **Ground or drop.** Every fingerprint signal must resolve to a real `technology_slug`, `keyword_slug`, or firmographic filter — otherwise it isn't queryable. Drop unmatched terms and list them separately.
- **Size with a count.** Size look-alike searches with a [count](https://theirstack.com/en/docs/api-reference/features/count-results): `include_total_results: true` and `limit: 1`. A count is not free: it is charged for the one record it returns (3 API credits for a company search, 1 for a job search), and a search with no matches costs nothing. Only pull more than that if the user explicitly asks for the actual look-alike companies.
- **Exclude the seeds and existing customers.** Look-alikes are _new_ companies; note that the user should exclude their current account list when they action the search.

The skill runs in **three parts with a human checkpoint before look-alike search**. Do NOT skip the checkpoint — Part 3 (finding look-alikes) only starts once the user has confirmed the fingerprint.

## Part 1 — Build the customer fingerprint

1. **Read the product briefly (website only).** Read the product's landing + product/features page (1-3 pages is enough here — you are not deriving signals from it, just learning what it sells and to whom, so you can later rank the shared signals by relevance). Capture: what it sells (one line), its category, the distinct use cases/products/modules/workflows it supports, and the roles/pains each serves.

2. **Normalize the seed set.** Create a seed table with one row per customer purchase context: company/domain, matched TheirStack company ID, purchase date if provided, and the use cases/products/modules/workflows/problems/goals/solutions they were interested in or bought. If one customer bought multiple things on different dates, keep multiple rows. If the seed list mixes materially different buying reasons or product areas, segment it and run one fingerprint per segment; do not average incompatible buyers into one pattern. If the user volunteered product areas or workflows that were explicitly unrelated to the purchase, keep them as exclusions, but do not make exclusions a required input.

3. **Profile each won customer in TheirStack.** For each company, look it up (`search_companies` by domain/name) and read its signals across the same families as the [Sales signals library](https://theirstack.com/en/docs/data/sales-signals):

   - **Technographics** — the technologies it runs (competitors, integrations, adjacent tools, infra). Read them with `get_company_technographics`.
   - **Jobs & hiring** — the roles it posted and the departments it was scaling (`search_jobs` scoped to that company). If a purchase date is known, search the pre-close window before that date; otherwise use a recent window and mark it weaker.
   - **Buying intent** — the keywords/pains mentioned in those job descriptions (`company_keyword_slug`-type topics), filtered to the relevant use cases, products, modules, workflows, problems, goals, or solutions where known.
   - **Firmographics** — size, industry, country/region, and funding stage if available.

   Keep a per-customer table so the overlap is visible.

4. **Extract the fingerprint = the intersection.** Across the customers in each coherent segment, find the signals shared by **2+** of them, per family:
   - shared **technologies** (the strongest signal — a tool most customers run is a near-perfect look-alike filter),
   - shared **hiring roles** / scaling departments,
   - shared **pain/category keywords**,
   - shared **firmographics** (size band, industry, geo, funding).
     Record the **support count** for each (e.g. "Shopify — 12/18", "hiring 'data engineer' — 9/18"). **Ground each** to a real `technology_slug` / `keyword_slug` via `get_catalog_keywords`, `get_keyword_categories` / `get_keyword_subcategories`. Drop and list anything with no catalog match.

## Part 2 — Confirm the fingerprint with the user (STOP and ask — do not continue alone)

Present the fingerprint back to the user, clearly and skimmably:

> I profiled these customers: `<list the companies and the domains/IDs I matched in TheirStack>`. Here's the pattern they share — the **customer fingerprint**:
>
> - **Shared tech:** `<tech (support)>`, … (e.g. Shopify 4/5, Klaviyo 3/5)
> - **Shared hiring:** `<role (support)>`, …
> - **Shared pains/keywords:** `<keyword (support)>`, …
> - **Firmographics:** `<size · industry · geo · funding>`
> - **Timing/purchase context:** `<pre-close window used>`; `<use cases/products/modules/workflows/problems/goals/solutions segment>`
> - **This product sells:** `<one line>` — so the signals most relevant to pitch are `<…>`.

Then ask, and **wait for the answer before doing anything else**:

- Does this fingerprint look right? Anything to **drop** (a coincidence, not a real driver) or **weight up**?
- Any customer I **mis-matched** in TheirStack (wrong company / wrong domain)?
- Any purchase dates or purchase-context details I should add or correct before looking for current-market matches?
- Any signal you **know** matters from your sales experience that didn't surface? Add it.
- Want me to **profile more customers** to firm up the pattern? Paste them.

Fold in corrections (re-profile any new/fixed customers), show the updated fingerprint, and ask again until they confirm. **Only when the user confirms the fingerprint** do you move to Part 3.

## Part 3 — Find the look-alikes (only after confirmation)

4. **Build look-alike searches from the fingerprint.** Compose searches that `AND` the confirmed shared signals, strongest-support first. Build, at minimum:

   - a **tight look-alike** search — the highest-support signals stacked (e.g. shared tech + size band + geo) → the closest matches to your customers,
   - one **single-signal** search per strong shared signal (e.g. each shared technology on its own) → broader reach,
   - and, where the firmographics are tight, one **firmographic-only** overlay to show the addressable ICP size.

5. **Size every search (one record per count).** For each, call `search_companies` / `search_jobs` as a [count](https://theirstack.com/en/docs/api-reference/features/count-results) (`include_total_results: true`, `limit: 1`) and read `metadata.total_companies`. That number is the headline of each look-alike path. Note the nuance: tech/keyword counts are whole-database ceilings before the user excludes their existing customers; current hiring counts are a flow within the posting window.

6. **Generate one deep-link search URL per look-alike path** so each count opens the live, pre-filtered search in the app — **do NOT create saved searches**. Call `generate_app_url` with `type: "company_search"` (or `"job_search"` for a hiring path) and `filters` set to the same filters you sized with, **minus the count-only flags** (`include_total_results`, `limit`). It returns a URL of the form `https://app.theirstack.com/search/companies/new?query=<encoded>` that opens the search and runs it automatically. Put that URL on the count.

## Output

Write the output **in the user's language** (match the conversation). It has two parts:

**1. The customer fingerprint** — a short, skimmable block: the shared signals with their support counts, grouped by family (tech / hiring / pains / firmographics / timing and purchase context), one line each. This is the "why these companies look alike" summary.

**2. The look-alike paths** — a **bullet list, professional tone, no emojis**, ordered by closeness to the fingerprint, with **only two levels** — e.g. `Look-alikes prioritarios` / `Look-alikes secundarios` (or `Priority look-alikes` / `Secondary look-alikes`).

Each path is **one bullet** that **starts with the company count, and that count is the hyperlink to its search**:

> - **[1,280 companies that look like your customers](https://app.theirstack.com/search/companies/new?query=eyJxdWVyeSI6…)** — run Shopify + Klaviyo, 50-500 employees, US (shared by 4/5 of your won accounts). Closest match to your existing customers.

Output rules:

- Lead with the number of companies; that number is the link to the search. Put the link **only** on the count.
- One path per line, starting with the count + what it finds — **not** with the signal family.
- For each path, name **which shared signals** it stacks and **their support** (e.g. "4/5"), so the user sees why it's a look-alike.
- When purchase dates were used, say that the seed signals came from the pre-close window and the look-alikes are companies showing similar signals now.
- Put the **tight look-alike** at the top of the priority level; single-signal searches are broader and go below.
- End with: a one-line note that each link opens the live, pre-filtered search in the app (nothing is saved to the account), a reminder to **exclude their existing customer/account list** when they pull companies, an offer to turn the tightest path into a saved search with email alerts (only on request), and a one-line note of any signals dropped for lack of a catalog match ("request these as new keywords").

## Notes

- **The fingerprint is only as good as the seed set.** More _similar_ won customers with the same purchase context usually make the pattern stronger. A grab-bag of different use cases, products, modules, industries, or buyer motions gives a vague pattern; segment them and run one fingerprint per segment.
- **Dates make intent sharper.** If purchase dates are available, prefer pre-close hiring and keyword signals because they describe what the account looked like before it converted or expanded. Use current signals only for finding new look-alikes.
- **Shared technology is the strongest look-alike signal**, then firmographics, then hiring/keywords. Weight accordingly when ordering paths.
- **Combine for precision, split for reach.** The tight look-alike (several signals `AND`-ed) is precise but small; single-signal searches reach further but are noisier. Offer both.
- **Re-run per market** — pain phrasing and role titles don't translate 1:1 (FR / EN / ES passes).
- Sizing costs one record per count (3 API credits per company search, 1 per job search, nothing when there are no matches). Tell the user roughly how many credits the sizing used, and only spend more if they explicitly ask to pull the actual look-alike companies.
- Related skill: `find-theirstack-sales-signals` derives signals from the _product_ (top-down); this skill derives them from _won customers_ (bottom-up). They're complementary — run both and intersect.
