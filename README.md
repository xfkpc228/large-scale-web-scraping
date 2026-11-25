# Large Scale Web Scraping: What Breaks First at 100K Pages a Day, How to Get Past Pagination Caps, and When Buying Beats Building (With Full Plan and Credit Tables)

Ten million pages a day is roughly where teams stop running their own scrapers and hand the job to a managed service. Between 100,000 and a million pages a day, a small worker fleet behind a real queue is enough. Below 100,000, one node and a rotating proxy list still work.

That rule of thumb holds because each tier breaks a different part of the system. What follows is the arithmetic behind it: the three ways a working script fails, how to get around a pagination cap, which proxy belongs on which URL, what changed in anti-bot defenses, and what each route costs per thousand pages.

> **If you only want the plan:** Startup at **$149/month** gives 1,000,000 credits and 50 concurrent threads, which covers roughly 200,000 Amazon pages or 40,000 fully rendered, bot-protected ones. Hobby at **$49/month** (100,000 credits, 20 threads) handles plain HTML at small volume. Both are billed monthly in USD, annual billing takes 10% off, and new accounts get a 7-day trial with 5,000 credits and no card.
> 👉 [Check the current Startup plan price](https://www.scraperapi.com/pricing?fp_ref=coupons)

## All seven ScraperAPI plans, and what each one actually buys

ScraperAPI bills in credits, and a credit maps to a request type rather than to a page. Every paid plan runs the same product; what changes is the credit allowance, the concurrency ceiling, and how precisely you can target a country. Prices for the first four tiers are published on the plan page. Professional, Advanced and Enterprise are quoted by sales.

| Plan | Credits/month · concurrent threads | Price/month | Best for |  |
| --- | --- | --- | --- | --- |
| Free | 1,000 · 5 | $0 | Testing the endpoint before you commit | [Start free](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | 100,000 · 20 | $49 | Plain HTML at low volume; US and EU targeting only | [See Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 · 50 | $149 | The first build that has to run every day | [See Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 · 100 | $299 | Country-level targeting, e-commerce catalogs | [See Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 · 200 | $475 | Continuous runs that need pay-as-you-go overage | [See Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 · 300 | Custom quote | Multi-domain programs with heavy rendering | [Request pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 · 500 | Custom quote | Catalog-scale monitoring across regions | [Request pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ · 500+ | Custom quote | Replacing an in-house fleet | [Request pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Three limits sit outside the credit count and they decide more purchases than price does:

- **Concurrency is a hard ceiling.** Go over it and the API returns a 429 and turns the request away. You are not charged for it, and it is not queued.
- **Country-level geotargeting outside the US and EU starts at Business ($299).** Hobby and Startup target US and EU only. If your project is a price monitor for a retailer in Brazil or Japan, the entry plan is Business.
- **Credits reset at renewal and do not roll over.** Annual plans can renew early to get a fresh batch, and unspent credits at the end of a cycle are simply gone.

Analytics history follows the same split: 30 days on Hobby and Startup, unlimited from Business up.

## What one credit buys, and why budgets go wrong here

The rate card charges by target difficulty and by flag, which is why two projects on the same plan can burn credits at wildly different rates.

| Request type | Credits |
| --- | --- |
| Standard page | 1 |
| Amazon, Walmart, eBay | 5 |
| Google, Bing (incl. subdomains) | 25 |
| LinkedIn | 30 |
| Bot-protection bypass (Cloudflare, Turnstile, DataDome, HUMAN, Akamai) | 10 |
| JS rendering (`render=true`) | 10 |
| Premium proxies (`premium=true`) | 10 |
| Screenshot | 10 |
| Premium proxies + JS rendering | 25 |
| Ultra premium (`ultra_premium=true`) | 30 |
| Ultra premium + JS rendering | 75 |
| Geotargeting, sessions, device type, output format, auto-parsing | free |

Flags add up, and the card prices the common combinations directly. An Amazon page that also needs rendering and a Cloudflare bypass carries the 5-credit base, the 10-credit bypass and the 10-credit render; premium proxies with rendering are priced at 25 on their own. For hard targets, budget 25 credits per request rather than 5.

Translated into money at the published tiers:

| Plan | Per 1,000 one-credit pages | Per 1,000 Amazon pages (5 credits) | Per 1,000 hard targets (25 credits) |
| --- | --- | --- | --- |
| Hobby ($49 / 100K) | $0.49 | $2.45 | $12.25 |
| Startup ($149 / 1M) | $0.15 | $0.75 | $3.73 |
| Business ($299 / 3M) | $0.10 | $0.50 | $2.49 |
| Scaling ($475 / 5M) | $0.095 | $0.48 | $2.38 |

Every response reports its own cost in the `sa-credit-cost` header, and the dashboard has a Domain Cost Estimator you can run against a URL before committing. Do that on ten real targets from your list before picking a plan, because the difference between a 1-credit page and a 25-credit page is a factor of 25 on the same invoice.

👉 [Start the 7-day trial with 5,000 credits and price your own URLs](https://www.scraperapi.com/?fp_ref=coupons)

## Why the script that handled 20,000 pages dies later

A normal scraping workflow is fine for a normal site. Take a resale catalog with 21,487 bags across 359 paginated pages, where a brand like Louis Vuitton sits in the $1,000 to $2,000 band. You enumerate the pages, fetch each detail page, run it on one server, and it finishes.

Three things break when the target gets bigger:

1. **The pagination cap.** Large e-commerce sites stop serving results after roughly 1,000 to 10,000 products per query. Page 400 does not exist, so no amount of crawling reaches the rest of the catalog.
2. **One server stops being enough.** Vertical scaling has a ceiling, and you hit it before the site does.
3. **The default proxy list does not scale.** A handful of datacenter IPs is fine for 20,000 requests and gets blocked long before a million.

The pagination cap has a fix that costs nothing but a little recursion. Slice the catalog with search filters and subcategories first, then split by price band ($0–10, $10–100, $100+), then halve each band again ($0–5, $5–10) until every slice returns fewer results than the cap. A 40,000-product category with a 5,000-item cap needs three levels of splitting at most, and each slice is an ordinary crawl.

## The six layers of a system that keeps running

A production scraper ends up looking like any other distributed service. These are the parts that cannot be skipped, with the selection call for each.

**1. A job queue.** Most teams reach for Redis because it is already in the stack, and it works at low volume. Redis was built for caching, and its limited persistence, missing native message replay and weak consumer-group management surface once volume climbs. RabbitMQ gives reliable routing, built-in acknowledgements and retry logic without the operational weight of a streaming platform. Kafka is the choice when you need to replay a URL stream after a bad deploy or survive a broker failure without losing work. Pick by whether replay matters to you.

**2. Workers.** Stateless and restartable, which makes them a natural fit for spot instances on AWS and preemptible VMs on GCP, where compute runs up to 90% below on-demand pricing. Teams that design for interruption from day one get the cheapest tier on every cloud.

**3. The fetch layer.** Two kinds of client, used together. Scrapy and HTTPX are fast and cheap for server-rendered HTML. Playwright and Puppeteer execute JavaScript and simulate interaction, cost far more CPU per page, and are unavoidable for single-page applications. Most teams running at volume operate both and route by target.

**4. A deduplication layer.** Distributed workers will hand the same URL to more than one node, especially after a retry. Without an explicit dedup step you get duplicate records, conflicting versions of the same product, and a dataset nobody trusts.

**5. The proxy layer,** covered below.

**6. Parsing, validation and monitoring.** Parsers break when a site ships a redesign, so build selectors against attributes, semantic tags and content structure rather than class names and DOM depth. Version your scraper configs like code so a bad update can be rolled back in minutes. And monitor the output, not the uptime: a scraper can run successfully and return garbage.

Four metrics catch almost everything: extraction success rate, data freshness, field-level null rate (if price comes back null on 30% of pages, the site changed), and queue depth against worker throughput. A 2% failure rate across a million requests is 20,000 missing records, and something downstream has to absorb that.

## Proxies: route by URL pattern, not by preference

Datacenter proxies get blocked at volume. Residential proxies are expensive. The working answer is to use both, plus third-party providers behind a single endpoint, and to route by the kind of page being fetched.

Category and listing pages go through datacenter IPs because they are cheap and usually lightly defended. Product detail pages go through residential IPs. When the CAPTCHA rate on a given URL class climbs, the orchestrator reclassifies that class to a higher proxy tier automatically. With a few dozen domains you can maintain that mapping by hand. Past a hundred, the routing has to learn from its own success and failure metrics per URL pattern, because no one is going to keep a spreadsheet current.

Wrapping several providers behind one endpoint also buys reliability: when one pool degrades, traffic shifts to the others instead of the pipeline stalling.

## Three things that got measurably harder

**Turnstile replaced the image grid.** Cloudflare Turnstile has been displacing traditional CAPTCHAs, and it usually runs invisibly in the background using fingerprinting plus a cryptographic proof-of-work. There is no puzzle to solve, so the older "send it to a solving service" step no longer applies.

**The detection surface moved to the protocol and to behavior.** The Chrome DevTools Protocol is now a primary detection vector. Patchright and rebrowser-patches fix specific CDP leaks such as the `Runtime.enable` command; Pydoll and Selenium Driverless drop the WebDriver layer and talk to Chrome over CDP directly; Nodriver minimizes CDP usage to avoid the patterns anti-bot systems flag; BotBrowser ships a browser with modified internals. Patching fingerprints is only the entry ticket. Systems like DataDome and HUMAN Security score the whole session, and a fingerprint-perfect browser with no behavioral simulation still gets caught, because real users move the mouse along curves and pause while scrolling.

**Fake pages now return 200 OK.** Cloudflare's AI Labyrinth, live since March 2025 and available on free-tier plans, does not block scrapers. It serves AI-generated pages with plausible content. Your request succeeds, the HTML is valid, the fields parse, and the data is synthetic. This changes what "success" means for a pipeline: checking for a block page is no longer enough, and the extraction layer has to verify that what it captured is real. For context on the scale of the problem, automated traffic has passed 51% of all web traffic, and AI crawlers alone generate more than 50 billion requests to Cloudflare's network every day.

A site does not have to stop you. It only has to make scraping expensive enough to be uneconomical, and serving fake data at zero cost to itself does exactly that.

## The cost line that decides build versus buy

Extraction is where the per-page economics changed. Across a 3,000-page study, AI-based extraction hit 98.4% to 100% accuracy and held up when page structures changed, while rigid CSS selectors broke on every redesign. That resilience is not free: hosted LLM extraction tools run **$666 to $3,025 per million pages**, against **$1 to $5 per million pages** for deterministic parsers.

The two numbers have different denominators, so do not blend them into a single "cost per page." The practical rule is a hybrid: price, stock and rating fields go through deterministic code because they must be exact and they change often; descriptions, categories and attributes go through AI extraction because the layout around them shifts and a slightly imperfect string is tolerable. Paying the AI tier for a price field is how a $1,000 monthly bill becomes $3,000.

Then add the fetch cost on top, because fetching and extraction are separate line items. At Startup, a million one-credit pages cost $149 to fetch; at 25 credits apiece, the same million pages cost $3,730 before any extraction happens. Compute discounts of up to 90% on spot instances apply to the self-hosted route, not to a managed API, which is why the build-versus-buy comparison has to be run per project rather than in the abstract.

👉 [Compare per-credit pricing across all plans](https://www.scraperapi.com/pricing?fp_ref=coupons)

## When to stop building

Ten million pages a day is the point where managed infrastructure usually wins, and the reason is operational rather than technical. Batch APIs in this category sell job orchestration: you submit a URL list as one job and poll it, get a signed webhook with HMAC verification when the run completes, rerun only the failed tasks by status filter, upload input as CSV through a presigned URL, schedule recurring runs, and attach a credit-ceiling monitor that stops a runaway job. Reference implementations of that workflow default to 100,000 tasks per job and can take hours to reach a terminal state.

None of that is hard to build. All of it is work someone has to own at 2 a.m., and it is the part that quietly consumes a data team's quarter.

The public plans also show where the ceiling sits. Enterprise at 22,000,000+ credits a month is about 730,000 standard pages a day. If your target is genuinely 10 million pages a day, you are on a custom quote, and the honest comparison is between that quote and two full-time engineers plus a proxy bill.

## Before you order

Two things are worth confirming before you pick a tier. First, run your actual URLs through the Domain Cost Estimator, because credits are charged per domain and flag, not per page count; ten sample URLs will tell you more than any per-page estimate. Second, know your overage path: credits do not roll over, pay-as-you-go overage exists only from Scaling upward, and exceeding concurrency returns a 429 rather than an extra charge, so a plan sized too tightly will simply stop mid-run.

The three-tier rule, then: under 100,000 pages a day with mostly plain HTML, Hobby at $49 is enough. Between 100,000 and a million pages a day, or any volume of rendered and protected targets, Startup at $149 is the sane starting point, and Business at $299 is the jump you make when you need country-level targeting or your credit burn crosses two million. Above that, Scaling and the quoted tiers exist for a reason, and the trial is how you find out which one you actually are.

👉 [Start with Startup at $149/month and 1,000,000 credits](https://www.scraperapi.com/pricing?fp_ref=coupons)
