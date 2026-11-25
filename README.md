# Amazon Offers Scraper: What the JSON Must Contain, What 500 ASINs Cost per Month, and Why Offer Counts Change With the ZIP Code

Three things decide whether an Amazon offers scraper earns its bill: the unit you're charged in, the delivery ZIP the offers were pulled for, and whether the Buy Box copy got counted twice. Get any of them wrong and the JSON still parses, the dashboard still looks healthy, and the prices underneath are wrong.

ScraperAPI covers the first two with a structured endpoint, `/structured/amazon/offers`, which takes one ASIN per request and returns JSON or CSV. Amazon calls are billed at **5 credits each**, and a 7-day trial hands you 5,000 credits (about 1,000 ASIN pulls) without a card.

> **Most readers need Hobby.** 100,000 credits a month is 20,000 Amazon offers requests, and a daily sweep of 500 ASINs burns 75,000 credits. That leaves room for reruns and a second marketplace. If you only want to see the response shape before spending anything, the trial covers roughly 200 ASINs a day for a week.
>
> 👉 [See the current ScraperAPI plans and credit allowances](https://www.scraperapi.com/pricing?fp_ref=coupons)

## All ScraperAPI plans, converted into Amazon requests

Credits are the only number the pricing page leads with, and it isn't the number you'll plan with. Divide by five and you get requests you can actually make.

| Plan | API credits / month | Amazon offers requests / month | Concurrent threads | Fits | Get started |
| --- | --- | --- | --- | --- | --- |
| Free | 1,000 | 200 | 5 | Checking the JSON shape | [ Free plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | 100,000 | 20,000 | 20 | A few hundred ASINs, once or twice a day | [ Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 | 200,000 | 50 | Agency work across several marketplaces | [ Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | 600,000 | 100 | Country-level geotargeting starts here | [ Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 | 1,000,000 | 200 | Hourly repricing on a large catalog | [ Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 | 2,100,000 | 300 | Multi-tenant repricer backends | [ Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 | 4,300,000 | 500 | Continuous sweeps, many marketplaces at once | [ Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 4,400,000+ | 500+ | Volume beyond the listed tiers | [ Enterprise](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Two specs in that table don't fit: geotargeting is US and EU only on Hobby and Startup, and becomes country-level from Business upward. Analytics history is 30 days on the two cheapest paid tiers and unlimited from Business.

Monthly and annual prices for each tier sit on the pricing page and move with promotions, so the table sticks to the specs that don't.

## What you send to the endpoint

The request is a single GET with the ASIN as the required parameter. Everything else narrows or localises the result.

| Parameter | What it does | Watch out for |
| --- | --- | --- |
| `asin` | The product to pull offers for | ASINs are marketplace-specific; one copied from amazon.com won't resolve on amazon.co.uk |
| `tld` | Picks the marketplace: com, co.uk, ca, de, es, fr, it, co.jp, in, cn, com.sg, com.mx, ae, com.br, nl, com.au, com.tr, sa, se, pl | 20 in total, so the same code covers most of your expansion list |
| `country_code` | Two-letter code for the country you're scraping from | Needed alongside `tld` when you want amazon.com pricing as seen from Canada, for example |
| `output_format` | `json` (default) or `csv` | CSV is fine for a warehouse load, JSON if you need the nested product block |
| `F_NEW`, `F_USED_LIKE_NEW`, `F_USED_VERY_GOOD`, `F_USED_GOOD`, `F_USED_ACCEPTABLE` | Boolean condition filters | The used filters are how you see the resale side of a listing without new offers drowning it |

ZIP code targeting is documented separately and changes what comes back. That's the next section, because it's where most of the bad data originates.

## What the response actually gives you, and what it doesn't

The documented response has an `item` block (name, brand, product information, average rating, review count, link to the offers page) and a `listings` array. Each listing carries six fields. Whether that's enough depends entirely on what you're building.

| Field | What it's for | In the documented response |
| --- | --- | --- |
| `price` / `price_with_currency` | The comparison number itself | Yes |
| `shipping_price` | Landed cost; a $2 cheaper offer with $30 shipping isn't cheaper | Yes |
| `has_prime_shipping` | Prime badge filtering | Yes |
| `ships_from` | Spotting cross-border sellers | Yes |
| `sold_by` | Competitor identity | Yes |
| `fullfilled_by_amazon` | FBA versus FBM (note the double "l" in the field name if you're mapping columns) | Yes |
| Buy Box winner flag | Who actually wins the sale | Not a separate field in the sample |
| Seller feedback percentage and rating count | Whether a cheaper seller is real competition or a dormant account | Not in the sample |
| Condition label per offer | Used/new mix in one payload | Filters exist on input; the sample doesn't show a per-listing condition field |
| Quantity limits, coupons, delivery countdown | Promo and cart-cap signals | Not in the sample |

For straight price-and-availability monitoring, those six fields do the job. For a repricing engine that has to weigh seller quality, or a Buy Box strategy tool that needs to know who holds the box rather than who's cheapest, the fields above the line aren't enough on their own. Either verify against a live response before you commit, or plan a second data source for seller metrics.

That gap is worth naming because the rest of the market prices around it. Easyparser's Amazon ASIN Offers Scraper returns `buybox_winner_name`, `seller_positive_percent`, `seller_ratings_total`, `min_quantity`, `max_quantity` and a `has_coupon` flag per offer, and bills **1 credit per page of roughly 10 offers**. ScrapeHero's offer listing scraper returns 18 fields including `seller_percentage` and `seller_condition_notes` at **2 data credits per record**. Neither is automatically better; they're answering a heavier question for a heavier price.

## The billing unit is where the real cost hides

Four tools in this space use four different units, and the headline numbers can't be compared directly:

| Billing unit | Example | What one unit buys | 500 ASINs at ~8 offers each |
| --- | --- | --- | --- |
| Per request | ScraperAPI, 5 credits per Amazon request | One ASIN's full offer list, whatever its length | 2,500 credits |
| Per 1,000 results | Oxylabs, from $0.50 per 1,000 results | 1,000 offer records | about $2.00 |
| Per page | Easyparser, 1 credit per page of ~10 offers | Roughly 10 offers | 500 credits |
| Per record | ScrapeHero, 2 data credits per record | One offer record | 8,000 data credits |

Run the per-request column forward. A daily sweep of 500 ASINs is 2,500 credits a day, or 75,000 for a 30-day month. Hobby's 100,000 absorbs it. Move to hourly and you're at 1.8 million credits a month, which lands on Business.

Now run the per-result column the same way. 4,000 offer records per sweep, 120,000 a month, at the published starting rate of $0.50 per 1,000 results, comes to roughly $60 a month. That's not a verdict on which is cheaper, because Oxylabs' rate is a floor and its page parameter bills each page as a separate fetch and charge. It's a demonstration that the two units invert the economics:

- **Per request** flattens the tail. A long-tail ASIN with one seller costs exactly what a 30-seller ASIN costs.
- **Per record** does the opposite. Long-tail catalogs get cheap, and the heavily contested ASINs that matter most get expensive.
- **Per page** sits between the two and rounds up, since a page returns roughly ten offers whether the ASIN has eleven or three.

Pick the unit that matches your catalog's shape. A repricer tracking a few hundred crowded electronics listings is better served by result-based billing. A brand monitoring 5,000 of its own ASINs, most with two or three sellers, is better served by per-request billing, where the empty and near-empty listings cost the same as everything else.

One more trap: "1,000 credits" is not a unit of measurement across vendors. On ScraperAPI's free plan, 1,000 credits is 200 Amazon requests. Read any competitor's credit allowance as a raw number only after you've found its per-request multiplier.

👉 [Check which ScraperAPI tier your sweep rate actually lands in](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Why your own script stops after a few calls

Amazon sits behind AWS WAF. Every request gets scored on its IP, its headers and recent activity, and a `requests` call or a headless browser on a datacenter IP returns a CAPTCHA page or a block within a handful of attempts. Getting past that means maintaining residential proxies, browser-like headers and fingerprints, then re-testing all of it every time the scoring changes. That upkeep is the product a managed scraper is selling.

There's a second reason to use one, and it's the one people miss: a plain request often returns a page without a product title, which is what a soft block looks like. Parsers that assume success then write garbage into your database, or throw an exception deep in a nightly job at 3 a.m.

## The ZIP code decides how many offers exist

Offer counts are not a property of the ASIN. They're a property of the ASIN plus the delivery address, because Amazon shows different sellers and different shipping prices depending on where the item is going. Scraping the same listing from two ZIPs can return different seller sets and different prices for the same seller. One offer, or zero, is a perfectly valid result.

If you're comparing your price against "the market," you first have to decide which market. ScraperAPI handles this with ZIP code targeting; Oxylabs takes a delivery postal code through its `location` parameter and will fail the request outright rather than return content that ignored the location. Both exist because the underlying page genuinely changes.

This is also where plan tier bites. If your offers need to reflect a delivery address outside the US or EU, Hobby and Startup's geotargeting won't do it. Country-level targeting starts at Business.

## Two timing traps to build around

**Buy Box duplication.** Amazon's "Other sellers" panel is served from an AJAX endpoint, and it contains a pinned copy of the Buy Box offer inside its own container. A naive parse counts that offer twice, so a listing with seven real sellers reports eight. Skip the pinned block and you get the true count. Any tool that returns a clean `offer_count` is doing this deduplication for you; check that it does.

**Reviews are gone from the product page.** Since May 2026, public review pages return Page Not Found to logged-out clients, and review text was removed from the product page HTML entirely. Any 2024-era pipeline that scraped the PDP and picked up reviews along the way is now silently returning empty arrays. Reviews need a dedicated endpoint that serves featured reviews; the offers endpoint won't produce them.

## Compliance, briefly

Offers, prices and seller names are publicly visible page data, and scraping them is generally treated as permissible depending on your jurisdiction and how you use the output. Amazon's Conditions of Use restrict automated access, and anything behind a login is off limits. Keep request rates sane, skip customer personal data, and take legal advice if you're reselling the data rather than acting on it.

## FAQ

**How many offers does a typical ASIN have?**
Anywhere from zero to thirty-plus, and the number shifts with the delivery ZIP and your condition filters. Size your batch by ASIN count, not by expected offer count, unless you're on result-based billing.

**Can I request only Prime or only FBA offers?**
Condition filters are exposed as input parameters. Prime and FBA are returned per listing as `has_prime_shipping` and `fullfilled_by_amazon`, so filter after the response rather than before the call.

**Is 1,000 free credits enough to evaluate this?**
It's 200 Amazon requests, which is enough to run 200 ASINs once and inspect the JSON. The 7-day trial's 5,000 credits gets you about 1,000 ASIN pulls, which is enough to run a real sample of your catalog and see whether the fields hold up.

**Can I pass a product URL instead of an ASIN?**
The offers endpoint takes an ASIN. Extract it from the URL, and make sure you're using the ASIN for the marketplace you set in `tld`.

**Can I pull reviews from the same call?**
No. The offers response covers listings, not review text, and public review pages have been closed to logged-out clients since May 2026.

## How to choose

- **One-off research on a few dozen ASINs:** free plan, 200 requests, no card.
- **Daily sweeps of 500 ASINs, US or EU delivery addresses:** Hobby at 100,000 credits.
- **Hourly sweeps or a catalog in the thousands:** Business or Scaling for the thread count and country-level targeting.
- **Buy Box ownership history as the core product:** confirm the response carries a winner flag before you pay, because the documented sample doesn't label one.

Two things to settle before you enter card details. First, whether your delivery addresses fall outside the US and EU, because that decision alone moves you up two tiers. Second, whether your repricing logic needs seller feedback percentages and per-offer condition labels, which the offers response doesn't carry, and where a second source is cheaper than a bigger plan.

👉 [Compare ScraperAPI plans and start the 5,000-credit trial](https://www.scraperapi.com/pricing?fp_ref=coupons)
