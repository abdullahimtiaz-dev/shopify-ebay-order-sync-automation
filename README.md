# shopify-ebay-order-sync-automation
n8n workflow automating multi-channel order sync, deduplication, and inventory management for Shopify + eBay stores

## What it does
Automates order processing across Shopify and eBay in a single pipeline:
- Ingests orders from both platforms in real time (webhooks)
- Verifies and deduplicates orders so nothing gets processed twice
- Matches order line items to inventory by SKU
- Updates stock levels automatically
- Sends low-stock alerts and drafts reorder emails when inventory runs low
- Includes a built-in failsafe that halts inventory updates on unmatched SKUs or repeated errors, protecting data integrity

## Status
Core order sync, deduplication, and inventory automation are fully working.
Shopify/eBay push-back sync (writing fulfillment updates back to the source platforms) is in progress.

## Demo
[Loom link here]

## Architecture
[embed your screenshot here]
