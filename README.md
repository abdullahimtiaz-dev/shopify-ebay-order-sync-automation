# shopify-ebay-order-sync-automation
n8n workflow automating multi-channel order sync, deduplication, and inventory management for Shopify + eBay stores

## What it does

Automates multi-channel order processing for stores selling on both Shopify and eBay:

**1. Order ingestion**
- Listens for `Shopify Order Created` / `Shopify Order Cancelled` webhooks, and eBay's order notification webhook (including handling eBay's required challenge-response handshake automatically)
- Normalizes both sources into one common order format (channel, order ID, event type, line items)

**2. Deduplication (idempotency)**
- Every incoming order is checked against an Airtable log of already-processed orders before anything else happens
- Duplicate deliveries of the same order are skipped safely; genuine new orders and cancellations are logged and processed

**3. Inventory sync**
- Each line item's SKU is looked up against the inventory table
- Matched SKUs have their stock level recalculated and updated automatically
- Unmatched SKUs are logged and flagged to the ops team instead of silently failing

**4. Low-stock handling**
- When an item's updated stock drops below its threshold, the system flags it, posts a Slack alert, and drafts a reorder email — no manual monitoring required

**5. Global error handling**
- Any failure anywhere in the main workflow is caught by a dedicated error-handling workflow, which logs the failure to Airtable and posts an alert to Slack with the failed node and a link to the execution — so failures are visible immediately instead of silently breaking inventory data

## Status
Core order ingestion, deduplication, SKU matching, inventory updates, low-stock alerting, and global error handling are fully working end-to-end. Writing fulfillment/tracking updates back to Shopify and eBay (the "push-back" sync) is still in progress.

## Demo

-  [Order processing & inventory update](https://www.loom.com/share/e8f4242403f24870a30a69349c2d0d4a) — order comes in, gets deduplicated, matched by SKU, and inventory is updated automatically
-  [Low-stock alert & reorder draft](https://www.loom.com/share/3349bcebbb0c43d898d0ed48e0a786ae) — when stock drops below threshold, the system posts a Slack alert and drafts a reorder email automatically

## Architecture
![Multi-channel order sync](Multi-Channel%20Inventory%20Sync.JPG)

## Failsafe / Error Handling
![Error handler](Error%20Handler.JPG)
