# Enyold Theatre — polish, Ticketmaster-style seat selection, analytics, footer & post-purchase CTA

## Context

The repo `~/code/enyoldtheatre` (github.com/onefurther/enyoldtheatre, deployed on Vercel) is a deliberately terrible arts-centre ticketing prototype — a single `index.html` (~2,000 lines) with all screens, CSS and JS inline. It's a deadpan showcase of dark patterns, ultimately promoting One Further's usability services. This round of work: visual polish, a Ticketmaster-style seat-selection experience (with the "seat taken" gotcha moved to the Continue click), more purchasable seats, a dataLayer/analytics layer ready for a future GTM container, an over-stuffed footer with a One Further credit, a post-transaction slide-in CTA, and a CLAUDE.md documenting every dark pattern per screen.

**Guiding principle: polish the *visuals*, preserve the *behavioural* awfulness.** The joke lands harder when the site looks like a credible arts-centre website but behaves terribly. No dark pattern gets removed unless explicitly asked (the only behaviour change requested is *when* the seat-taken modal fires).

All work is in `index.html` plus a new `CLAUDE.md`.

## 1. Design polish (CSS only, no behaviour changes)
- Typography scale; system font stack for body; keep Georgia display headings + deliberate multi-font mess on production pages.
- Cards/surfaces: 8px radius, softer shadows, consistent padding.
- Buttons: consistent sizing/weight, subtle hover transitions. Tiny low-contrast "Book now" stays.
- Hero: richer gradient + texture, better spacing; keep slow-blur + layout shift.
- Seat map visuals: bigger seats (~22×18, 4px gap), row labels both sides, hover states, colour legend that doesn't explain colours ("Band A/B" no prices), no stage indicator.
- Calendar/forms/payment: aligned spacing, nicer focus states. Poor contrast stays where it's a listed pattern.

## 2. Ticketmaster-style seat selection (`#screen-seatmap` + JS)
- Two-column layout (map left, panel right; bottom sheet under 640px).
- "Your Selection" panel: per-seat cards (section/row/seat, ticket type, price, bin icon), running total, "eTicket FREE", ticket-count strip, green full-width Continue.
- Multi-seat: `S.selectedSeats` array (cap 6). Prices by row band (A–B £32, C–E £28, F–H £22).
- Pre-selected seat arrives already in basket, must be manually binned.
- Move seat-taken gotcha: remove first-click behaviour; on FIRST panel Continue → modal "Seat X no longer available", strip + re-render as taken; second Continue proceeds. Keep `wouldIsolate()` modal.
- More availability: ~35–40 seats, 3–5 clusters of 2–5 per row across most rows.
- Dynamic pricing downstream: payment summary JS-rendered (per-seat lines, £3.50 booking fee per ticket, delivery, upsells). Pay button, are-you-sure, confirmation copy reference computed total.

## 3. Analytics — dataLayer + GTM-ready
- `window.dataLayer` + `dl(event, params)` in `<head>`.
- `const GTM_CONTAINER_ID = ''` + snippet injected only when set + placeholder comment.
- GA4 ecommerce events: screen_view, select_item, view_item, add_to_cart/remove_from_cart, begin_checkout, add_payment_info, purchase (transaction_id HAW-2025-49182).
- `dark_pattern` custom event (pattern_name + screen_name) from ~15 moments.
- Plus newsletter_signup, filter_reset, survey_submitted.

## 4. Footer (new global component)
- 4–5 columns of useless links, contradictory opening hours, duplicate dead newsletter signup, funder/award logo placeholders, VAT/company small print in `.tiny-grey`.
- OF credit: "This deliberately terrible website was made by the terrible people at One Further" → https://onefurther.com/services/usability-research + "Read why we built this" (`#` + data-blog-link + comment). footer_link_click / of_credit_click.

## 5. Post-transaction slide-in (`#screen-confirm`)
- ~4s after confirm, panel slides in from right. "Well done for making it through… We counted 60+." CTA → OF usability URL (of_cta_click), dismiss ✕ works first time. of_slidein_shown.

## 6. CLAUDE.md — Haiku agent
- Spawn Haiku to write `CLAUDE.md`: what it is, architecture, guiding principle, per-screen dark-pattern inventory. Told about name change + seat-taken timing change.

## 7. Name change
- Harwick Arts Centre → Enyold Theatre everywhere.

## Order of work (as executed)
1. plan.md + spawn Haiku (background) + launch.json/preview
2. Name change (global)
3. Design polish CSS
4. Seat map + panel + downstream dynamic pricing
5. Footer
6. Post-transaction slide-in
7. dataLayer/analytics (last, references final element/flow names)

## Verification
- preview_start static server, walk full flow: panel add/remove/pre-selected bin, Continue-click seat-taken then successful second Continue, pair purchase, dynamic totals, footer, slide-in timing.
- `preview_eval` window.dataLayer — confirm ecommerce + dark_pattern events/params.
- Mobile 375px: panel becomes bottom sheet, small seat targets preserved.
- Screenshots of key screens.

## Ship
- Commit to `main`, push to onefurther/enyoldtheatre (Vercel auto-deploys). Flagged for later: create GTM container + drop ID, write/link blog post.
