---
name: linkzoid
description: Audit and improve a website's internal linking through the LinkZoid MCP connector - crawl the site, find orphan and under-linked pages and broken links, map topics, run an AI analysis that suggests links with anchor text, and approve the links so the LinkZoid script places them on the live site. Use when the user wants to improve internal links or SEO structure on a site, asks which pages are orphaned or under-linked, wants broken links found, or asks about their LinkZoid sites, suggestions or credits. Do not use for general SEO questions that need no site data.
version: 1.0.0
author: LinkZoid
license: MIT
compatibility: Requires the LinkZoid MCP connector at https://linkzoid.xyz/mcp (Streamable HTTP, OAuth 2.1 with Dynamic Client Registration) and network access.
metadata:
  tags: "SEO, Internal linking, Websites, Content"
  category: "seo"
---

# LinkZoid

LinkZoid crawls a website, works out which pages should link to each other, and suggests the exact anchor text. The links the user approves go live through one small LinkZoid script tag on their site, so no page is edited by hand. Crawling, topic maps and link checks are free; an analysis spends 1 credit per page analyzed.

## Setup

The tools come from the `linkzoid` MCP server (`mcp__linkzoid__*` in Hermes). It signs in with OAuth. If no `mcp__linkzoid__` tools are available, the connector is not set up yet: tell the user to run this command once, sign in on the linkzoid.xyz page it opens, and start a new session, rather than trying the calls.

```bash
hermes mcp add linkzoid --url https://linkzoid.xyz/mcp --auth oauth --connect-timeout 300
```

A free LinkZoid account is enough to sign in. Analyses need the account's email to be confirmed.

## How the tools fit together

- **Start:** `get_account` shows the plan and credits left; `list_sites` gives each site's `site_id`.
- **A new site:** `add_site` takes the site's address (or a sitemap URL), finds its sitemap and starts a crawl. If it reports no sitemap, ask the user for the sitemap URL and pass it as `sitemap_url`. `start_crawl` refreshes a site that is already added.
- **Background jobs:** crawls, analyses and link checks return a job. Call `get_job` with its `job_id` until it is finished before reading results; tell the user it is running rather than calling it in a tight loop.
- **What needs work:** `get_site_overview` is the summary. `list_pages` with `filter` set to `orphans` or `underlinked` finds pages that need links; `get_page` shows one page's links in and out. `list_topics` and `list_topic_pages` show the topic map.
- **Link suggestions:** `estimate_analysis` says how many pages an analysis covers and what it costs. `start_analysis` runs it. `list_opportunities` shows the suggested links (source, target, anchor text, score); `review_opportunities` approves, dismisses, restores or retries them.
- **Broken links:** `start_link_check`, then `list_broken_links`; `mark_broken_link_fixed` once the user has fixed one.
- **Going live:** `get_install_snippet` gives the script tag for the site's pages (before `</body>`); `check_script_installation` confirms it works.

## Rules

- **Credits:** before `start_analysis`, run `estimate_analysis`, show the user the page count and credits it will use, and wait for their go-ahead, unless they already asked for exactly that analysis. Never re-run an analysis to "check" it.
- **Approving goes live:** an approved link is placed on the user's real website by the LinkZoid script. Show the user the suggestions you mean to approve (anchor text, source and target page) and get their confirmation before calling `review_opportunities` with `approve`, especially in bulk. `dismiss` and `restore` are safe to do when asked.
- **Script not installed:** approved links wait until the script is on the site. If `check_script_installation` says it is missing, give the user the snippet from `get_install_snippet` instead of approving more.
- **Not in the connector:** deleting a site, changing the plan and team management happen in the web app at https://linkzoid.xyz. Point the user there.

## Showing results

Lead with what needs doing: the number of orphan and under-linked pages, broken links, and suggestions waiting for review. List suggestions as `"anchor text" → target page (on source page, score)`, best first, a handful at a time. Give page titles with their paths, not raw IDs.
