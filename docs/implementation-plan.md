# 3D Print Pricing Calculator — Initial Implementation Plan

Status: planning only; no runnable app or deployment yet.
Working product name: Print Pricing. MakerBridge-style expansion is a direction, not a cleared business name.

## Smallest useful release

Build a phone-friendly single-page calculator for a small paid-print workflow. Enter filament grams (including supports and purge), spool cost and spool grams, print hours and minutes, optional machine cost per hour, extra costs, either percentage markup or a fixed profit amount, and customer discount. Show material cost, estimated total cost, regular asking price, discounted price and estimated profit after discount.

Default currency: USD. Default spool weight: 1000 g. Machine hourly cost starts at zero and is clearly editable; it can represent electricity and wear. Extra costs cover attachments, packaging and labor. Results are estimates before tax and transaction fees unless those are included in extras. Profit means price minus entered costs, so omitted labor or overhead overstates it.

Use a simple React + TypeScript + Vite app with plain CSS, Vitest for pricing tests and Playwright for a small browser smoke test. Select supported stable versions and commit the lockfile during scaffolding. One app and one pricing module; no backend, accounts, database or external services in release one.

## Phone workflow

1. Enter print usage and spool details using labeled numeric inputs with units.
2. Expand optional costs; select markup percentage OR fixed desired profit.
3. Adjust discount, with a quick 20% family discount option.
4. Read a prominent results summary and copy a plain-text quote.
5. Remember only cost/settings defaults on this device, with a reset button.

Use a single column at 360 px, large tap targets, decimal-friendly keyboards, visible validation and no horizontal scrolling. Keep results accessible when the keyboard is dismissed. Clipboard failure must offer selectable text. Blank required inputs are incomplete, not zero; optional fields may default to zero. No customer names or saved quote history in this release.

## Pricing contract

Let g = filament used, s = spool cost, w = spool grams, h = hours + minutes/60, r = machine hourly cost, e = extra costs.

- Material cost = g / w * s.
- Estimated cost C = material cost + h * r + e.
- Markup mode: regular price P = C * (1 + markup/100).
- Fixed profit mode: P = C + desired profit.
- Discounted price D = P * (1 - discount/100).
- Estimated profit after discount = D - C.

Markup is a percentage of cost, not profit margin. Fixed desired profit is BEFORE discount; show the reduced profit after discount explicitly. Apply discount to the asking price, never to the underlying cost. Warn visibly when profit is negative.

Accept finite nonnegative costs, grams, time and profit; spool grams must exceed zero. Discount is 0–100%; minutes are 0–59; markup must be nonnegative. Reject NaN, infinity and malformed entries. Use decimal arithmetic (for example decimal.js) from input strings to avoid binary money errors. Round regular asking price to cents first, discount that amount and round to cents, then subtract unrounded estimated cost and round profit to cents. Round intermediate cost displays only; document this policy in tests.

Reference case: 100 g, $20/1000 g spool, 2 hours at $1/hour, $1 extras => $2 material, $5 cost. 100% markup => $10 regular, 20% discount => $8 discounted, $3 profit. Fixed profit $5 produces the same result. 100% discount produces $0 asking price and -$5 profit.

## First files to implement

| File | Responsibility |
| --- | --- |
| package.json and package-lock.json | Dependencies; dev, build, typecheck, test and browser-test commands |
| index.html, vite.config.ts, tsconfig*.json | App entry and build/type configuration |
| src/main.tsx and src/App.tsx | App entry and single calculator screen |
| src/styles.css | Responsive layout, focus styles and touch controls |
| src/features/pricing/types.ts | Pricing input and result contracts |
| src/features/pricing/calculate.ts | Pure validation and calculation functions |
| src/features/pricing/calculate.test.ts | Money examples, discount boundaries and invalid inputs |
| src/features/pricing/PricingForm.tsx | Units, fields, optional costs and pricing mode |
| src/features/pricing/PriceSummary.tsx | Prices, estimated profit and copy quote |
| src/lib/settings.ts | Versioned local settings, safe parse, reset and storage failure handling |
| tests/calculator.spec.ts | Phone viewport happy path and error recovery |
| .gitignore | Exclude dependencies, builds, local settings and environment files |
| .github/workflows/ci.yml | Install from lockfile, typecheck, tests and build; no deployment |
| docs/implementation-plan.md | This scope, formulas and release criteria |

Create a root README with setup commands once the app exists. Since this repository shares the owner's username, consider whether that README should also appear on the GitHub profile before adding it.

## Milestones and completion criteria

### 1. Scaffold and pricing engine
Create the app scaffold and pure calculator. Add tests for both pricing modes, 0% and 100% discount, negative profit, fractional grams/time, cent rounding, zero cost, invalid numbers and zero spool weight.
Done: clean install, typecheck, tests and production build pass; reference case matches exactly.

### 2. Usable phone calculator
Build the form, results, family discount shortcut and quote copy. Add versioned local defaults with reset, including recovery from corrupt or unavailable storage.
Done: calculator works at 360 px without horizontal scroll; keyboard and screen-reader labels work; invalid inputs cannot produce a misleading quote; browser smoke test covers the reference case and correction of an invalid spool weight.

### 3. Private release
Choose a host that supports access control, protect both the main address and any direct preview/origin URL, then deploy a production build behind sign-in. A public source repository does not make a hosted app private. Do not enable public GitHub Pages for this requirement. Keep secrets out of source and browser bundles.
Done: authorized phone access works, an unauthenticated browser cannot load the app through any deployment URL, and no calculator inputs are sent to analytics or third parties. Public publishing requires the owner's approval.

### 4. Quotes and customers
After daily use validates the calculator, add saved quote snapshots, products, customers and export/backup. Quotes retain their original inputs, formula version and outputs when defaults change. Add an authenticated backend/database only when cross-device sync is needed, with per-owner access enforcement.
Done: quotes can be retrieved without recalculation changing historical prices; storage and access checks pass.

### 5. Business discovery pilot
Start with San Antonio bakeries as one configurable niche; allow expansion to other businesses and printed products. Add a server-side provider adapter for an approved business-listing API, deduplication by provider ID, source links, discovery time and lead statuses. Keep API keys server-side. Run discovery manually first, then add a scheduled worker with limits, retry/backoff and deduplication once results are useful.
Done: a small sourced lead list can be reviewed and connected to a product/quote without duplicates. Provider terms, costs and credentials are resolved before enabling collection. Outreach drafts require review; automatic discovery does not automatically send messages.

## Expansion boundaries

Keep pricing code independent of React, local storage and discovery providers. Later features can live beside pricing under src/features/quotes, customers and leads; add a server and scheduled worker when those milestones need them. Avoid an early monorepo, scraper, CRM framework or automated outreach system.

Suggested first implementation change: scaffold plus tested pricing engine (milestone 1). Next change: phone interface (milestone 2). No live app, customer records, credentials or business outreach is created by this planning document.
