# How to Scrape Reed.co.uk: UK Job Postings (Salary, Location, Employment Type)

> **Run it now, no code: [reed-jobs-scraper on the Apify Store](https://apify.com/bibian/reed-jobs-scraper)** &nbsp;|&nbsp; $5.00 per 1,000 job postings &nbsp;|&nbsp; no browser, no proxy, no login

Reed.co.uk is one of the UK's largest job boards. It has no public export and no official API for third parties. This guide shows how to get its postings as clean structured data: title, company, salary range, location, employment type and full description, with a free Apify Actor, from the console or from code.

Searches this page answers: reed.co.uk scraper, UK job postings scraper, UK salary data scraper, scrape reed jobs.

## What you get from the Reed.co.uk scraper

- Every job posting matching your keywords and location, with title, company name and full location (city, region, postal code).
- Salary range (`salaryMin`, `salaryMax`) with its currency and period (`year`, `day` or `hour`) kept explicit, not left for you to guess from the raw numbers.
- Employment type (full-time, part-time, contractor...), posting and expiry dates.
- Optional full description as plain text.

Export the dataset as JSON, CSV or Excel, or read it through the Apify API. A real sample (five results, personal data removed, long texts truncated) is in [`sample-output.json`](sample-output.json).

## Output fields

| Field | What it is |
|---|---|
| `id`, `url` | Reed posting id and public link |
| `title`, `companyName` | Job title and hiring company |
| `city`, `region`, `postalCode`, `country` | Where the job is based |
| `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod` | Salary range and how it is paid (`year`, `day` or `hour`), null when not published |
| `employmentType` | Array, for example `["FULL_TIME"]` or `["FULL_TIME", "CONTRACTOR"]` |
| `datePosted`, `validThrough` | ISO timestamps |
| `description` | Plain text, only when full descriptions are enabled |

## Use cases

- **Monitor the UK job market.** Track new postings by job title and location, with salary data when the employer discloses it.
- **Build a daily job alert.** Schedule a run every morning and dedupe by `id` to catch only the new postings.
- **Find the companies that are hiring.** List which employers recruit for a given role right now, in which city.
- **Benchmark salaries.** Group `salaryMin`/`salaryMax` by role, region or employment type to see what the UK market pays.

## Input

| Field | What it does |
|---|---|
| `keywords` | Job title or skill, for example `developer` or `nurse`. Empty browses every posting. |
| `location` | UK city or region, for example `Manchester` or `London`. Empty searches nationwide. |
| `maxItems` | The run stops once this many postings are saved (default 50). |
| `includeDescriptions` | Adds the full job description as plain text. |

Example input:

```json
{
  "keywords": "developer",
  "location": "Manchester",
  "maxItems": 100,
  "includeDescriptions": true
}
```

## Three ways to use it

### 1. Apify console, no code

1. Open [the Actor page](https://apify.com/bibian/reed-jobs-scraper) and click **Try for free** (a free Apify account is enough).
2. Fill in the input form (keywords, location and the maximum number of results) and click **Start**.
3. When the run finishes, open the **Output** tab, check the table, then **Export** as CSV, Excel or JSON.
4. For a daily alert, open **Schedules**, create a daily schedule for the Actor and, optionally, add an integration that sends new results to Slack, e-mail or Google Sheets.

### 2. Apify API with JavaScript

```bash
npm install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
node examples/run.js
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('bibian/reed-jobs-scraper').call({
    "keywords": "developer",
    "location": "Manchester",
    "maxItems": 100,
    "includeDescriptions": true
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
for (const item of items) {
    console.log(`${item.title} | ${item.companyName} | ${item.city} | ${item.salaryMin ?? '-'}-${item.salaryMax ?? '-'} ${item.salaryCurrency ?? ''}/${item.salaryPeriod ?? '-'}`);
}
```

The complete file is [`examples/run.js`](examples/run.js) (ES module, Node 18 or later).

### 3. Apify API with Python

```bash
pip install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
python examples/run.py
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("bibian/reed-jobs-scraper").call(run_input={
    "keywords": "developer",
    "location": "Manchester",
    "maxItems": 100,
    "includeDescriptions": True,
})

for item in client.dataset(run.default_dataset_id).iterate_items():
    print(f"{item['title']} | {item.get('companyName')} | {item.get('city')} | {item.get('salaryMin')}-{item.get('salaryMax')} {item.get('salaryCurrency')}/{item.get('salaryPeriod')}")
```

The complete file is [`examples/run.py`](examples/run.py). Never put your token in the code: read it from the `APIFY_TOKEN` environment variable as above.

## Pricing

Pay per result: **$5.00 per 1,000 job postings** (that is $0.005 per job posting), plus a tiny start fee. You pay only for the results saved to the dataset. Apify's free plan includes monthly credits, enough to try it on a few hundred results. Prices are those shown on the [Store page](https://apify.com/bibian/reed-jobs-scraper) when this guide was written; the Store page is the reference.

## Why this one

- **No login, no CV upload.** Everything comes from Reed's own public search and posting pages.
- **Half the leading Reed scraper's price**, at $5.00 per 1,000 postings.
- **Salary period kept explicit** (`year`, `day` or `hour`), not left for you to guess from the raw numbers.

## Limits and honest notes

- Reed's own `sortBy` parameter is off-limits to robots; this actor never uses it, so it reads results in the site's default order instead.
- `employmentType` can carry more than one value at once (for example `["FULL_TIME", "CONTRACTOR"]`): that is how Reed itself tags the posting, not a scraping artifact.
- Salary fields are empty when the employer does not publish a figure.
- The dataset contains job postings only. No candidate or recruiter personal data is collected; a line that carries a recruiter's e-mail address is dropped from the description.
- This is an unofficial tool, not affiliated with Reed.co.uk. Check the site terms and the data protection rules that apply to your use.

## FAQ

**Does it need a browser or a proxy?**  
No. The Actor reads the data directly, so runs are fast and cheap.

**How do I only get new postings every day?**  
Schedule the Actor once a day and dedupe the results by `id` in your own pipeline.

**Can I filter by location?**  
Yes, set `location` to any UK city or region, for example `"London"`.

**Can I get the data in Excel or Google Sheets?**  
Yes. Export the dataset as CSV, Excel or JSON from the Apify console, or connect the Apify Google Sheets integration.

## Try it

**[Open reed-jobs-scraper on the Apify Store](https://apify.com/bibian/reed-jobs-scraper)**

The code in `examples/` is released under the MIT license (see [LICENSE](LICENSE)). The data belongs to its original publishers.
