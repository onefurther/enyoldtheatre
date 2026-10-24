# Enyold Theatre Ticketing Website — Claude.md

## Project Overview

This is **Enyold Theatre** — a satirical, single-file HTML prototype of a user-hostile ticketing website for a fictional arts venue. Built by the digital agency **One Further**, it is deliberately designed to showcase "dark patterns" and poor user experience as a counter-example for usability testing demonstrations.

**Critical Design Principle:** The site looks professionally designed but behaves terribly on purpose. The frustrating behaviours *are the point* of the project. When editing, always **preserve the dark patterns** — do not "fix" intentional frustrations unless explicitly asked by the user.

---

## Architecture

### Single-File Structure
- **File:** `index.html` (~3300 lines)
- **Assets:** `images/og-image.png` (social preview), `images/stock/` (optional stock photos, see its README). `/images/productions/` deliberately does not exist (broken images on production pages)
- **All content:** HTML, CSS, and JavaScript are inline in one file
- **No external dependencies** — fully self-contained, mockup-only (no real backend)

### Hosting
- **Live:** https://offcuts.onefurther.dev/terrible-ticketing/ (Cloudflare; previously Vercel). Served under a sub-path, so keep asset paths relative (`images/...`, never `/images/...`) and keep the trailing slash on the URL
- **Repo:** github.com/onefurther/enyoldtheatre
- Canonical, `og:url`, `og:image` and `twitter:image` use the absolute live URL; update them if the site moves

### Navigation Model
The site is a **screen-based single-page application (SPA)**:
- Each screen is a `<div class="screen" id="screen-...">` container
- Navigation via the `navTo(screenId)` function, which toggles the `.active` class on screens
- Global mutable state object `S` at the top of the `<script>` tag tracks session data
- Screens scroll to top on navigation via `window.scrollTo(0,0)`

### Global State (`S`)
Core variables track journey frustrations:
- `selectedSeats` — ARRAY of seat IDs, e.g. `["D-10", "E-11"]` (cap 6 seats; supports MULTI-selection)
- `preselectedSeat` — "D-10" (arrives pre-added to basket; user must manually bin it)
- `takenSeats` — dynamically filled array of seats stripped during the "Continue" button gotcha
- `seatTakenFired` — tracks whether the seat-taken modal has fired yet
- `acceptedUpsells` — array of {title, amount} for upsells the user accepted
- `orderTotal` — computed total from `computeOrder()`
- `upsellIndex` — current upsell position (0–5 in the array)
- `timerSeconds` — countdown timer (starts at 300 = 5 minutes)
- `captcha1Done`, `captcha2Done` — tracks two-CAPTCHA dark pattern
- `a11yDismissCount` — counts accessibility widget dismissals
- `swapPayBtns` — 25% chance of swapping "Pay" and "Cancel" button positions
- `cookieBannerShown` — tracks if cookie banner has been shown
- `cookieSavedStep` — tracks the two-step cookie save process

### Data Arrays
**`EVENTS[]`** — Four productions:
- An Evening of Music & Things (Main House)
- Performance (Main House)
- Theatre – see website (Studio)
- Dance (Main House)

**`UPSELLS[]`** — Six forced upsells (steps 1–6):
1. Enyold Membership (£4.99/month)
2. Support Enyold Arts (£5.00 donation)
3. Programme (£4.50)
4. Booking insurance (£4.95)
5. Parking (£12.00)
6. Carbon offset (£3.80)

### Global Fixed UI (Outside Screens)
These elements persist across all screens:
- **Site header** (`.g-nav`) — utility bar (box office, Access, Membership, Donate, Sign in) + main bar (logo, Discover/Visit/Get involved/Support us/About us, search, account, basket, "Book tickets"). Under 860px collapses to logo + icons + burger (`toggleMobileMenu()`). Search always "temporarily unavailable"; basket count stays at 0
- **Timer bar** (`#timer-bar`) — replaces the header when booking starts (`body.booking`); removed again by `stopTimer()` on the error-styled confirmation
- **Skip to the end** (`#skip-pill`) — see Launch features
- **Facilitator bar** (`#fac-bar`) — only with `?facilitator`, see Launch features
- **Newsletter popup** — triggers 5 seconds after page load, blocks access
- **Cookie banner** — appears ~1.8 seconds after page load with "847 trusted partners"
- **Cookie preferences overlay** — requires two saves to fully close
- **Live chat widget** — fixed bottom-right, always live with generic auto-responses
- **Accessibility widget** — bottom-right button that re-opens itself 3 times before staying closed
- **Toast notifications** — transient messages for various events
- **Modals** — overlaid dialogs for seat taken, "are you still there", login, isolation, confirmation
- **Global footer** (`.site-footer`) — injected on every content screen
- **Post-transaction slide-in** (`#of-slidein`) — appears ~4s after confirmation

---

## Screen Inventory & Dark Patterns

### **Screen: Home** (`#screen-home`)
**URL Path:** Entry point

**Dark Patterns:**
- **Hero image blur delay** — the whole carousel (`#hero-img.hero-slides`) starts blurred and takes 14 seconds to come into focus
- **Newsletter popup auto-trigger** — appears uninvited 5 seconds after page load with full-screen overlay (z-index 950), blocking access to page
- **Carousel auto-advance** — four photo slides (one per production) rotate every 3 seconds, too fast to read; the progress dots look clickable but do nothing. Slides are clickable (no button; the hero "Book now" was removed on purpose)
- **Cookie banner timing** — appears ~1.8 seconds after load, again blocking access
- **"This week at Enyold" strip** — seven real dates from today with invented event counts; every day opens the same unfiltered listing
- **Explore by genre tiles** — six tiles, all open the same unfiltered listing
- **Inconsistent data** — home cards swap the prices and dates shown on What's On; 2025 dates kept as an "unmaintained site" gag (do not update)
- **"A Welcome from our Artistic Director"** — long, deliberately verbose wall of text with self-important language; no option to skip (now sits below the featured events)
- **Featured events cards** — photo cards with genre tag, venue and "More info"; click anywhere to open the production
- **"Support Enyold" band** — Become a member / Make a donation buttons that go nowhere

### **Screen: Events Listing** (`#screen-events`)
**URL Path:** "What's On"

**Dark Patterns:**
- **Filter spinner trap** — "Apply filters" button shows a fake spinner and "Applying…" for 1.2 seconds
- **Filter gotcha** — Selecting "Theatre" genre or "Under £15" price shows "No results" and requires clicking again to reset
- **Re-populating filter selects** — filters auto-reset on second click, trapping users into re-filtering
- **Pagination dots** — five dots shown for four events; clicking dots does nothing
- **Brochure banner** — "Download brochure (PDF, 12MB)" does nothing
- **Feeling filter and "Free events only" toggle** — the toggle empties the grid (nothing is free)
- **Cards** — same photo card design as the home page, built from `EVENTS[]` (`img`, `genre`, `venue` fields)

### **Screen: Calendar / Date Selection** (`#screen-calendar`)
**URL Path:** Event production page → "Book tickets"

**Dark Patterns:**
- **Calendar starts in the past** — `initCalendar()` subtracts one month on load, forcing users to navigate forward before seeing current dates
- **Past dates greyed out** — all past date cells are styled as "soldout"
- **Limited availability badges** — "LIMITED" and "LIMITED AVAILABILITY" badges on most available dates, creating false scarcity
- **"12 people currently viewing" notice** — fake viewer count with pulsing red dot, fluctuates every 4 seconds
- **Misleading newsletter toast and cookie banner again** — both shown on entry to the seat map

### **Screen: Seat Map** (`#screen-seatmap`)
**URL Path:** Event date/time selected → "Seat selection"

**Layout:** Ticketmaster-style TWO-COLUMN design:
- **Left:** Seat map (Rows A–H, 18 seats per row, ~42 available in clusters)
- **Right:** "Your Selection" side panel (becomes bottom sheet under 640px width)
- **Colour legend:** Shows Band A/B/C/D/E/F/G/H swatches; **deliberately never states prices or bands** (pattern preserved)
- **No stage indicator** (pattern preserved)

**Row-band pricing (never shown on map):**
- Rows A–B: £32
- Rows C–E: £28
- Rows F–H: £22

**Seat state colours:**
- Colour classes by row (`ROW_COLOUR`): A–B `.s-a`, C–D `.s-b`, E–F `.s-c`, G `.s-d`, H `.s-e`. They don't line up with the price bands (pattern)
- `.s-taken` — greyed out (unavailable)
- `.presel` — golden outline (pre-selected D-10)
- `.selected` — green outline (user-selected)

**Selection Panel ("Your Selection"):**
- Sticky header: "Your Selection" title + count indicator (6 pips)
- Each selected seat is a card: `SECTION · Row X · Seat Y | Full Price Ticket | £XX | 🗑`
- Footer shows ticket subtotal, "eTicket delivery FREE" and total (the booking fee and £1.25 delivery only appear at payment)
- Green "Continue" button (disabled if no seats)
- At 640px: fixed bottom sheet, rounded corners, flex layout

**Dark Patterns:**
- **Pre-selected seat already in basket** — D-10 arrives ALREADY SELECTED and in the panel; user must manually bin it (nudge you must actively remove)
- **Seat isolation validation** — Checked only on Continue (`selectionIsolatesSeat()`): if the settled selection strands a single seat, the modal shows
- **"Seat no longer available" gotcha** — Fires on Continue only if the pre-selected D-10 is still in the basket. Strips D-10, marks it taken, shows a generic modal that never names the seat. If the user binned D-10 first, it never fires
- **Zoom buttons do nothing** — Three zoom buttons (+, −, ↺) all call `fakeZoom()` and show toast "Zoom is not available on this device"
- **Progress indicator misleading** — Shows "Step 2 of 3" but there are more steps ahead
- **"Cannot be guaranteed until payment is complete"** — Disclaimer at bottom removes any sense of security
- **Accessibility widget re-opens** — Will re-open itself twice more if dismissed

### **Screen: Booking Form** (`#screen-form`)
**URL Path:** Seat selected → "Your details"

**Dark Patterns:**
- **Form that wipes itself** — Any empty field (marketing excluded) clears the WHOLE form, re-ticks every marketing box, shows the error wall and reveals the password rules (which are hidden until then)
- **Wall of required personal data** — Title, first/last name, DOB, email (twice), phone, full address, county, emergency contact details, password (12+ chars, uppercase, number, special character, no spaces, not matching last 8)
- **Marketing emails pre-checked** — Four checkboxes for marketing (email, post, SMS, phone) are **all checked by default**; unchecking requires individual action
- **Fake postcode lookup** — Button shows error "Sorry, we couldn't find that postcode" after 4 seconds; user must enter manually
- **Two reCAPTCHA checkboxes** — First CAPTCHA checkbox triggers second one after 1.2 seconds
- **Second CAPTCHA requires scroll** — Page auto-scrolls to new CAPTCHA at bottom
- **Terms & Conditions wall** — Scrollable box with 10 dense terms; reading required but user must still submit
- **"All fields are required" warning** — Printed at bottom of form
- **"Are you still there?" modal** — Appears 18 seconds after form is loaded and displayed, asking if session should continue
- **Login interrupt modal** — After form submission, modal appears: "Create an account to continue"; user must dismiss to proceed, but cannot continue as guest (only via "Or, continue as guest" link)
- **Confirmshaming button** — "Create account & continue" vs. small "Or, continue as guest" link

### **Screen: Upsells** (`#screen-upsell`)
**URL Path:** Form submitted → "Upsell 1 of 6"

**Dark Patterns:**
- **Six mandatory upsell screens** — Each upsell (membership, donation, programme, insurance, parking, carbon offset) requires clicking through
- **Pre-checked "Yes" checkbox** — All upsells have checkbox pre-ticked: "Yes, add this to my booking"
- **"No thanks" is a small link, not a button** — Visual hierarchy favours the primary button
- **Upsells increment a counter** — "Step 1 of 6", "Step 2 of 6", etc., creating false urgency
- **After all 6 upsells, user forced to survey** — No way to skip survey

### **Screen: Survey** (`#screen-survey`)
**URL Path:** Upsells completed → "Tell us about yourself"

**Dark Patterns:**
- **5-question survey in booking flow** — Collects household income (intrusive), experience rating, how they heard about event, attendance frequency, comments
- **Star rating with no neutral option** — 1–5 stars only; must choose
- **Income bracket question** — Collects sensitive data (£20k–£80k+ brackets)
- **"Just 30 seconds"** — Survey claims to be brief but involves 5 questions with radio groups and textarea

### **Screen: Newsletter Re-Signup** (`#screen-newsletter2`)
**URL Path:** Survey submitted → "Newsletter 2"

**Dark Patterns:**
- **Second newsletter push** — User already saw newsletter popup on home; now forced to see it again with pre-checked checkbox
- **Pre-filled email** — Shows placeholder "your.email@example.com" (not user's input)
- **"No thanks, skip this step"** — Small confirmshaming link vs. primary button

### **Screen: Payment** (`#screen-payment`)
**URL Path:** Newsletter dismissed → "Payment"

**Dark Patterns:**
- **Order summary computed dynamically** — Shows all seats (with dynamic pricing per row) + booking fee (£3.50 PER TICKET) + delivery (£1.25) + any accepted upsells. Total updates live based on seat selection
- **"Pay £X" button reflects live total** — Updated by `renderPaymentSummary()` whenever order changes
- **Button swap 25% of the time** — `S.swapPayBtns` flips "Pay £X" and "Cancel" buttons using `.reversed` flex direction
- **"DO NOT REFRESH THIS PAGE DURING PAYMENT"** — All-caps warning creates anxiety
- **Confirmation modal before payment** — one "Are you sure?" modal (the second "absolutely sure" step was removed)
- **Card security false reassurance** — "256-bit SSL encryption" and list of 5+ companies (TicketHouse, EventStream, SecurePay, GlobalPay) creates confusing chain of responsibility

### **Screen: Processing** (`#screen-processing`)
**URL Path:** Payment confirmed → "Processing your payment…"

**Dark Patterns:**
- **Fake progress bar** — Increments slowly to 90% but never reaches 100%; simulates payment processing
- **Multi-stage text updates** — "Contacting payment provider…" → "Verifying card…" → "Awaiting bank…" → "Confirming…" → "Finalising…"
- **"This is taking longer than expected"** — Warning appears at 5 seconds, creating anxiety
- **"Do not close this window or press the back button"** — Threat reinforces stress
- **White screen pause** — After 7.5 seconds, switches to white screen (`#screen-white`) for 4 seconds, disorienting user
- **Then shows error-styled confirmation** — Not an error, but looks like one

### **Screen: Error-Styled Confirmation** (`#screen-confirm-err`)
**URL Path:** Processing → "Something went wrong"

**Dark Patterns (The Entire Screen):**
- **Error styling for success** — Red header, ⚠️ icon, "Something went wrong" h2 in red
- **Error detail text** — "We encountered an unexpected error while processing your request"
- **Error code + session ID** — "Error code: TKTHSE_5504 · Session: 8f3a2c91 · Time: 19:43:22"
- **Button says "Continue"** — Leads to the real confirmation
- **The joke:** User cannot tell if the transaction succeeded
- **Timer stops here** — `stopTimer()` hides the timer bar and brings the header back

### **Screen: Success Confirmation** (`#screen-confirm`)
**URL Path:** Error screen → "Transaction complete"

**Dark Patterns:**
- **Deadpan card** — "Transaction complete. Your e-tickets will arrive shortly before the show date."
- **End-of-journey reveal** (`#reveal`) — see Launch features
- **Post-transaction slide-in** — ~4 seconds after this screen shows, a panel slides in from the right with a One Further usability-research CTA and a dismiss ✕ that works (unlike everything else)

### **Global UI: Timer Bar**
**Behaviour:**
- Starts at 5:00 (300 seconds) when user enters seatmap
- Counts down every second
- Replaces the navigation bar (nav hides, timer bar shows)
- Colour changes: Yellow (< 3 min) → Orange (< 2 min) → Red (< 1 min) → Flash red (< 30 sec)
- Shows "12 people currently viewing" count that fluctuates every 4 seconds
- **Dark pattern:** Creates artificial urgency; user feels pressured to complete booking before timeout
- **Session extension at 3-minute mark** — If user is on form screen, timer extends by 60s when it hits 3:00

### **Global UI: Newsletter Popup** (`#newsletter-popup`)
**Behaviour:**
- Auto-triggers 5 seconds after page load
- Full-screen overlay with z-index 950
- Email input + "Subscribe to our newsletter" button
- **"Close ✕"** button in top-right (easy to dismiss)
- On submit, fires confetti animation (60 confetti bits)
- Shows success toast: "You've successfully joined our mailing list!"
- Appears again on seatmap entry

**Dark pattern:** Interrupts user journey uninvited; fires celebration confetti to discourage cancellation

### **Global UI: Cookie Banner** (`#cookie-banner`)
**Behaviour:**
- Appears ~1.8 seconds after page load
- States "We and our **847 trusted partners**" use cookies (absurd number to seem authoritative)
- Offers three buttons: "Reject non-essential", "Manage preferences", "Accept all"
- "Accept all" button is purple (primary colour), others are subtle grey
- On mobile, appears at bottom; on desktop, modal at centre

**Cookie Preferences Overlay** (`#cookie-prefs-overlay`):
- Lists 5 cookie types: Strictly necessary, Performance, Functional, Targeting, Social media
- All categories **enabled by default** (noted: "All categories are enabled by default to ensure the best possible experience")
- Each category can be expanded
- First time clicking "Save preferences" button, message appears: "✓ Preferences saved. Please also confirm your choices on the previous screen."
- Requires second click of "Confirm and close" to actually close

**Dark patterns:** 
- "847 partners" inflates trust through false authority
- Preferences defaulted to ON, forcing users to opt-out rather than opt-in
- Two-step save process for one action
- "Reject" and "Manage" buttons de-emphasized compared to "Accept all"

### **Global UI: Live Chat** (`#live-chat`)
**Behaviour:**
- Fixed bottom-right corner (z-index 600)
- Header: "Live Chat Support" with green dot (online status)
- Chat toggle button (−/+)
- Initial message: "Hi there! I'm Harriet from the Enyold box office team. How can I help you today?"
- User can type and send messages
- Auto-response after 800ms: "Thanks for your message! Our team typically responds within 2–3 business days…"

**Dark pattern:** Live chat is fake; auto-response says it takes 2–3 business days despite "live" label and green dot

### **Global UI: Accessibility Widget** (`#a11y-widget`)
**Behaviour:**
- Fixed bottom-right corner, above live chat (z-index 600)
- Black circular button with ♿ emoji
- Clicking opens panel: "Accessibility options"
- Shows 5 options: Text size, Contrast, Cursor size, Dyslexia font, Reduce motion
- None of these options are functional (no JavaScript implementation)
- Message at bottom: "You must dismiss this panel three times before it closes permanently."
- Clicking close button toggles it closed, but automatically re-opens after 200ms for first 3 dismissals
- After 3rd dismissal, stays closed

**Dark patterns:**
- Non-functional options (fake accessibility)
- Forced 3-dismiss requirement before staying closed (trap)
- Message mocks accessibility by requiring repeated actions

### **Global UI: Global Footer** (`.site-footer`)
**Behaviour:**
- Injected by `injectFooters()` on every content screen (except processing and white-screen pause)
- Over-stuffed with useless-link columns:
  - "About Enyold" (Our Story, Awards, Corporate Hospitality, Governance, Annual Report)
  - "Your Visit" (Getting Here, Venue Hire FAQs, Accessibility Statement, Food & Drink, Cloakroom)
  - "Support Us" (Donation, Member, Legacy, Volunteering, Modern Slavery)
  - "Legal & Policies" (Cookie Policy, Privacy Policy, Terms of Sale, Press Enquiries)
  - "Opening Hours" (contradicts live-chat hours: "Mon–Sat 9am–8pm, Sun 11am–5pm", plus duplicate newsletter box)
- **Award/funder logo placeholders** (6 placeholder boxes)
- **VAT/company small print** — "Enyold Theatre is a trading name of TicketHouse Ltd…"
- **One Further credit** — "One Further did this on purpose. They're not sorry." (links to onefurther.com/services/usability-research; hidden in facilitator mode)
- **"Read why we built this" link** — Placeholder (`data-blog-link`, href="#" for now)

### **Global UI: Post-Transaction Slide-In** (`#of-slidein`)
**Behaviour:**
- Appears ~4 seconds after the real confirmation screen (screen-confirm) shows
- Panel slides in from the right edge (transform translateX)
- ~380px width, max-width 92vw on mobile
- Content: "Well done for making it through." plus two short paragraphs about friction costing ticket sales
- One Further usability-research CTA button and "See your score ↓" (scrolls to the reveal)
- Close button (✕) in top-right — **works first time** (unlike everything else; the joke)
- After dismissal, stays dismissed

### **Global Modals**

**Modal: Seat Taken** (`#modal-seat-taken`)
- Triggers on Continue from the seat panel, only if the pre-selected D-10 is still in the basket
- Generic message that never says which seat was stripped
- Dismissed via "Choose another seat" button
- User must re-select to reach the form

**Modal: Still There** (`#modal-still-there`)
- Appears 18 seconds after entering form screen
- "Are you still there? Your session is about to expire due to inactivity."
- Buttons: "Yes, continue" and "No"
- Both buttons dismiss the modal (no functional difference)

**Modal: Login Interrupt** (`#modal-login`)
- Appears the first time the form is submitted with every field filled and both CAPTCHAs ticked
- "Create an account to continue"
- Requires email and password (12+ chars, etc.)
- Button: "Create account & continue"
- Link: "Or, continue as guest" (small, underlined)
- Clicking either leads to upsell screen

**Modal: Isolation** (`#modal-isolation`)
- Appears on Continue if the selection would leave a single seat stranded in a row
- Orange header: "Unable to select this seat"
- Button: "OK" dismisses it
- User must re-select different seat

**Modal: Are You Sure** (`#modal-sure`)
- One confirmation before payment: "You are about to be charged £X. This action cannot be undone."
- "Yes, continue" goes to processing; "Cancel" closes the modal

### **Global UI: Keyboard Trap** (Escape Key Behaviour)
- Escape key closes accessibility panel (if open)
- Escape key closes "Are you still there" modal (if open)
- **Escape key does NOT close newsletter popup** (trap — user must use the close button)
- Newsletter popup blocks the entire page with z-index 950, forcing user to click the small close button

---

## Launch features (end of journey, skip, facilitator, sharing)

### Dark pattern tally
- `dark(name)` now also records `name` in `S.seen`; `darkOnce(name)` only fires the first time (used for passive, per-visit patterns)
- `PATTERNS` — catalogue keyed by `pattern_name`: `[group, title, one-line explanation]`. `PATTERN_GROUPS` sets the reveal's section order (Interruptions, False urgency, Sneaky defaults, Obstruction, Misdirection, Hidden costs and data grabs)
- **When adding a new dark pattern, add a `PATTERNS` entry and call `dark()`/`darkOnce()` where it happens**, or it won't appear in the reveal
- Passive patterns are counted in `navTo()` (production page, calendar, form, survey, newsletter2) and the screen hooks (seat map, payment, processing)

### End-of-journey reveal (`#screen-confirm` → `#reveal`)
- Rendered by `renderReveal(order)` from `onEnterConfirm()`; a dark "behind the curtain" section in One Further's voice, below the deadpan Enyold card
- Headline "You survived N dark patterns" (or "You gave up after N" when skipped), three stats (patterns found of total, time since load, £ added on top of the seats), the main CTA to onefurther.com/services/usability-research, secondary links (Read why we built this placeholder, Share your score, Try again), the grouped list of patterns hit and a collapsed list of the ones missed
- `shareScore()` uses the Web Share API, falling back to clipboard; `tryAgain()` reloads
- Slide-in keeps its CTA and adds "See your score ↓" (`scrollToReveal()`)
- Newsletter popup and cookie banner timers don't fire once the visitor is at the end (`body.at-end`)

### Skip to the end (`#skip-pill`, `skipToEnd()`)
- "Had enough? Skip to the end →" on every screen until processing. Bottom-left pill on desktop; slim strip under the header on phones (screens get extra top padding so it never covers content)
- Works first time. Closes all overlays, cancels any processing timers, adds a seat if the basket is empty (`ensureSeat()`), records `S.skippedFrom`, then goes to the error-styled confirmation so the punchline still lands
- Hidden in facilitator mode and at the end

### Facilitator mode (`?facilitator`)
- Adds `body.facilitator`: a faint prev/next bar bottom-left (`#fac-bar`, full opacity on hover) and `[` / `]` keys (ignored while typing in a field)
- Steps through `FAC_FLOW` (home → events → prod-1 → calendar → seatmap → form → upsell → survey → newsletter2 → payment → processing → confirm-err → confirm), closing overlays, cancelling processing timers, seeding a seat and resetting the upsell index as needed
- Hides the skip pill and the footer's One Further credit so participants don't get the joke early

### Stock photography slots
- `.ph` + `.ph-music / -performance / -theatre / -dance / -comedy / -family / -talks / -auditorium` layer `images/stock/<name>.jpg` over a stage-lit gradient. Missing file → gradient shows. See `images/stock/README.md` for the shot list

### Sharing and meta
- Meta description, theme colour, inline SVG favicon, Open Graph and Twitter card tags, `images/og-image.png` (1200×630). Social image and URL tags are absolute (live domain, see Hosting)

### New analytics events
- `skip_to_end` (`screen_name`, `patterns_found`), `reveal_shown` (`patterns_found`, `patterns_total`, `skipped_from`), `reveal_share` (`method`), `reveal_try_again`, `reveal_scroll_click`, `facilitator_step` (`screen_name`, `direction`), `search_click`, `genre_tile_click` (`link_text`), `brochure_download_click`, `filter_free_toggle`
- `of_cta_click` now carries `cta_location` (`reveal`, `reveal_footer`, `slidein`)
- New `dark_pattern` names: `carousel_autoplay`, `a11y_reopen`, `cookie_double_save`, `search_unavailable`, `filter_no_results`, `free_events_empty`, `brochure_dead_link`, `broken_production_page`, `calendar_past_month`, `fake_scarcity`, `misleading_newsletter_toast`, `hidden_seat_prices`, `false_progress`, `zoom_unavailable`, `prechecked_marketing`, `emergency_contact`, `postcode_lookup_fail`, `live_chat_autoreply`, `intrusive_survey`, `newsletter_resignup`, `drip_pricing`, `slow_processing`
- GTM: the existing `dark pattern` trigger picks up the new pattern names automatically; the new non-dark-pattern events need adding to the `engagement events` trigger if they should reach GA4

---

## Analytics — dataLayer & GTM

### Data Layer Setup
- `window.dataLayer` is initialised in `<head>`, with helper functions:
  - `dl(event, params)` — standard dataLayer push
  - `dlEcom(event, ecommerce)` — pushes `{ecommerce: null}` to clear, then the event with its payload nested under `ecommerce` (standard GA4 ecommerce shape)
- `const GTM_CONTAINER_ID = 'GTM-TDCTZVJ'` in `<head>` — live container; standard GTM snippet auto-injects when set (clear to `''` to disable)

### GA4 Ecommerce Events
All nest under `ecommerce` object (GA4 shape):
- `view_item` — when landing on a production detail page
- `select_item` — when clicking an event card on listings page
- `add_to_cart` — when selecting each seat on map
- `remove_from_cart` — when removing a seat from selection
- `begin_checkout` — entering the form screen (with order total and seat items)
- `add_payment_info` — "Are you sure?" modal before final charge
- `purchase` — on confirmation screen, with `transaction_id: 'ENY-2025-49182'` (not sent when the visitor skipped to the end), all seats + accepted upsells, order total

### Custom Dark Pattern Event
- `dark_pattern` event (params: `pattern_name`, `screen_name`) fires from ~38 moments (full list in `PATTERNS`, see Launch features). The originals:
  - `newsletter_popup` — on home page 5s delay
  - `cookie_banner_shown` / `cookie_banner_reshown` — first time / subsequent times
  - `preselected_seat` — when D-10 is added to basket on seatmap load
  - `seat_taken_modal` — when seat is stripped on Continue button
  - `seat_isolation_modal` — when isolation validation fires
  - `countdown_started` — timer begins on seatmap
  - `timer_extended` — session extension at 3-min mark on form
  - `are_you_still_there` — "Are you still there?" modal fires
  - `login_interrupt` — login modal appears after form
  - `second_captcha` — second CAPTCHA appears
  - `form_error_wall` — an empty field wipes the form and shows the error wall
  - `upsell_shown` — each upsell screen (with `upsell`, `upsell_index`)
  - `upsell_response` — accepted/declined (with `upsell`, `upsell_response`)
  - `white_screen` — white screen pause in processing
  - `fake_error_confirmation` — error-styled success screen
  - `pay_buttons_swapped` — when Pay/Cancel buttons are reversed

### Other Events
- `screen_view` — fires on every `navTo()` navigation (with `screen_name`)
- `newsletter_signup` — when user submits newsletter form (with `method`: "popup", "checkout", "footer")
- `filter_reset` — when event filters are cleared
- `survey_submitted` — when survey is submitted
- `footer_link_click` — when any footer link is clicked (with `link_text`)
- `of_credit_click` — when One Further credit link is clicked
- `of_cta_click` — when One Further CTA in slide-in is clicked
- `of_slidein_shown` — when post-transaction slide-in appears

### GTM Container Details
- **Container ID:** GTM-TDCTZVJ
- **GTM Account:** "One Further" (account 2795618, container 13419955)
- **GA4 Config Tag:** `google tag - G-SX7T6ML902` (pre-existing Google tag / GA4 config; reused, not modified)
- **UNPUBLISHED Workspace:** `of - ga4 event tracking`
  - Contains `ga4` folder with:
    - **DLV Variables:** `dlv - screen_name`, `dlv - pattern_name`, `dlv - link_text`, `dlv - method`, `dlv - upsell`, `dlv - upsell_response`
    - **Custom-event Triggers:** `ecommerce events`, `screen view`, `dark pattern`, `engagement events`
    - **GA4 Event Tags:** `ga4 - ecommerce events`, `ga4 - screen view`, `ga4 - dark pattern`, `ga4 - engagement events`
  - **Status:** UNPUBLISHED — reviewed in GTM Preview, then published by human

---

## Behavioural Sequences & Timing

### **Page Load → Home Screen**
1. `window.load` fires
2. Hero carousel starts blurred; `.loaded` added after 200ms, then a 14s transition into focus
3. Newsletter popup appears after 5000ms (5 seconds)
4. Cookie banner appears ~1.8 seconds after page load
5. Carousel auto-advances every 3 seconds
6. Viewer count fluctuates every 4 seconds

### **Home → Events Listing → Calendar → Seat Map**
1. Timer starts when entering seatmap (`startTimer()`)
2. Pre-selected seat D-10 is ALREADY in the basket (not just a visual)
3. Toast message appears: "You've successfully joined our mailing list!" (misleading; user may not have signed up)
4. Cookie banner shown again
5. Calendar loads in **previous month** (dark pattern)
6. Continue with D-10 still selected triggers the "Seat Taken" modal (D-10 is stripped)
7. Continue again (or first time, if D-10 was binned) checks for a stranded single seat, then proceeds to the form

### **Seat Map → Form → Upsells → Survey → Newsletter2 → Payment → Processing → Confirmation**
1. Submitting with any empty field wipes the whole form and shows the error wall
2. The first complete submit (with both CAPTCHAs) shows the "Create account to continue" modal (login interrupt)
3. After dismissing login modal, enters upsell carousel (6 screens)
4. Each upsell pre-checked; user must uncheck to skip
5. After 6 upsells, survey appears
6. After survey, newsletter re-signup appears
7. Payment screen shows computed order total (seats + booking fee × ticket count + delivery + upsells)
8. One "Are you sure?" confirmation modal
9. Processing screen for 7.5 seconds
10. White screen pause for 4 seconds (disorienting)
11. Error-styled confirmation appears (red styling, but success underneath)
12. Real confirmation screen shows
13. ~4 seconds later, post-transaction slide-in appears from right

---

## Key Functions & Logic

### State Management
- `S` object — global mutable state (top of the main `<script>`)
- `EVENTS` array — four production objects
- `UPSELLS` array — six upsell objects with icon, title, desc, price, amount, check label

### Navigation
- `navTo(screenId)` — toggles `.active` class, scrolls to top, fires GA4 `screen_view`, calls hooks for specific screens
- Screen hooks: `onEnterSeatmap()`, `onEnterForm()`, `showUpsell()`, `onEnterPayment()`, `startProcessing()`, `onEnterConfirm()`

### Seat Selection Logic
- `AVAILABLE_SEATS` object — defines which columns are available in each row
- `SEATS_PER_ROW = 18` — total columns per row
- `MAX_SEATS = 6` — maximum seats per selection
- `selectionIsolatesSeat()` — on Continue, checks whether the selection strands a single seat
- `clickSeat(id)` — handles seat click; toggles on/off; fires `add_to_cart` / `remove_from_cart`
- `removeSeat(id)` — removes from selection (bin icon in panel)
- `renderSelection()` — renders the "Your Selection" side panel from `S.selectedSeats`
- `continueSeat()` — strips D-10 with the seat-taken modal if it's still selected, else shows the isolation modal if needed, else goes to the form

### Pricing & Order Logic
- `seatPrice(row)` — returns price per row (A/B=32, C/D/E=28, F/G/H=22)
- `seatSection(row)` — returns "STALLS" or "CIRCLE"
- `seatItem(id)` — GA4 ecommerce item for a single seat
- `seatEcom(id)` — GA4 ecommerce payload for a seat
- `computeOrder()` — builds order from live seat selection + accepted upsells; calculates booking fee (£3.50 × ticket count) + delivery (£1.25) + upsells
- `renderPaymentSummary()` — renders order summary and updates "Pay £X" button text

### Timer Logic
- `startTimer()` — starts countdown from 300 seconds and adds `body.booking`
- `stopTimer()` — stops the countdown and removes `body.booking` (called on the error-styled confirmation)
- `tickTimer()` — decrements every second; extends by 60s at 180s remaining if on form; updates colour
- `updateTimerDisplay()` — updates timer bar display and colours (yellow/orange/red/flash)

### Form Logic
- `onEnterForm()` — fires GA4 `begin_checkout`, sets "are you still there?" timer (18s)
- `submitForm()` — any empty field: `clearFormFields()` wipes everything and re-ticks marketing, error wall and password rules appear; otherwise requires both CAPTCHAs, then the login interrupt, then upsells
- `tickCaptcha(n)` — first captcha triggers second captcha after 1.2s

### Payment Logic
- `onEnterPayment()` — renders order summary, applies button swap if needed
- `startPayment()` — triggers first "Are you sure?" modal, fires GA4 `add_payment_info`
- `advanceAreYouSure()` — closes the modal and goes to processing
- `startProcessing()` / `cancelProcessing()` — timers are kept in `S.procTimers` so skip and facilitator mode can cancel them

### Cookie Logic
- `buildCookieSections()` — generates 5 cookie type sections, all enabled by default
- `saveCookiePrefs()` — requires two button clicks to fully save
- `toggleCookieSection(i)` — expand/collapse section

### Upsell Logic
- `showUpsell()` — displays current upsell from `UPSELLS[S.upsellIndex]`
- `nextUpsell(respectCheckbox)` — increments index; tracks accepted/declined; loops through all 6 before moving to survey

### Footer Logic
- `buildFooterHTML()` — generates footer with 5 columns + logos + One Further credit
- `injectFooters()` — injects footer into all content screens (except processing/white-screen)
- `footerLink(text)` — fires GA4 `footer_link_click`
- `footerNewsletter(ev)` — fires GA4 `newsletter_signup` with method "footer" (duplicate, gives no feedback)

### Post-Transaction Slide-in Logic
- `onEnterConfirm()` — fires GA4 `purchase` (unless skipped), renders the reveal, sets timer for slide-in (4s delay)
- `dismissSlidein()` — closes slide-in panel (works first time, unlike everything else)

### Accessibility Widget
- `toggleA11y()` — toggles open/closed; re-opens automatically for first 3 dismissals via `S.a11yDismissCount`

---

## CSS Classes & Styling Notes

### Colour Variables (`:root`)
- `--purple: #5B2D8E` (primary brand)
- `--purple-dark: #3d1f63` (nav, headers)
- `--purple-light: #8B5CF6` (lighter variant)
- `--lavender: #e8e2f0` (light background)
- `--lavender-mid: #d4b8f0` (medium accent)
- `--grey-body: #444` (body text)
- `--grey-mid: #888` (secondary text)
- `--grey-light: #f2f1ef` (light background)
- `--grey-border: #ddd` (borders)
- `--red: #c0392b` (errors, urgency)
- `--orange: #e67e22` (warnings)
- `--yellow: #f39c12` (timeout warnings)
- `--green: #2d8a4e` (success)

### Key Styles
- **`.screen.active`** — displays screen; others hidden
- **`#timer-bar.active`** — shows timer bar
- **`#timer-bar.t-yellow/t-orange/t-red/t-flash`** — colour states for urgency
- **`.confetti-bit`** — animates falling confetti on newsletter signup
- **`.presel`** — golden outline on pre-selected seat (psychological nudge)
- **`.selected`** — green outline on user-selected seat
- **`.s-taken`** — greyed out unavailable seat
- **`.modal` classes** — centred overlays with high z-index (850–950)
- **`.seat-legend`** — shows colour bands without prices (pattern preserved)
- **`.seatmap-layout`** — two-column flex layout (left: seat map, right: panel)
- **`.selection-panel`** — sticky side panel; becomes bottom sheet under 640px

---

## Critical Patterns to Preserve

**Do NOT remove these dark patterns unless explicitly asked:**

1. ✅ Seat-taken modal fires on Continue (not seat click) when D-10 is still selected; strips it
2. ✅ Pre-selected seat D-10 arrives ALREADY in basket (user must manually bin)
3. ✅ Multi-seat selection (up to 6) with dynamic pricing per row
4. ✅ Two CAPTCHAs instead of one
5. ✅ Newsletter popup blocking access on home page
6. ✅ Calendar starting in past month
7. ✅ Marketing emails pre-checked
8. ✅ Six mandatory upsells
9. ✅ Error-styled success confirmation
10. ✅ Payment button swap (25% chance)
11. ✅ "Are you still there?" modal during form (18 seconds)
12. ✅ Login interrupt modal after form
13. ✅ Accessibility widget re-opens 3 times before staying closed
14. ✅ Cookie preferences require two saves
15. ✅ Filter spinner trap
16. ✅ "847 trusted partners" wording
17. ✅ Timer bar with artificial urgency
18. ✅ Fake "Harriet" live chat with 2–3 business day response time
19. ✅ Fake postcode lookup error
20. ✅ Seat isolation validation modal
21. ✅ Misleading newsletter toast on seatmap entry
22. ✅ Keyboard trap: Escape doesn't close newsletter
23. ✅ Global footer on every screen (over-stuffed, contradictory hours)
24. ✅ Post-transaction slide-in (works, unlike everything else)
25. ✅ Dynamic order computation (not hardcoded £64.19)
26. ✅ GA4 ecommerce tracking + dark pattern events
27. ✅ "Skip to the end" works first time (like the slide-in); it's the visitor's honest exit
28. ✅ Every dark pattern has a `PATTERNS` entry so it shows in the end-of-journey reveal

---

## Editing Guidelines

### When Polishing Visuals
- Update colours, spacing, fonts, shadows to improve professional appearance
- Refactor CSS for cleaner code
- Improve mobile responsiveness
- Enhance button hover states, transitions

### When Preserving Dark Patterns
- Keep all timing delays (`setTimeout`, `setInterval` calls)
- Keep all fake validation errors and modal interrupts
- Keep all pre-checked checkboxes and confirmshaming language
- Keep all multi-step processes (two CAPTCHAs, two cookie saves, the error-styled confirmation before the real one)
- Keep all screen sequences and upsell loops
- Keep all timing calculations (timer, delays, fluctuations)

### When Changing Content (If Asked)
- Venue name: "Enyold Theatre" (never "Harwick")
- All venue references should use "Enyold Theatre", "Enyold Membership", "Enyold box office", etc.
- Event names, dates, and prices can be updated but preserve the dark patterns around them
- Upsell copy can be refined but keep the 6-step loop and pre-checked state

### When Adding New Features
- Do NOT add real validation (keep fake errors)
- Do NOT remove interrupts or modals
- Do NOT make the site faster or less frustrating
- DO add more dark patterns if inspired (but check with user first)
- Ensure GA4 events are wired up if adding new user interactions

---

## Project Intent & Testing Use Case

This site is a **usability testing teaching tool.** When One Further runs user sessions on this site, test participants experience mounting frustration through:

1. **Unexpected modals** (newsletter, cookies, seat taken, are you still there, login)
2. **Fake errors** (form validation, postcode lookup, CAPTCHA multiplication)
3. **Psychological nudges** (pre-selected seat, pre-checked marketing, pre-checked upsells)
4. **Artificial urgency** (timer, viewer count, limited availability badges)
5. **Confusing states** (error styling for success, accessibility widget traps, white screen pause)
6. **Data extraction** (intrusive form, survey, income brackets, emergency contact)
7. **Mandatory upsells** (6-step carousel)
8. **Delayed/unclear feedback** (white screen, processing stages, 10-day wait messaging)

Participants' frustration demonstrates why UX research and testing are critical. The site is a **negative exemplar** — it shows what *not* to do.

---

## File Size & Performance Notes

- ~3300 lines in `index.html` (CSS and JavaScript inline)
- External assets: `images/og-image.png` and optional `images/stock/*.jpg` (gradient fallbacks when missing); GTM loads from Google
- Favicon is an inline SVG data URI
- Confetti is dynamically generated (no sprite sheets)
- Works fully in modern browsers (no IE support needed)

---

## Future Enhancements (If Requested)

- [ ] Add more production pages (screen-prod-5, etc.)
- [ ] Expand upsell offerings (add 7th/8th upsells)
- [ ] Add A/B testing variants (e.g., different modal order, different error messages)
- [ ] Inject real performance metrics (track time per screen, click counts)
- [ ] Add session recording hooks (for video replay of user testing)
- [ ] Implement actual "Continue as Guest" flow
- [ ] Add more realistic seat map (visual theatre layout)
- [ ] Expand terms & conditions with more legal bloat
- [ ] Add accessibility fails in CSS (low contrast, missing labels, etc.)
- [ ] Create print-friendly confirmation page
- [ ] Link blog post URL for "Read why we built this" (currently placeholder)

---

## Contact & Attribution

**Built by:** One Further (digital agency)  
**Purpose:** Usability research demonstration tool  
**Target Audience:** UX researchers, designers, product managers attending usability testing sessions  
**Current Version:** Enyold Theatre (multi-seat selection, dynamic pricing, GA4 analytics, global footer, redesigned header and home page, end-of-journey reveal, skip to the end, facilitator mode)
