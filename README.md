# glassdoor scraper python: Why Every Old Tutorial Fails Now, and How to Pull Jobs, Reviews & Salaries Without Getting Blocked

Most "glassdoor scraper python" code you'll find online is dead. Requests-based scrapers get a 403 before the first byte of HTML loads, Selenium scripts crash on a login form that no longer exists, and Glassdoor's old public API now returns a 410 Gone. Since Glassdoor was folded into Indeed in July 2025, you need either a real browser session with residential proxies and careful pacing, or a scraping API that handles all of that for you. This article covers both paths, then walks through ScraperAPI's actual pricing — including the credit math that most reviews skip — so you know what a Glassdoor scrape really costs before you commit.

> **Quick pick:** Start with the free tier (1,000 credits, 5 concurrent connections) to verify Glassdoor works for your use case, then move to **Hobby at $49/month** (100,000 credits, 20 threads) for any real project. Annual billing takes 10% off every plan. 👉 [Check current plans and start the free 5,000-credit trial](https://www.scraperapi.com/signup?fp_ref=coupons)

## Why your old Glassdoor scraper stopped working

Glassdoor didn't just get harder to scrape — the attack surface you learned in 2022 has been removed entirely.

Three separate things happened. First, Recruit Holdings folded Glassdoor into Indeed in July 2025 and tightened the anti-bot stack. As of February 2026, Glassdoor logins are handled entirely through Indeed Login, which means any script that hard-codes a Glassdoor-specific login form is structurally broken. Second, the old public partner API at api.glassdoor.com now returns **410 Gone** — it's permanently shut down, and there was never a public endpoint for reviews to begin with. Third, Cloudflare Bot Management now sits in front of every Glassdoor URL: a recent field test against glassdoor.com in April 2026 confirmed HTTP 403 on every endpoint tested — job search, reviews, overview pages, salaries, even sitemap.xml — across every User-Agent tried, including a full Chrome string and Googlebot.

The technical reason is worth understanding, because it explains why the usual tricks don't help. Glassdoor's Cloudflare setup runs a "managed challenge": JavaScript probes your browser fingerprint right after the page loads, no CAPTCHA click required. The `__cf_bm` cookie returned in the 403 response is bound to your browser's TLS fingerprint, so replaying it from a Python session gets you nowhere. Changing headers changes nothing — your TLS handshake identifies you as Python before any header is read.

There's also a login wall layered on top. Job listings are partially viewable without an account, but full review text (the pros/cons) and salary data require being logged in, and salary pages can add a "contribute to unlock" flow where you must submit your own salary report before seeing anyone else's.

## What actually works in 2026

Two viable paths remain, and the right one depends on how much engineering you want to own.

**Path 1: a scraping API like ScraperAPI.** You send your Glassdoor URL to a single endpoint with parameters like `render=true` and `country_code=us`, and the service runs real browsers through residential proxy pools, solves the Cloudflare challenge, and returns HTML or parsed JSON. You never manage proxies, fingerprints, or retry logic. The tradeoff is that you pay per request.

**Path 2: self-managed browser automation.** The current open-source consensus is Patchright — a patched fork of Playwright that fixes the CDP `Runtime.Enable` leak that gets vanilla Playwright detected on Glassdoor's login page. It's actively maintained (version 1.58.2, released March 2026), and the recipe involves logging in once manually, saving the authenticated session state, reusing it for runs, rotating residential proxies, and waiting 5+ seconds per page for the Cloudflare challenge to resolve. There's also a fast path: some Glassdoor pages ship their data in a `__NEXT_DATA__` JSON blob in the initial HTML, and a client like curl_cffi can pull it 10–20x faster than a full browser — but it can't handle login or challenge pages.

What no longer works, for the record: vanilla Playwright (detected on the login page), Selenium with undetected-chromedriver (last release February 2024, effectively legacy — one test suite found it "failed on every domain in our test"), and requests + BeautifulSoup (blocked instantly by TLS fingerprinting). The most-starred legacy Glassdoor Selenium repo on GitHub — roughly 1.4k stars — has 12+ unresolved issues and is effectively abandoned. If a tutorial's code dates from 2021–2023, assume it's a fossil.

## ScraperAPI plans and pricing, all in one table

ScraperAPI prices by **API credits** per request, across seven paid tiers plus a free tier. Annual billing takes **10% off** every plan. All paid plans include JS rendering, premium proxies, CAPTCHA and anti-bot handling, automatic retries, custom sessions and headers, unlimited bandwidth, and a 99.9% uptime guarantee.

| Plan | Monthly price | Annual (per mo) | Credits/mo | Concurrent threads |
| --- | --- | --- | --- | --- |
| Free | $0 | — | 1,000 | 5 |
| Hobby | $49 | $44.10 | 100,000 | 20 |
| Startup | $149 | $134.10 | 1,000,000 | 50 |
| Business | $299 | $269.10 | 3,000,000 | 100 |
| Scaling | $475 | $427.50 | 5,000,000 | 200 |
| Professional | $975 | $877.50 | 10,500,000 | 300 |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 |
| Enterprise | Custom | — | 22,000,000+ | 500+ |

A few differences don't fit in the table columns:

- **Geotargeting:** Hobby and Startup are limited to US & EU exit nodes. Business and up get country-level targeting worldwide.
- **Analytics:** Hobby and Startup only show the last 30 days of usage; Business and up have unlimited history.
- **Overage behavior:** Hobby through Professional can opt into auto-upgrade (your account moves to the next plan when credits run out). Scaling and up additionally support pay-as-you-go at a fixed per-credit rate, capped at a monthly spending limit you set.
- Enterprise is quote-based, aimed at 22M+ credits per month, with dedicated support and Slack access.

Every plan's buy button below points to ScraperAPI's pricing page with the referral parameter attached, so you can compare tiers with tracking intact:

- Hobby — 👉 [Get Hobby at $49/mo](https://www.scraperapi.com/pricing?fp_ref=coupons)
- Startup — 👉 [Get Startup at $149/mo](https://www.scraperapi.com/pricing?fp_ref=coupons)
- Business — 👉 [Get Business at $299/mo](https://www.scraperapi.com/pricing?fp_ref=coupons)
- Scaling — 👉 [Get Scaling at $475/mo](https://www.scraperapi.com/pricing?fp_ref=coupons)
- Professional — 👉 [Get Professional at $975/mo](https://www.scraperapi.com/pricing?fp_ref=coupons)
- Advanced — 👉 [Get Advanced at $1,975/mo](https://www.scraperapi.com/pricing?fp_ref=coupons)
- Enterprise — 👉 [Talk to sales about Enterprise](https://www.scraperapi.com/contact-sales?fp_ref=coupons)
- Free — 👉 [Claim the free 1,000-credit plan](https://www.scraperapi.com/?fp_ref=coupons)

## What a Glassdoor request actually costs

Here's the part most ScraperAPI reviews gloss over: "100,000 credits per month" does not mean 100,000 Glassdoor pages.

A standard page costs 1 credit. But Glassdoor sits behind Cloudflare, and ScraperAPI charges **10 credits per request** whenever it has to bypass bot protection (Cloudflare, Datadome, PerimeterX, Akamai all cost 10). Add `render=true` for JavaScript rendering and that's another 10 credits, on top of the base. So a realistic Glassdoor job-page request with rendering looks like:


base request        1 credit
bot-protection bypass   +10 credits
render=true         +10 credits
────────────────────────────
per page           ~21 credits


Do the math on Hobby: 100,000 credits ÷ ~21 = roughly 4,700 rendered Glassdoor pages per month, not 100,000. That's still a solid run — a few hundred companies' review pages, or a few thousand job listings — but budget with the real number, not the headline one.

Three ways to keep the number down:

1. **Check before you scrape.** The dashboard's Domain Cost Estimator (called Domain Multiplier) tells you the exact credit cost of any URL, and there's a no-GUI version: `https://api.scraperapi.com/account/urlcost?api_key=YOUR_KEY&url=...`. It accepts all the same parameters, so you can price a request with `render=true` included before you fire it.
2. **Try the fast path first.** If your target Glassdoor page ships `__NEXT_DATA__` in the initial HTML, you may not need `render=true` at all — that alone cuts the cost by 10 credits per request. Test one URL in the API Playground and inspect the response before committing a whole job to rendering.
3. **Skip premium flags you don't need.** `premium=true` adds 10 credits, `ultra_premium=true` adds 30, and stacking them with rendering gets expensive fast (25 and 75 credits respectively). For Glassdoor, premium proxies come as part of the anti-bot handling anyway.

A few parameters are free, which matters for a Glassdoor scraper specifically: `autoparse=true` (structured JSON output), `country_code` (important if you need reviews from a specific Glassdoor country domain), `session_number` (sticky sessions for paginating through a company's reviews), and `device_type`. Use them — they cost nothing extra.

Also worth knowing: you're only charged for successful requests — status 200 and 404 count, everything else doesn't. But if you cancel a request yourself within 70 seconds, it still bills.

## A minimal Python example

Once you have an API key from the dashboard, a Glassdoor request is one line of HTTP plus whatever parsing you'd do anyway:

python
import requests

API_KEY = "your_key_here"
url = "https://www.glassdoor.com/Job/jobs.htm?sc.keyword=data+engineer"

payload = {
    "api_key": API_KEY,
    "url": url,
    "render": True,        # Glassdoor needs JS execution
    "country_code": "us",  # free, sets the exit country
    "autoparse": True,     # free, returns structured JSON where supported
}

resp = requests.get("https://api.scraperapi.com/", params=payload, timeout=90)
print(resp.headers.get("sa-credit-cost"))  # actual credits charged for this request
print(resp.status_code, len(resp.text))


That `sa-credit-cost` response header is the cheapest debugging tool you have: log it on every request during development and you'll know your true per-page cost before the month's credits vanish. Set a `max_cost` per request as a hard ceiling so no single scrape can eat your quota.

Pacing still matters even with the API doing the heavy lifting. The community guidance for Glassdoor is random delays of at least 3–8 seconds between requests (5–15 for long jobs) regardless of proxy quality — hammering a company's review pagination at machine speed is a good way to burn credits on challenges.

## The fine print that changes your plan choice

Several details live only in the pricing FAQ and billing docs, and two of them regularly surprise people:

- **Concurrency overage is a 429, not a queue.** If you exceed your plan's thread limit, extra requests are simply turned away — there's no charge, but there's also no automatic queuing. Your scraper needs its own retry logic for 429s. On Hobby that's 20 concurrent requests; it fills faster than you'd think if you're fan-out scraping 50 companies at once.
- **Geotargeting is plan-gated.** Hobby and Startup can only exit from US and EU nodes. Glassdoor serves region-specific content, so if you need reviews or salary data from, say, Japan or Brazil as they appear to local users, you need Business ($299/mo) or higher for country-level targeting.
- **Free credits have a ceiling.** The permanent free plan gives 1,000 credits with a max of 5 concurrent connections. Since a single bypassed Cloudflare request costs 10 credits, that's on the order of a hundred Glassdoor pages before you're dry — enough to validate your selectors, not enough for a dataset. If you need more for testing, support can grant additional test credits.
- **Running out of credits pauses you by default.** Scraping stops until renewal unless you've enabled auto-upgrade or (on Scaling and up) pay-as-you-go. For a scheduled weekly job this is fine; for a time-sensitive project, set it up in advance.

If any of these constraints matter to your project, it's worth scanning the full comparison before picking a tier: 👉 [Compare all plan limits on the pricing page](https://www.scraperapi.com/pricing?fp_ref=coupons)

## When ScraperAPI is the wrong tool

Honesty helps here: if you're scraping Glassdoor once, for a one-off dataset, a paid subscription may be overkill.

The DIY route works if you already own infrastructure. The Patchright approach needs Python 3.10+, Chrome, a free Glassdoor account (mandatory for salary and review data), rotating residential proxies at roughly $3–8/GB for production-grade traffic, and about 30–60 minutes of setup plus 10 minutes per data type afterward. If you already pay for residential proxies and have the code skeleton from another project, adding Glassdoor costs you an evening, not $49/month.

There are also other scraping APIs with published Glassdoor scrapers. ScrapFly maintains an open-source Glassdoor scraper (Python 3.10, scrapfly-sdk, installed via Poetry) covering job listings, salaries, and reviews, with a companion tutorial on their blog — positioned as educational code rather than a supported product, and you'd need a ScrapFly account and API key to run it. It's a legitimate alternative; compare credit pricing and Glassdoor-specific handling on your own before deciding.

Where ScraperAPI earns its price is the maintenance burden you offload: Glassdoor's selectors and anti-bot setup have churned repeatedly since mid-2025, and a managed API absorbs that churn on their side. The most-starred community scraper going unmaintained with a dozen open issues is what happens to the DIY path when nobody's paid to keep up.

## FAQ

**Can I still scrape Glassdoor without logging in?**
Partially. Job listings expose titles, companies, locations, salary estimates, ratings, and links to anonymous visitors, though login prompts can appear after a few pages of pagination. Full review text and salary data require an authenticated session — and since February 2026, that login runs through Indeed's system, so your scraper needs a session that survived a real login.

**Is there a Glassdoor API?**
No usable one. The old partner API at api.glassdoor.com returns 410 Gone, there has never been a public endpoint for reviews, and Indeed's Publisher API offers no migration path for Glassdoor review or salary data. That gap is precisely why scrapers exist for this site.

**How many Glassdoor pages can I scrape per month on the $49 Hobby plan?**
Depends on your parameters. With rendering and a Cloudflare bypass in play, budget roughly 20+ credits per page — call it 4,000–5,000 pages. If some pages render server-side data without needing `render=true`, you'll stretch considerably further. Log the `sa-credit-cost` header for a week and you'll have your own real number.

**Does ScraperAPI charge for failed requests?**
Only 200 and 404 responses bill. Blocked, timed-out, or errored requests don't consume credits — though a request you cancel within 70 seconds does. Concurrency overages (429) are free.

**Is scraping Glassdoor allowed?**
Glassdoor's terms prohibit scraping — the maintainer of one of the best-known open-source Glassdoor scrapers states this plainly and warns that accounts can be banned. Rate-limit your requests, use official data exports where they exist, and weigh the risk for your use case.

## Which plan should you actually pick

Match the tier to the job:

- **Validating an idea or building selectors** → free tier (1,000 credits) or the 7-day, 5,000-credit trial, no credit card needed. Confirm Glassdoor works for your target data type before spending anything.
- **Ongoing side project, a few thousand pages a month** → Hobby at $49/month ($44.10 annual). Just remember the US/EU geotargeting limit and the 30-day analytics window.
- **Recruiting analytics, competitor monitoring, lead-gen pipelines** → Business at $299/month is the first tier with country-level targeting and unlimited usage history, and 3M credits covers a lot of Glassdoor.
- **Multi-site production scraping** (Glassdoor plus Google, LinkedIn, e-commerce) → Scaling at $475/month, where pay-as-you-go unlocks and the credit multiplier differences across sites stop hurting as much.

One last check before you subscribe: run two or three of your actual target URLs through the `urlcost` endpoint with your exact parameter set, multiply by your monthly page volume, and confirm the result fits inside the plan's credit allowance with headroom to spare. 👉 [Sign up for the free trial and run your first cost check](https://www.scraperapi.com/?fp_ref=coupons)
