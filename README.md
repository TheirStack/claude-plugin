# TheirStack for Claude

TheirStack tracks job postings from company career sites and job boards in 195 countries, and reads them to work out which technologies each company uses. This plugin connects Claude to your TheirStack account. You ask about hiring and tech stacks in plain language, and Claude answers from our data.

## What you can ask

- Find jobs by title, company, location, technology, salary, seniority or remote policy, posted in the last day or the last year.
- Find companies that use a technology (Snowflake, Salesforce, Kubernetes and around 33,000 others), filtered by country, industry, size or the roles they are hiring for.
- List the technologies a specific company uses, with a confidence level and the dates we first and last saw each one in their job posts.
- See what a company is investing in, from the topics that come up in its job descriptions.
- Open any search in the TheirStack app to refine it or export the results.
- Save a search, add companies to a list or set up a webhook for new matches, without leaving the chat.

Some examples:

- "Which companies in Germany use Snowflake and are hiring data engineers?"
- "What technologies does Stripe use?"
- "Show me remote senior Python jobs posted this week"

## Skills

The plugin comes with three research workflows:

- `find-theirstack-sales-signals`: give it your product's website and it finds the hiring and tech stack signals your sales team should prospect on.
- `find-lookalike-companies`: give it a few customers you have won and it finds companies that look like them now.
- `job-description-keyword-research`: give it your product's website and it finds the phrases companies use in job posts when they have the problem you solve.

## Setup

Install the plugin, then connect your TheirStack account when Claude asks. Sign-in uses OAuth, so you never paste an API key. You need a TheirStack account; you can [sign up for free](https://app.theirstack.com).

## What it connects to

- The TheirStack MCP server at `https://api.theirstack.com/mcp`, over HTTPS with OAuth. Your questions are turned into searches against your TheirStack workspace. Searches use the API credits on your workspace (1 credit per job, 3 per company), the same as the TheirStack API. Looking up locations, industries and technology names is free.
- Websites you point it at: the skills read the product website you give them, using Claude's own web tools, to understand what you sell.

The plugin runs no local code, installs no packages and sends nothing anywhere else.

## Support and privacy

- Documentation: https://theirstack.com/en/docs/mcp
- Support: support@theirstack.com or https://theirstack.com/en/contact
- Privacy policy: https://theirstack.com/en/docs/legal/privacy-policy
- Terms: https://theirstack.com/en/docs/legal/terms-and-conditions
