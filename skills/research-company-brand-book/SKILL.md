---
name: research-company-brand-book
description: Research a company from its own website and fill its TheirStack brand book — what it sells, the problems it solves and who owns them, its ideal customer profiles, its customers and its competitors — then save it and reply with its link. The research step behind `find-theirstack-sales-signals` and `find-lookalike-companies`. Use when the user says "fill my brand book", "research my company", "build the brand book for <url>", "update the brand book", or pastes their own company URL to profile it.
user-invocable: true
argument-hint: <company-url> [extra URLs or free-text context]
---

# Research Company Brand Book

1. **Get the company URL.** Ask for it if missing. Extra URLs and free-text context win over your own inference. Run end-to-end without asking for confirmation.
2. **Check the TheirStack MCP.** If `find_technologies_and_topics` isn't available, stop and point the user to https://theirstack.com/en/docs/mcp.
3. **Read the current brand book.** Call `get_brand_book`. When it describes the same website, start from it: what a person wrote there wins, and fix the profiles listed in `filter_problems`.
4. **Read the website.** The landing, then in one parallel batch its product, solutions, pricing, integrations and customers pages, and at most 5 case studies. Don't research the company with `search_companies` or `search_jobs`, and use web search only to find competitors the site doesn't name, or what it sells when the website refuses you; if you still know nothing, don't save, and say why. Read at least 8 pages when the site has them, including its customers and integrations pages.
5. **Write down what the company is.** Up to 6 main products, the problems they solve and the job titles that own them, up to 5 ideal customer profiles, the customers its site names (never guessed), its competitors, and its integrations marked must-use or nice-to-have. Infer its buyers' countries, industries and size; don't ask. Cover at least 6 problems, every customer the pages name with its page as `case_study_url`, every integration the pages name, and at least 6 competitors, searching the web once if the site names too few. Exactly one product is `is_main`. A profile's `filters` take the filter names of `search_companies`, such as `company_country_code_or` (ISO-2), `industry_or`, `min_employee_count`, `max_employee_count` and `company_technology_slug_or`.
6. **Resolve the technologies.** In one parallel batch, look up with `find_technologies_and_topics` (`keyword_type: "technology"`, `limit: 3`) the integrations and the competitors that are software. Skip service firms, and keep industries as names.
7. **Save it.** Write it once, in the save call, without the integrations, which the brand book has no field for: where the calling skill says, or with `save_brand_book` when it is the workspace's own company. Never drop what a person wrote, and never save anything else.
8. **Reply with the link only.** `https://app.theirstack.com/workspace/{team_id}/settings/brand-book`, or the calling skill's link, and no notes. If you didn't save, reply with the brand book as compact JSON in `save_brand_book`'s shape. When a skill called you, also return the integrations and the buyers' countries, industries and size as compact JSON.
