MSZ Freight & Logistics — Website Deployment Package
═════════════════════════════════════════════════════

FILES TO UPLOAD (all must be in the same folder on your server):
────────────────────────────────────────────────────────────────
  index.html        — main website (77KB)
  logo.png          — MSZ logo used in nav, hero, footer (33KB)
  og.png            — social share image for WhatsApp/LinkedIn (40KB)
  truck-fleet.jpg   — Scania R500 fleet photo (75KB)
  truck-quarry.jpg  — red tipper at quarry photo (73KB)
  favicon.svg       — browser tab icon (<1KB)
  sitemap.xml       — for Google Search Console
  robots.txt        — search engine crawl rules

TOTAL UPLOAD: ~300KB across 8 files

BEFORE GOING LIVE — update these in index.html:
────────────────────────────────────────────────
  1. Phone number:  +27 000 000 0000  →  your real number
  2. WhatsApp link: wa.me/27000000000 →  wa.me/27[your number]
  3. Email:         info@msz.co.za    →  your real email
  4. Address:       Gauteng, SA       →  your full address
  5. OG image URL:  update msz.co.za  →  your actual domain
  6. Schema URL:    https://msz.co.za →  your actual domain

REPEAT VISITOR PERFORMANCE:
────────────────────────────
  First visit:   ~300KB total download
  Repeat visit:  ~77KB (images served from browser cache)
  vs old single file: 527KB every visit, nothing cached

HOSTING RECOMMENDATION:
────────────────────────
  Any standard web host works. For best performance:
  - Enable gzip compression on the server
  - Set cache-control headers: images → 30 days, HTML → 1 day
  - Point sitemap.xml in Google Search Console after launch
