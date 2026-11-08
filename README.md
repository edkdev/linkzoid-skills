# LinkZoid plugin for Hermes Agent

Audit and fix your website's internal linking from inside Hermes Agent. Ask which pages are orphaned, where your broken links are, or what each blog post should link to. Then approve the links you want, and they go live on your site without editing a single page.

LinkZoid crawls your site from its sitemap, groups pages into topics, and uses AI to find the pages that should link to each other, with the exact words to link from. Approved links are placed by one small LinkZoid script tag on your site, so it works on WordPress, Webflow, Shopify, Squarespace or any site where you can add a script.

One install ships `skills/linkzoid/`, the workflow skill: how the crawl, analysis and review tools fit together, and the rules that keep it safe (show the credit estimate before an analysis, confirm before approving links that go live).

## Install

```bash
hermes plugins install linkzoid   # from the Hermes plugin catalog
hermes plugins enable linkzoid
```

Later versions arrive with `hermes plugins update linkzoid`.

## Sign in (once)

LinkZoid signs in with OAuth 2.1 and Dynamic Client Registration. There is no API key, and nothing secret lives in this repository. The server itself (`https://linkzoid.xyz/mcp`, Streamable HTTP) is added with one command, because a plugin cannot tell Hermes to sign in with OAuth:

```bash
hermes mcp add linkzoid --url https://linkzoid.xyz/mcp --auth oauth --connect-timeout 300
```

Hermes prints a linkzoid.xyz link (and opens it when it can): sign in or create a free account there and approve access. Keep `--connect-timeout 300`: it gives you five minutes on that page. Then start a new Hermes session, or run `/reload-mcp` in an open one.

## Try it

- "Add example.com to LinkZoid and tell me which pages are orphaned."
- "Which of my blog posts have two or fewer internal links pointing at them?"
- "How many credits would it cost to analyze my 'pricing' topic? Go ahead if it's under 20."
- "Show me the best link suggestions for my site."
- "Approve the suggestions above 85% match."
- "Check my site for broken links."
- "Is the LinkZoid script installed on my homepage?"

## Tools

Hermes exposes each tool as `mcp__linkzoid__<tool>`. The server's own tool list is the source of truth.

| Area | Tools | Access |
|---|---|---|
| Account and sites | `get_account`, `list_sites`, `get_site_overview` | Read |
| Sites | `add_site`, `start_crawl` | Write |
| Pages and topics | `list_pages`, `get_page`, `list_topics`, `list_topic_pages` | Read |
| Analysis | `estimate_analysis` | Read |
| Analysis | `start_analysis` (1 credit per page) | Write |
| Link suggestions | `list_opportunities` | Read |
| Link suggestions | `review_opportunities` (approve places links on the live site) | Write |
| Broken links | `list_broken_links` | Read |
| Broken links | `start_link_check`, `mark_broken_link_fixed` | Write |
| Script | `get_install_snippet`, `check_script_installation` | Read |
| Jobs | `get_job` | Read |

Deleting sites, changing the plan and team management are only available in the web app at https://linkzoid.xyz.

## Privacy and security

- Hosted-only connector: this folder is the whole plugin. It has no Python code, hooks or environment variables, and it does not update itself. Every tool call goes to `https://linkzoid.xyz/mcp` over HTTPS.
- Access is scoped to the signed-in LinkZoid account and the sites it (or its team) owns.
- LinkZoid fetches the public pages of the sites you add, and `check_script_installation` fetches the page you name.
- Approving a suggestion changes your live website: the LinkZoid script inserts that link on the source page. Dismissed or restored suggestions change nothing on the site.
- An analysis spends credits from your LinkZoid plan. The skill asks Hermes to show the estimate and wait for your go-ahead first.
- Privacy policy: https://linkzoid.xyz/legal/privacy · Terms: https://linkzoid.xyz/legal/terms

## Troubleshooting

**No LinkZoid tools in the session.** Run the sign-in command above, then start a new session. `hermes mcp test linkzoid` should report the tools.

**`hermes mcp add` timed out, or the browser shows "This site can't be reached" after you approve.** Hermes stopped listening before you approved. Answer `n` to "Save config anyway?", then run the sign-in command again with `--connect-timeout 300`.

**Tools fail with 401 or "unauthorized".** The sign-in lapsed. Run `hermes mcp login linkzoid`, then start a new session.

**An analysis is refused.** Confirm your email from the link LinkZoid sent you, or check your credits with "How many LinkZoid credits do I have left?".

## Other clients

The same server works in any MCP client: add `https://linkzoid.xyz/mcp` as a remote (Streamable HTTP) server with OAuth. Setup guide: https://linkzoid.xyz/mcp-server

## Support

- E-mail: Edmond@linkzoid.xyz
- Website: https://linkzoid.xyz

## License

This folder (plugin manifest, skill and docs) is MIT licensed. The hosted LinkZoid service and MCP server are separate and governed by the LinkZoid terms.
