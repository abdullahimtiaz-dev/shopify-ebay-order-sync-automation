# Multi-Channel Inventory Sync & Auto-Reorder (Shopify + eBay)

An n8n workflow that keeps stock in sync across Shopify and eBay, stops the same item from getting oversold on both platforms, and automatically flags low stock before it becomes a stockout, with its own error-handling workflow watching in the background.

## Why this exists

If you sell the same SKU on Shopify and eBay, a sale on one doesn't touch stock on the other. Without something bridging that gap, you'll eventually oversell, accept an eBay order for something that just sold out on Shopify five minutes ago. This workflow treats Airtable as the single source of truth and pushes every stock change to whichever channel didn't just make the sale.

## How it flows

**Orders come in** from both Shopify (via its native trigger) and eBay (via a webhook that also handles eBay's required one-time verification handshake, it needs to respond to a GET challenge before eBay will even start sending real notifications, and both request types share the same endpoint).

**Every order gets normalized** into one shape regardless of which platform it came from, then checked against an Airtable log before anything else happens, this is what makes duplicate webhook deliveries and retries harmless instead of dangerous. Cancelled orders and anything already processed get skipped here.

**Each line item is looked up by SKU.** If it matches inventory, stock gets recalculated and updated. If it doesn't match anything, it's logged separately and someone gets pinged on Slack instead of the order silently failing to update stock.

**The new stock number gets pushed to the other channel** — not the one where the sale happened, since that channel already decremented itself. A small safety buffer is subtracted before pushing, so there's always a little headroom in case two orders land on different channels close together.

**If stock drops below the reorder threshold**, it gets flagged, Slack gets a heads-up, and a reorder email gets drafted to the supplier (drafted, not sent, someone still needs to actually approve it before it goes out).

**A separate error-handling workflow** catches anything that breaks anywhere in this pipeline, logs it to Airtable, and posts to Slack with the failed node and a link straight to the execution.

## A few decisions worth calling out

- Deduplication happens before any stock math runs, not after, order matters here.
- Pushing to the *other* channel (not the one that sold) avoids double-decrementing stock that was already reduced natively.
- Reorder emails are drafts, not auto-sends, automation should flag the decision, not make it.
- One shared error handler instead of per-node error handling, so failures are visible immediately instead of showing up later as a stock discrepancy nobody can explain.

## Stack

n8n · Airtable · Shopify Admin API · eBay Sell Inventory API (OAuth2) · Slack · Gmail

## Status

End-to-end working: order intake, dedup, SKU matching, stock updates, push-back to both Shopify and eBay, low-stock alerts, and error handling, tested against Shopify's live API and eBay's sandbox.

## Demo

- [Order processing & inventory update](https://www.loom.com/share/e8f4242403f24870a30a69349c2d0d4a)
- [Low-stock alert & reorder draft](https://www.loom.com/share/3349bcebbb0c43d898d0ed48e0a786ae)

## Architecture

![Multi-channel order sync]()

## Error Handling

![Error handler]()
