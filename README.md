# residential proxy api: what a $49 plan really buys, how credits are charged, and when it beats raw proxies

Most people searching for a residential proxy API want one of two very different things, and mixing them up is the fastest way to buy the wrong product. If you want to send a request like `https://api.scraperapi.com/?url=...` and get clean HTML back — with proxy rotation, CAPTCHA solving, JavaScript rendering, and retries all handled on the server side — you're looking for a scraping API, and ScraperAPI is one of the established options in that category: paid plans start at $49/month, every plan includes premium residential and mobile IPs, and a 7-day free trial with 5,000 credits needs no credit card. If instead you already run your own proxy rotation logic and just want a REST API to manage proxy accounts and traffic quotas, that's a different product category (Decodo, ProxyApp, Proxy-seller all sell those), and you'll be wasting money on ScraperAPI.

This article covers the second kind of confusion only briefly. The bulk of it is about the first: what ScraperAPI's plans actually cost, how its credit system changes your real price per page, and where it quietly gets expensive.

> **Quick picks**
> For most individuals testing residential-proxy-backed scraping, the **Hobby plan at $49/month ($44.10/month on annual billing)** with 100,000 credits and 20 concurrent threads is the realistic entry point. If your targets are plain HTML pages without heavy bot protection, that allocation stretches far. If you're hitting Google SERPs or Cloudflare-protected sites, read the credit multiplier section below before paying for anything.
> 👉 [Check ScraperAPI's current plans and start the 7-day free trial](https://www.scraperapi.com/pricing?fp_ref=coupons)

## "Residential proxy API" means two different products. Which one do you need?

The search results for this term are split between two product shapes, and vendor pages don't always make the distinction obvious.

The first shape is what ScraperAPI sells: an all-in-one scraping endpoint. You pass a target URL plus parameters like `country_code=us`, `render=true`, or `premium=true`, and the service picks a residential IP from its pool (40M+ IPs across 50+ countries, per its own pricing page), handles CAPTCHAs and bot-protection bypasses, renders JavaScript if asked, and returns the HTML. You never touch an individual IP. Billing is per request, via credits.

The second shape is a management API on top of a traditional residential proxy pool. Decodo's residential proxy API, for example, lets you create sub-users, set traffic limits, and monitor usage in real time — but you still connect through standard proxy endpoints with your own rotation logic, and you pay per gigabyte (its plans run from $3.75/GB at 3GB down to $2.75/GB at 100GB, with a 14-day money-back option). ProxyApp and Proxy-seller work similarly: package-based traffic pools, sticky sessions, and REST endpoints for extending subscriptions or adjusting rotation between 1 and 3,600 seconds.

A third, smaller group of results — IPinfo Max — is the inverse problem entirely: detecting residential proxy traffic rather than using it. Ignore it unless you're building fraud filters.

If you already have scraping code and just need better IPs, the per-GB management-API products are usually cheaper per unit of bandwidth. If you want the whole anti-bot problem off your plate, a scraping API is the simpler purchase. The rest of this article is about the second choice, using ScraperAPI as the reference.

## Full plan breakdown: all seven paid tiers, plus the free options

ScraperAPI's pricing page currently shows seven paid plans, each with a monthly and an annual rate. Annual billing saves 10% across the board. Here's the complete list:

| Plan | Monthly | Annual (per month) | Credits | Concurrency | Link |
| --- | --- | --- | --- | --- | --- |
| Hobby | $49/mo | $44.10/mo | 100,000 | 20 | [Try Hobby free for 7 days](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | $149/mo | $134.10/mo | 1,000,000 | 50 | [See Startup plan pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | $299/mo | $269.10/mo | 3,000,000 | 100 | [See Business plan pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | $475/mo | $427.50/mo | 5,000,000 | 200 | [See Scaling plan pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | $975/mo | $877.50/mo | 10,500,000 | 300 | [See Professional plan pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | $1,975/mo | $1,777.50/mo | 21,500,000 | 500 | [See Advanced plan pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [Request an Enterprise quote](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Beyond the paid tiers, there are two free ways in. A permanent free plan gives you 1,000 credits per month with 5 concurrent connections, no card required. On top of that, your first 7 days after signup include 5,000 free requests for larger-scale testing — also without a card. The official docs say you can contact support for additional testing credits if 5,000 isn't enough to validate your targets.

One billing detail that applies to every tier: all plans include JS rendering, premium residential and mobile IPs, CAPTCHA and anti-bot handling, automatic retries, unlimited bandwidth, and a 99.9% uptime guarantee. There's no feature paywall between Hobby and Enterprise; what changes is volume, concurrency, and (importantly) geotargeting, covered below.

## The credit system: why "100,000 credits" rarely means 100,000 pages

This is the part most comparison articles mention in one line and then skip. It's also the part that determines whether ScraperAPI is cheap or expensive for your specific use case.

A standard page request costs 1 credit. But ScraperAPI applies automatic multipliers based on the target domain and your parameters:

- **E-commerce sites** (Amazon, Walmart, eBay): 5 credits per request
- **Search engines** (Google, Bing, including subdomains): 25 credits per request
- **LinkedIn**: 30 credits per request
- **Bot-protected sites** (Cloudflare, Datadome, PerimeterX, Akamai): +10 credits per request when the bypass triggers
- **`render=true`** (JavaScript rendering): +10
- **`premium=true`** (higher-quality proxy pool): +10
- **`ultra_premium=true`** (paid plans only): +30
- **Combos don't add linearly**: `premium` + `render` costs 25 credits together; `ultra_premium` + `render` costs 75

The official docs are explicit about one consumer-friendly rule: **you're only charged for successful requests** — HTTP 200 and 404 responses — plus requests you cancel yourself before the 70-second timeout. Every response also includes a `sa-credit-cost` header showing exactly what that request cost, and the dashboard's Domain Cost Estimator lets you look up the credit price of any URL before you run a job. You can set a `max_cost` parameter to cap what any single request may spend.

Here's what those numbers do to the Hobby plan's 100,000 credits:

| Target type | Cost per request | Pages on Hobby (100K credits) |
| --- | --- | --- |
| Standard HTML page | 1 | 100,000 |
| JS-rendered page | 10 | 10,000 |
| Amazon product page | 5 | 20,000 |
| Google SERP | 25 | 4,000 |
| Ultra-premium + JS rendering | 75 | 1,333 |

That last row is the trap. One independent review put it bluntly: the Hobby plan's "100,000 credits" is actually 1,333 scrapes on heavily protected pages with full rendering. For plain news sites and lightweight blogs with no JavaScript, you genuinely get the full 100,000. Know which row you live in before you pick a plan.

Two practical habits make this system work in your favor. First, run your target URLs through the Domain Cost Estimator or the API Playground before committing to a plan — a five-minute check tells you which multiplier row applies to you. Second, remember that `country_code`, `session_number`, `autoparse`, and `device_type` cost nothing extra, so geotargeting and sticky sessions are free to use.

## The limitations nobody leads with

ScraperAPI's pricing page is honest about what each plan includes, but three constraints only become clear when you read the fine print or the critical reviews.

Geotargeting is gated by plan. Hobby and Startup (that's everything up to $149/month) can only target US and EU locations. Full country-level targeting starts at Business, $299/month. One reviewer called this "a hard jump," and it's hard to argue: if your targets are Japanese or Brazilian sites, the two cheapest plans don't apply to you.

Concurrency rejections are hard stops. Exceed your plan's thread limit and you get a 429 response — the request is refused, not queued, and it isn't billed. That's fair pricing, but it means your code needs to handle 429s gracefully or you'll lose requests during bursts.

Third-party benchmark results are mixed. Independent testing reported by one review site measured overall success rates around 62–64%, and reviewers at Scrapeway noted that ScraperAPI forces a 10-minute result cache on difficult targets, which can return stale data. Official marketing claims a 99.9% uptime guarantee — that's a service-level commitment, not a scraping success rate. Against lightly protected targets, the multipliers barely matter and your effective success rate will be much higher than those aggregate benchmarks suggest. Against Cloudflare-hardened pages, budget for retries.

## What happens when you run out of credits (and how to get out)

Credits running out doesn't mean surprise overage charges by default. The default behavior is that scraping pauses. What you can do next depends on your plan:

1. **Auto-upgrade** (opt-in, available on Hobby through Professional): when credits run dry, your account automatically moves to the next plan, which usually has a better per-credit rate.
2. **Pay-As-You-Go** (Scaling, Professional, Advanced, Enterprise): keep scraping past your limit at a fixed per-credit rate, capped by a monthly spending limit you set yourself. The docs note this cap is a spending limit, not a pre-authorization.
3. **Annual plans**: renew early to refresh your credit allowance, or switch to a higher monthly tier.

Cancellation is free and can be done anytime from the dashboard or by contacting support. ScraperAPI's official docs also state a **7-day, no-questions-asked refund policy** on paid plans — if your first week of real workloads shows the credit math doesn't work for your targets, you're not stuck with the bill. That refund window pairs well with the free trial: use the 5,000 trial credits to measure your actual cost per page first, and the refund policy covers the gap between "tested a little" and "ran a real job."

One thing to check at checkout: a promo code `START10` (10% off your first month on any paid plan) circulates widely on third-party deal sites as recently as mid-2026, but it doesn't appear on ScraperAPI's own pricing page. Treat it as unverified until the checkout page confirms it applies.

## How the math compares against per-GB residential proxy providers

Scraping APIs look expensive next to raw residential proxies until you price in what the proxy vendors leave out. A few verified reference points:

| Product | Pricing model | Entry price | What's included |
| --- | --- | --- | --- |
| ScraperAPI | Per credit | $49/mo (100K credits) | Residential + mobile IPs, JS rendering, CAPTCHA solving, bypasses, parsing |
| Decodo residential proxies | Per GB | $3.75/GB (3GB plan) | 115M+ IPs, city/ASN targeting, SOCKS5; rendering and CAPTCHA solving are your problem |
| Oxylabs Web Scraper API (Micro) | Per month | $49/mo | Comparable scraping-API shape, per third-party comparison data |
| Oxylabs Residential Proxies (Micro) | Per month | $99/mo | Raw residential access, per third-party comparison data |
| Apify platform | Per month | $29/mo | Actor-based scraping platform with a free tier |

Third-party comparison sites list Bright Data's entry point around $499/user/month, which puts it in a different budget class entirely.

The per-GB numbers look dramatically cheaper — $11.25 buys 3GB from Decodo, and 3GB of raw HTML is a lot of pages. But that comparison assumes you've already solved rendering, CAPTCHAs, retries, and per-site anti-bot quirks yourself, and that your targets don't need them. Each Google SERP request through ScraperAPI costs 25 credits, roughly $0.012 at Hobby's monthly rate; comparable SERP handling through a raw proxy pool means engineering time plus failed-request overhead you pay for in bandwidth. If your targets are static, unprotected pages at high volume, the per-GB vendors win on cost and we won't pretend otherwise. If they're protected or JavaScript-heavy, the credit model is buying you solved problems.

## Who should skip ScraperAPI

Some honest disqualification, because it makes the recommendation for everyone else more trustworthy:

- **High-volume scrapers of simple, unprotected sites.** If you're pulling millions of static pages monthly, per-GB residential proxies cost a fraction of the credit model, and you don't need the bypass machinery.
- **Non-US/EU geotargeting on a small budget.** Country-level targeting starts at Business ($299/month). Below that, you're locked to US and EU exits.
- **Anyone who needs predictable per-unit costs at scale.** Credit multipliers vary per domain and per parameter combination, which makes month-to-month cost harder to forecast than a flat $/GB rate. The Domain Cost Estimator helps, but it's still one more thing to manage.

## FAQ

**Does ScraperAPI charge for failed or blocked requests?**
Per the official documentation, you're only billed for requests that return HTTP 200 or 404, and for requests you cancel before the 70-second timeout. Failed bypasses that never produce a response don't cost credits. Note that a successfully bypassed protected page still costs its +10 bypass premium — that's a successful request, and reviewers who say "blocked requests cost money" are usually describing this case.

**Is there a free trial without a credit card?**
Yes. Seven days, 5,000 API credits, no card required. The permanent free plan (1,000 credits/month, 5 concurrent connections) also requires no card.

**Do unused credits roll over?**
The available documentation doesn't state a rollover policy; monthly plans refresh their allowance each billing cycle, and annual plans can be renewed early to get fresh credits. If rollover matters to you, confirm with support before buying.

**What's the difference between `premium` and `ultra_premium`?**
Both activate higher-quality proxy pools; `ultra_premium` costs 30 extra credits per request versus 10 for `premium`, is restricted to paid plans, and can be stacked with rendering at a combined 75 credits. Check the Domain Cost Estimator before using either — the price jump is significant.

## Picking a plan in 30 seconds

- **Just evaluating a residential proxy API for the first time?** Start with the free plan and the 7-day trial. Nothing to cancel, nothing to pay.
- **Standard HTML targets, under ~100K pages/month, US/EU only?** Hobby at $49/month is the right size, and annual billing brings it to $44.10/month.
- **Scaling past a million pages or need country-level geotargeting?** Business at $299/month is where both the credit volume and global targeting kick in; Startup won't cover the geo requirement.
- **Heavy Google/LinkedIn/protected-site workloads?** Do the multiplier math first — your effective page count is 25x to 75x lower than the headline credits, and a higher tier's better per-credit rate may matter less than simply targeting fewer protected pages.

One last pre-purchase check that saves the most money: run your top three target URLs through the Domain Cost Estimator, note the credit cost per request, and divide your plan's credits by that number. If the resulting page count covers your monthly volume with headroom, you're safe to subscribe. 👉 [See ScraperAPI's full plan list and start your free trial](https://www.scraperapi.com/pricing?fp_ref=coupons)
