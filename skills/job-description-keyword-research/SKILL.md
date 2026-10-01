---
name: job-description-keyword-research
description: Do keyword research for job-description search on TheirStack. Works for any product or service offering, not just SaaS. Reads the offering's OWN website to understand what it sells and the problems it solves, derives the pains a buyer would describe in a job posting, expands each pain into a wide set of synonyms and real-world phrasings, then iterates against real job postings via the TheirStack MCP — reading actual descriptions to learn how companies phrase each problem, mining new phrasings, and refining the list round after round. Returns keyword groups, each explained and tied to a feature or service, with a live job count and a ready-made, pre-filtered TheirStack job-search URL. Use when the user says "keyword research for job postings", "what keywords should I search in job descriptions for <product>", "find the pain keywords for <product>", "build job-description search terms", or pastes a product or service URL for keyword research.
user-invocable: true
argument-hint: <product-url> [extra URLs or free-text context]
---

# Job Description Keyword Research

Given the website of any product or service offering, build the **keyword groups** its GTM team should search inside job descriptions to find companies that have the problem the offering solves. This works for any offering that sells one or more products or services — not just SaaS. It follows the [identify-companies guide](https://theirstack.com/en/docs/guides/how-to-identify-companies-with-problems-your-software-solves): read the offering → derive the pains a buyer would describe when hiring → expand each pain into many phrasings → **iterate against real job postings** until the list stops growing → size each group and hand back a deep-link.

The differentiator of this skill is the **iteration loop**: it does not guess keywords once. It reads real job descriptions on TheirStack, learns the exact words companies use for each pain, mines new synonyms from them, and refines round after round.

## Input

A **product or service URL** (the offering you want keywords for). May also include **extra URLs** (a feature page, a use-case page, a docs site) and/or **free-text context** (the seller's notes on ICP, the geography to focus on, the offering to push). Read every URL given and fold any free-text guidance into the research — it overrides your own inference where they disagree. If no URL is given at all, ask for one and stop.

**Run end-to-end — do NOT stop to confirm the research.** There is no human checkpoint: research the product, iterate against real jobs, and produce the full report in one pass.

## Pre-check — verify the TheirStack tools are available (run FIRST, before anything else)

This skill cannot run without the **TheirStack MCP** connected. Before reading any website or running any step:

1. Check whether the TheirStack tools are available in this session — look for `search_jobs` and the URL builder `generate_app_url` (some clients show them with a prefix, e.g. `theirstack__search_jobs`). This skill works on **free-text job-description patterns**, not catalog slugs, so it needs no catalog tool.
2. **If they are present**, continue to the Steps.
3. **If they are missing**, STOP and point the user to the setup guide, then wait — do not attempt the rest of the skill:

   > The TheirStack MCP isn't connected, and this skill needs it to read real job postings and size each keyword group. Follow the setup guide to install it, then re-run: https://theirstack.com/en/docs/mcp

## Hard rules

- **Keywords are the BUYER's words, not the seller's.** A keyword group must match how a hiring company phrases its _problem_ in a job description, not how the vendor markets its solution. "expense report", "chase receipts", "month-end close" — not "spend management platform". Marketing nouns rarely appear in job posts; the manual tasks and pains do.
- **Research the offering from its WEBSITE only.** Read the offering's site with your web browsing or fetch tool. Do **not** call `search_jobs` to learn what the offering does — that feeds demand-side job-posting patterns back into keyword generation (circular results) and charges credits on a step that does not need them.
- **Iterate against real postings — never guess once.** Every group must be grounded by reading actual job descriptions on TheirStack (Step 3). Mine the real phrasings, add them, re-run. A group that was never checked against a real posting is not done.
- **Sizing and reading both cost credits.** A [count](https://theirstack.com/en/docs/api-reference/features/count-results) (`include_total_results: true` + `limit: 1`) costs **1 API credit when anything matches and nothing when it doesn't**, whatever the total. Pulling real job descriptions to read them **costs 1 credit per job returned** — keep each sample small (5–10 jobs) and only pull what you need to mine phrasings. Tell the user roughly how many credits the run used.
- **A date filter is mandatory.** `search_jobs` requires at least one date filter. Default to `posted_at_max_age_days: 90` for every count and deep-link unless the user asks for a different window. Counts are a **flow within that window**, not a cumulative total — say so.
- **Ground or drop.** A keyword group that returns ~0 jobs, or whose sampled jobs are mostly irrelevant, is not actionable — tighten it, or drop it and note why.

## Step 1 — Research the product → product↔pain map

1. **Read MULTIPLE pages of the product site, not only the landing.** Read the home, **product / features**, **solutions / use-cases**, **pricing**, **integrations**, and **customers / case studies**. If you can only fetch the landing, follow the obvious nav links and read at least 3–5 pages. Fold in any extra URLs and free-text context passed in.

2. **Build the product↔pain map.** For each distinct feature / use-case the product sells, capture:

   - **Feature** — the offering, in the company's own framing (e.g. "Expense management").
   - **What it does** — a one-line description of the feature (this is the product-description column the report keeps).
   - **Pains / jobs-to-be-done** — the concrete manual tasks, broken processes or responsibilities a company would mention in a job post when it has this problem (e.g. "manually chasing receipts", "reconciling card expenses", "month-end close").
   - **Owner roles** — the job titles of the people who own that pain (e.g. Bookkeeper, Accounting Manager) — used to optionally narrow a group by `job_title_or` when a pain phrase is too generic on its own.

   Example row for an expense-management product:

   - **Feature** — Expense management.
   - **What it does** — Lets employees submit expenses and finance reconcile and reimburse them.
   - **Pains / jobs-to-be-done** — "manually chasing receipts", "reconciling corporate card spend", "processing employee reimbursements", "month-end close".
   - **Owner roles** — Bookkeeper, Accounts Payable Clerk, Accounting Manager.

   This map is the seed for the keyword groups. One feature can spawn several groups; one group maps back to exactly one feature.

## Step 2 — Expand each pain into a keyword group (lots of synonyms)

3. **For each pain, brainstorm a WIDE set of phrasings** — the words a hiring manager would actually write. Go broad on purpose: verb and noun forms ("reconcile" / "reconciliation"), tool-agnostic descriptions of the manual process, common abbreviations, regional spellings, and **the same pain in every language spoken in the countries the user is targeting jobs in** (one language pass per target country — pain phrasing and role titles don't translate 1:1, so do a real per-language pass, not a literal translation). If the user targets jobs in Germany, France and Spain, cover German, French and Spanish; if only the US, English is enough. This first list is a hypothesis; Step 3 corrects it.

4. **Turn each group into a search filter.** Each group is an **OR set of regex patterns** passed to `job_description_pattern_or`. Conventions:
   - Make patterns **case-insensitive** with a leading `(?i)`, and **bound whole words/phrases** with `\b` so "expense" doesn't match "inexpensive". Example group: `["(?i)\\bexpense report\\b", "(?i)\\bchase (receipts|invoices)\\b", "(?i)\\breimbursement\\b"]`.
   - To require **two concepts together** (e.g. a pain AND a context, or a role AND a pain), use `job_description_pattern_and` with one alternation pattern per concept — every pattern in the list must match.
   - To **cut false positives**, add `job_description_pattern_not` with the noisy phrasings you find in Step 3.
   - For simple whole-word matches you don't need regex: `job_description_contains_or` matches complete words case-insensitively with built-in word boundaries.

## Step 3 — Ground & iterate against real job postings (the core loop)

Repeat this loop per keyword group until it **saturates** (a round adds no genuinely new phrasing) or you hit ~3–4 rounds:

5. **Size it (1 credit).** Call `search_jobs` as a count — `include_total_results: true`, `limit: 1`, `posted_at_max_age_days: 90`, plus the group's `job_description_pattern_or` (and any `_and` / `_not`). Read `metadata.total_results` (jobs) and `metadata.total_companies`.

6. **Read real postings (costs credits).** Pull a small sample — `limit: 5`–`10` — for the same filter, and read each `description`. This is the heart of the skill: see how companies actually phrase the pain.

   - **Mine new phrasings** the postings use that aren't in your group yet → add them.
   - **Spot false positives** — jobs that matched but don't really signal the pain → add the offending phrasing to `job_description_pattern_not`, or tighten the pattern.
   - **Pull sample jobs from different sources/geographies** (don't over-fit to one job board or one country) so the phrasings generalize.

7. **Re-run** the count with the refined group and note the delta. Stop when new rounds stop adding real phrasings.

8. **Cross-check overlap.** When two groups are near-duplicates, merge them; when one group secretly bundles two distinct pains, split it. The goal is a small set of clean, well-separated groups, each tied to one feature.

## Step 4 — Size and deep-link each group

9. **Final count (1 credit)** for each group with the converged filter, `posted_at_max_age_days: 90` applied.

10. **One deep-link per group** so the count opens the live, pre-filtered search — **do NOT create saved searches.** Call `generate_app_url` with `type: "job_search"` and `filters` set to the same filters you sized with, **minus the count-only flags** (`include_total_results`, `limit`). It returns a URL that opens the jobs search and runs it for whoever opens it. Put that URL on the count.

## Output

Write the output **in the user's language** (match the conversation). Professional tone, no emojis. Build a self-contained report in this order:

**Product summary** — a short heading with the product name, then a one-line **What they sell** and a one-line **Value proposition**, in the company's own framing.

**Keyword groups** — a table giving the at-a-glance map, with these columns:

| Feature | What it does | Keyword group | What the keywords signal | Jobs (90d) | Open in TheirStack |
| ------- | ------------ | ------------- | ------------------------ | ---------- | ------------------ |

- **Feature** and **What it does** come straight from the Step 1 product↔pain map (this is the product-description info the user asked to keep in the table).
- **Jobs (90d)** is the live count from Step 4; it is the hyperlink target if you prefer linking the number, otherwise put the deep-link in **Open in TheirStack**.

Then, **one short subsection per keyword group** (ordered by relevance to the product, most on-the-nose pain first), each containing:

- A one-line **explanation** of the buying signal — what hiring need the group detects and why it maps to this feature.
- The **final keyword list** (the converged patterns, shown readably — you don't need to print the raw regex escaping, but keep it copy-pasteable).
- The **job count** over the last 90 days and the **company count**, with the count linked to the deep-link job-search URL.
- One line of **iteration notes** when relevant — phrasings added from real postings, false positives excluded, or roles used to narrow (e.g. "added 'three-way match' and 'AP aging' from sampled postings; excluded 'expense' on its own — too noisy").

End with: a one-line note that each link opens the live, pre-filtered search in the app (nothing is saved to the account), the **posting window** the counts cover (90 days by default) and that they are a flow within it, a rough **credit cost** of the run (sizing and sample reads both count), and any group dropped for lack of real matches. If the user wants any group monitored over time, offer to turn it into a saved search with email alerts — but only on request.

## Notes

- **Buyer's words win.** The single most common failure is filling a group with the vendor's marketing vocabulary. Always reset to: _what would a hiring manager type into a job description when they have this pain?_
- **Iterate, don't guess.** The value of this skill over a one-shot prompt is Step 3 — reading real postings and refining. Don't skip it to save credits; just keep samples small.
- **Counts vs reads.** A count (`include_total_results`, `limit: 1`) costs 1 credit per group that has matches; reading real descriptions costs 1 credit per job returned. Never pull large batches just to size.
- **Combine for precision.** A generic pain phrase (e.g. "reporting") gets sharper when ANDed with a role (`job_title_or`) or a second concept (`job_description_pattern_and`). Single-phrase groups are broader and noisier.
- **Re-run per market.** Pain phrasing and role titles don't translate 1:1 — do a real per-language pass for every language spoken in the countries the user is targeting jobs in (as many languages as target countries), not a literal translation.
