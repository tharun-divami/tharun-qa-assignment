# Test Execution Report — Polymer Shop

**Application:** https://shop.polymer-project.org/
**Run started:** 2026-06-29T11:58:39.157Z
**Wall-clock duration:** 63.1s
**Browser:** Chromium (Playwright)

## Summary

| Total | Passed | Failed | Skipped |
|------|--------|--------|--------|
| 19 | 0 | 19 | 0 |

## Sanity (4)

| # | Test | Status | Duration |
|---|------|--------|----------|
| 1 | adding a product updates the cart badge | FAILED | 0.3s |
| 2 | home page loads with the app shell | FAILED | 0.3s |
| 3 | all four shopping categories are linked | FAILED | 0.3s |
| 4 | navigate from home to a category to a product | FAILED | 0.4s |

## Regression (12)

| # | Test | Status | Duration |
|---|------|--------|----------|
| 1 | button background turns black while pressed | FAILED | 0.4s |
| 2 | empty cart shows the empty message | FAILED | 0.3s |
| 3 | added product appears as a line item in the cart | FAILED | 0.3s |
| 4 | two different products produce two line items | FAILED | 0.3s |
| 5 | category "mens_outerwear" lists products | FAILED | 0.3s |
| 6 | category "ladies_outerwear" lists products | FAILED | 0.3s |
| 7 | category "mens_tshirts" lists products | FAILED | 0.3s |
| 8 | category "ladies_tshirts" lists products | FAILED | 0.3s |
| 9 | opening a product by name lands on its detail page | FAILED | 0.4s |
| 10 | submitting an empty form does not place the order | FAILED | 0.3s |
| 11 | detail page shows title, price, size and quantity controls | FAILED | 0.3s |
| 12 | adding with explicit size and quantity updates the cart | FAILED | 0.3s |

## E2E (2)

| # | Test | Status | Duration |
|---|------|--------|----------|
| 1 | a shopper can buy a product end to end | FAILED | 0.4s |
| 2 | checkout succeeds for an international customer | FAILED | 0.3s |

---
_Generated from `reports/results.json` by `scripts/gen-execution-report.js`._
