# European Central Bank Exchange Rate API — ecb-exchange-rate

[![npm version](https://img.shields.io/npm/v/ecb-exchange-rate.svg)](https://www.npmjs.com/package/ecb-exchange-rate)
[![license](https://img.shields.io/npm/l/ecb-exchange-rate.svg)](https://github.com/AllRates-Today/ecb-exchange-rate/blob/main/LICENSE)
[![zero dependencies](https://img.shields.io/badge/dependencies-0-brightgreen.svg)](https://www.npmjs.com/package/ecb-exchange-rate)
[![TypeScript](https://img.shields.io/badge/TypeScript-types%20included-3178C6.svg)](https://www.typescriptlang.org/)

**Official European Central Bank (the Eurozone) daily exchange rates for Node.js and TypeScript. The published central bank rates behind tax filings, customs valuations, audits, and compliant invoicing — not market estimates, but the numbers European Central Bank itself prints, every business day.**

## 🚀 Why this client?

- 🏛️ **Official published rates** — European Central Bank's own table, with the publisher's own `rate_date` on every response
- 📅 **History back to 1999** — point-in-time tables and daily series for any past date
- 🔀 **Published vs derived, always flagged** — computed inverse/cross pairs carry `derived: true`, never mixed with official prints
- ⚡ **Zero dependencies** — pure ESM + CJS over global `fetch`; Node 18+, Bun, Deno, and edge runtimes
- 🔷 **Type-safe** — full TypeScript definitions shipped with the package
- 🧾 **Compliance-grade metadata** — `rate_type`, publication date, and source disclaimer on every response

## 🔑 Get your API key

Get a free API key at [allratestoday.com/register](https://allratestoday.com/register) — no credit card required. Latest rates are on every plan, including free.

## 📦 Installation

```bash
npm install ecb-exchange-rate
```

```bash
yarn add ecb-exchange-rate
```

```bash
pnpm add ecb-exchange-rate
```

Also published under the org scope as [`@allratestoday/ecb-exchange-rate`](https://www.npmjs.com/package/@allratestoday/ecb-exchange-rate) — same code, same versions.

## 🏁 Quick start

```js
import { getRate } from 'ecb-exchange-rate';

const pair = await getRate('USD', 'EUR', { apiKey: 'art_live_...' });
console.log(pair.rate, pair.rate_date); // the official European Central Bank rate, on the central bank's own date
```

## 📚 API reference

- [Latest pair rate](#latest-pair-rate) — one pair from the latest published table
- [Full published table](#full-published-table) — everything the central bank printed, in one call
- [Table for a date](#table-for-a-date) — the official table for an invoice or filing date
- [Daily time series](#daily-time-series) — one pair across a date range

---

### Latest pair rate

Free plan and up. Pairs the central bank does not print directly are resolved from its table and flagged (see *Published vs derived rates* below).

```js
const pair = await getRate('USD', 'EUR', { apiKey: 'art_live_...' });
```

**Response:**

```javascript
{
  bank: 'ecb',
  name: 'European Central Bank',
  rate_date: '2026-08-11',   // European Central Bank's own publication date
  source: 'USD',
  target: 'EUR',
  rate: 0.866551,
  rate_type: 'reference',
  derived: true,
  method: 'inverse',  // computed from the published EUR/USD print
  disclaimer: '…'
}
```

### Full published table

Free plan and up. The complete table for the latest publication date.

```js
import { getLatestRates } from 'ecb-exchange-rate';

const table = await getLatestRates({ apiKey: 'art_live_...' });
console.log(table.rate_date, table.rates.length);
```

**Response:**

```javascript
{
  bank: 'ecb',
  name: 'European Central Bank',
  rate_date: '2026-08-11',
  rates: [
    { "base": "EUR", "quote": "USD", "type": "reference", "value": 1.154 },
    // … the rest of the published table (29 currencies vs EUR)
  ],
  disclaimer: '…'
}
```

### Table for a date

Paid plans. The official table for any date since 1999 — weekends and holidays return the most recent published date, flagged via `published_on_requested_date`, which is exactly the in-force rate a filing needs.

```js
import { getRatesForDate } from 'ecb-exchange-rate';

const day = await getRatesForDate('2026-06-30', { apiKey: 'art_live_...' });
// Optionally narrow to one pair:
const one = await getRatesForDate('2026-06-30', { apiKey: 'art_live_...', source: 'USD', target: 'EUR' });
```

**Response:**

```javascript
{
  bank: 'ecb',
  requested_date: '2026-06-30',
  rate_date: '2026-06-30',                // the date actually published
  published_on_requested_date: true,      // false when a weekend/holiday fell back
  rates: [ /* the full table for that date */ ],
  disclaimer: '…'
}
```

### Daily time series

Paid plans. One resolved rate per publication date — ready for charting, revaluation runs, or audit workpapers.

```js
import { getHistory } from 'ecb-exchange-rate';

const series = await getHistory(
  { source: 'USD', target: 'EUR', from: '2026-01-01', to: '2026-08-11' },
  { apiKey: 'art_live_...' }
);
```

**Response:**

```javascript
{
  bank: 'ecb',
  source: 'USD',
  target: 'EUR',
  from: '2026-01-01',
  to: '2026-08-11',
  count: 152,
  rates: [
    // one entry per publication date
    { date: '2026-08-11', rate: 0.866551, rate_type: 'reference', derived: true, method: 'inverse' },
    // …
  ],
  disclaimer: '…'
}
```

Pass `{ symbol: 'USD' }` instead of `source`/`target` to get the raw published rows for one currency (all rate types, no pair resolution).

---

## 🗺️ Currencies covered

European Central Bank currently publishes rates covering **30 currencies** (as of the latest table):

`AUD` · `BRL` · `CAD` · `CHF` · `CNY` · `CZK` · `DKK` · `EUR` · `GBP` · `HKD` · `HUF` · `IDR` · `ILS` · `INR` · `ISK` · `JPY` · `KRW` · `MXN` · `MYR` · `NOK` · `NZD` · `PHP` · `PLN` · `RON` · `SEK` · `SGD` · `THB` · `TRY` · `USD` · `ZAR`

## ⚖️ Published vs derived rates

If European Central Bank does not print a pair directly, the API resolves it from the central bank's own table and says so — official and computed values are never confused:

| `method` | `derived` | Meaning |
| --- | --- | --- |
| `published` | `false` | The central bank printed this pair directly |
| `inverse` | `true` | Computed as 1 ÷ the published opposite direction |
| `cross` | `true` | Computed via EUR from two published rates |

## 🛡️ Error handling

Errors are thrown as `Error` with `status` (HTTP code) and `body` (the API's JSON error) attached:

```js
try {
  const pair = await getRate('USD', 'XXX', { apiKey: 'art_live_...' });
} catch (err) {
  console.log(err.message); // human-readable reason
  console.log(err.status);  // e.g. 404
}
```

| Status | Meaning |
| ------ | ------- |
| — | Missing `apiKey` (thrown before any request) |
| `400` | Malformed date or parameters |
| `401` | Invalid API key |
| `403` | Endpoint needs a [paid plan](https://allratestoday.com/pricing/) (historical dates & series) |
| `404` | Pair or date range not covered by European Central Bank |
| `429` | Monthly quota exceeded |

## 🔷 TypeScript

Full definitions ship with the package — no `@types` install:

```ts
import type { LatestRates, PairRate, DatedRates, RateEntry, HistoryQuery, RequestOptions } from 'ecb-exchange-rate';
```

## 📦 CommonJS

```javascript
const { getRate } = require('ecb-exchange-rate');

getRate('USD', 'EUR', { apiKey: 'art_live_...' }).then((pair) => console.log(pair.rate));
```

## 💡 Quota tips

- Rates change once per business day — cache the published table locally and a small monthly quota goes a long way.
- Every request counts toward your AllRatesToday quota, shared across all AllRatesToday endpoints on your key.

## 📖 Methods reference

| Method | Plan | Description |
| ------ | ---- | ----------- |
| `getRate(source, target, { apiKey })` | Free | Latest rate for one pair, resolved from the published table |
| `getLatestRates({ apiKey })` | Free | The central bank's full latest published table |
| `getRatesForDate(date, { apiKey, source?, target? })` | Paid | The official table (or one pair) for a YYYY-MM-DD date |
| `getHistory({ symbol \| source+target, from?, to? }, { apiKey })` | Paid | Daily series since 1999 |

## 🔗 Links

- [European Central Bank rates page](https://allratestoday.com/central-bank-rates-api/ecb/) — live table, publication cadence, FAQ
- [All central bank sources](https://allratestoday.com/central-bank-rates-api/)
- [API documentation](https://allratestoday.com/docs/#central-bank) · [Interactive reference](https://allratestoday.com/api-reference/)
- [Register (free)](https://allratestoday.com/register) · [Pricing](https://allratestoday.com/pricing/)
- [GitHub](https://github.com/AllRates-Today/ecb-exchange-rate)

## 📜 License

MIT
