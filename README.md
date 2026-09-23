# rsRNADisease Analytics

rsRNADisease Analytics is the visitor analytics dashboard for the rsRNADisease database. It uses Cloudflare Pages Functions and Workers Analytics Engine to collect and query page views, visitors, visits, page paths, referrers, browsers, operating systems, and visitor regions.

This repository is adapted from [Han Analytics](https://github.com/uxiaohan/HanAnalytics). We sincerely thank the Han Analytics project for its open-source foundation and support.

## Architecture

```text
rsRNADisease page
  -> tracker.min.js
  -> /send Pages Function
  -> Workers Analytics Engine
  -> /api Pages Function
  -> rsRNADisease Analytics dashboard
```

`functions/send.js` writes browser events to the Analytics Engine binding. `functions/api.js` queries the Analytics Engine SQL API and serves dashboard data. The dashboard interface is implemented in `src/App.vue`.

## Cloudflare Pages configuration

Create a Cloudflare Pages project from this repository. Use the Vue build preset, or configure:

```text
Build command: npm run build
Build output directory: dist
```

Add these Pages environment variables. Keep the API token secret and never commit it to this repository.

```shell
# Cloudflare account ID
CLOUDFLARE_ACCOUNT_ID=your_cloudflare_account_id

# Token permitted to query the Analytics Engine SQL API
CLOUDFLARE_API_TOKEN=your_cloudflare_api_token

# Optional password for the analytics dashboard
CLOUDFLARE_WEBSITE_PWD=

# Optional tracking whitelist. Format: domain,site_id|domain,site_id
CLOUDFLARE_WEBSITE_WHITELIST=your-rsrnadisease-domain,rsRNADisease Analytics
```

In Pages **Settings -> Bindings**, add an **Analytics Engine** binding:

```text
Variable name: AnalyticsBinding
Dataset: AnalyticsDataset
```

To set the dashboard header's rsRNADisease link, add this optional Pages build variable:

```shell
VITE_RSRNA_DATABASE_URL=http://rsrnadisease.zhanglab-bioinfo.cn/
```

If it is unset, the header link defaults to `/`.

## Add the tracker to rsRNADisease

Add this script to the rsRNADisease pages, replacing the Pages domain. The `data-website-id` must match the whitelist site ID when a whitelist is enabled.

```html
<script
  defer
  src="https://your-analytics.pages.dev/tracker.min.js"
  data-website-id="rsRNADisease Analytics">
</script>
```

The tracker records a daily unique visitor and a visit per page path every 30 minutes using browser local storage. The Pages Function enriches events with browser, operating system, country code, referrer, and path before persisting them.

## Local development

```bash
npm install
npm run dev
npm run build
```

`npm run build` type-checks the project and produces the production bundle in `dist/`.

## Credits

- [Han Analytics](https://github.com/uxiaohan/HanAnalytics) for the original project and Cloudflare Analytics Engine implementation.
- [Cloudflare Pages](https://pages.cloudflare.com/) and [Workers Analytics Engine](https://developers.cloudflare.com/analytics/analytics-engine/) for hosting and analytics storage.
