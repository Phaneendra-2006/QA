# Bug Report — Swag Labs (SauceDemo)

**Application:** https://www.saucedemo.com
**Tester:** [Your Name]
**Date:** [Date]
**Build/Environment:** Chrome (latest), desktop, https://www.saucedemo.com

> **Verification note (read before recording Video 2):** Each bug below was identified through
> targeted exploratory testing of SauceDemo's known "problem" accounts and known trouble spots
> (sorting, cart-badge sync, checkout validation, footer links, rapid-click races). **Re-run every
> repro exactly once yourself immediately before filming**, since SauceDemo's seeded bugs have
> changed slightly between versions historically, and screenshots/timings below are placeholders
> for you to fill in with your own evidence. Delete any bug you can't reproduce and replace it —
> don't present an unverified bug as confirmed.

---

## BUG-01 — Sort "Price (low to high)" / "Price (high to low)" does not actually reorder items for `problem_user`

- **Severity:** High &nbsp;|&nbsp; **Priority:** High
- **User/Account:** `problem_user`
- **Environment:** Chrome, desktop, inventory page (`/inventory.html`)
- **Preconditions:** Logged in as `problem_user`

**Steps to Reproduce:**
1. Log in as `problem_user` / `secret_sauce`
2. On the Products page, open the sort dropdown (top right, default "Name (A to Z)")
3. Select **"Price (low to high)"**
4. Read the displayed price of each product top-to-bottom

**Expected Result:** Products re-render in strictly ascending price order.

**Actual Result:** The dropdown value changes but the on-screen product order does not change
to match — items remain in (or revert to) an order inconsistent with ascending price. The same
mismatch occurs for "Price (high to low)."

**Why it's easy to miss:** The page doesn't error or flash — it looks like a normal, populated
list. You only catch this by actually reading and comparing the price column against the
selected sort mode, not just glancing at the layout.

**Impact:** Customers trying to shop by budget (e.g., "show me the cheapest item") get a
misleading list, which can lead to purchase decisions based on wrong price ordering.

**Evidence:** [screenshot: sort dropdown = "Price (low to high)" next to the resulting price list]

---

## BUG-02 — Cart badge count desyncs from actual cart contents when removing an item from the Cart page (`problem_user`)

- **Severity:** High &nbsp;|&nbsp; **Priority:** High
- **User/Account:** `problem_user`
- **Environment:** Chrome, desktop, cart page (`/cart.html`)
- **Preconditions:** Logged in as `problem_user`, at least 2 items already added to cart

**Steps to Reproduce:**
1. Add 2–3 items to the cart from the inventory page (badge updates correctly here)
2. Click the cart icon to open `/cart.html`
3. Click **Remove** on one line item
4. Observe the cart badge count in the top-right, and cross-check it against the number of
   line items actually still listed on the cart page

**Expected Result:** Badge count decreases by exactly 1 and matches the number of line items
remaining on the page.

**Actual Result:** The badge count does not update to match the actual remaining items (it either
stays the same or updates inconsistently relative to what's still listed), so the badge and the
real cart contents disagree.

**Why it's easy to miss:** You have to deliberately count the visible line items and compare
them to the badge number — most testers only check "does clicking Remove make the row disappear,"
not whether the counter is still telling the truth afterward.

**Impact:** Directly undermines trust in the cart — a customer could believe they still have an
item in cart (or don't) and be surprised at checkout.

**Evidence:** [screenshot: cart page showing N line items next to badge showing a different number]

---

## BUG-03 — Checkout "Last Name" / "Zip/Postal Code" fields silently reject keyboard input for `problem_user`

- **Severity:** Critical &nbsp;|&nbsp; **Priority:** Critical
- **User/Account:** `problem_user`
- **Environment:** Chrome, desktop, checkout Step One (`/checkout-step-one.html`)
- **Preconditions:** Logged in as `problem_user`, 1+ item in cart, on the "Checkout: Your Information" step

**Steps to Reproduce:**
1. Click into the **First Name** field and type — confirm text appears normally
2. Click into the **Last Name** field and type any value
3. Click into the **Zip/Postal Code** field and type any value
4. Click **Continue**

**Expected Result:** All three fields accept and display typed text, and a fully filled form
proceeds to the order overview step.

**Actual Result:** First Name accepts input normally, but Last Name and/or Zip/Postal Code do
not visibly retain typed characters, so the form cannot be completed as a `problem_user` — the
checkout funnel is effectively blocked for this account even though the user has a non-empty cart.

**Why it's easy to miss:** The fields look identical to a working text input (same border,
cursor blinks, no error is thrown) — the only tell is that the characters you type don't appear,
which is easy to dismiss as "I must have mis-clicked" on a first pass.

**Impact:** Complete checkout funnel blocker for this account — highest-severity bug in this
report because it prevents order completion entirely, with no user-facing error explaining why.

**Evidence:** [screenshot or short clip: typing into Last Name field, field remains visually empty]

---

## BUG-04 — Footer social media icons link to broken/placeholder destinations (all users)

- **Severity:** Low &nbsp;|&nbsp; **Priority:** Low
- **User/Account:** All (`standard_user` used for verification)
- **Environment:** Chrome, desktop, any page with the site footer (e.g. `/inventory.html`)
- **Preconditions:** Logged in as `standard_user`

**Steps to Reproduce:**
1. Scroll to the footer on any logged-in page
2. Note the Twitter/X, Facebook, and LinkedIn icons
3. Click each one (or right-click → open in new tab) and observe the destination

**Expected Result:** Each icon opens the corresponding real, working social profile for the
Sauce Labs / Swag Labs brand.

**Actual Result:** At least one of the icons resolves to a broken link, a generic/placeholder
page, or a destination unrelated to an active Swag Labs profile.

**Why it's easy to miss:** Footer links are low-traffic UI that most functional test passes
skip entirely since they're not part of the "critical path" (login → cart → checkout).

**Impact:** Low functional impact, but a broken outbound link on a live storefront reads as
unpolished/untrustworthy branding, and is worth flagging even though it won't block a sale.

**Evidence:** [screenshot: broken/placeholder destination page, with the source footer link visible]

---

## BUG-05 — Rapid double-click on "Add to cart" can add an item twice while the badge still shows 1 (race condition)

- **Severity:** Medium &nbsp;|&nbsp; **Priority:** Medium
- **User/Account:** `standard_user` (baseline account, so this is a real app-level race condition, not an intentionally seeded "problem" bug)
- **Environment:** Chrome, desktop, inventory page
- **Preconditions:** Logged in as `standard_user`, empty cart

**Steps to Reproduce:**
1. Position the mouse over "Add to cart" for any single product
2. Double-click as fast as possible (fast enough that the button hasn't yet re-rendered to
   "Remove" between clicks) — this is easiest to trigger with a scripted double-click via
   DevTools console or Selenium `ActionChains` rather than a literal human double-click
3. Observe the cart badge and then open the cart page to count actual line items for that product

**Expected Result:** The second click is a no-op (or the button is already "Remove" and toggles
the item back out), and the cart ends up with exactly one unit of the item.

**Actual Result:** Under fast enough double-click timing, the click handler can fire twice before
the button's label/state updates, and the cart page can end up reflecting the item having been
added in a way inconsistent with a single badge increment — i.e., the visible badge and the
true cart state can disagree after the race.

**Why it's easy to miss:** This only surfaces under click timing faster than a human normally
clicks — it's the kind of bug that a scripted/automated interaction (or a very fast, deliberate
double-click) finds far more reliably than manual exploratory clicking at a normal pace.

**Impact:** Low likelihood in real-world usage, but represents a missing debounce/guard on a
state-mutating button, which is worth flagging as a code-quality/robustness issue.

**Evidence:** [screen recording at reduced playback speed showing both clicks and the resulting badge/cart mismatch]

---

## BUG-06 — Add-to-cart click during `performance_glitch_user`'s load delay produces an inconsistent loading/click state

- **Severity:** Medium &nbsp;|&nbsp; **Priority:** Medium
- **User/Account:** `performance_glitch_user`
- **Environment:** Chrome, desktop, login → inventory transition
- **Preconditions:** Logged out

**Steps to Reproduce:**
1. Log in as `performance_glitch_user` / `secret_sauce` and immediately start a stopwatch
2. As soon as any part of the inventory page becomes visible/clickable (even before it looks
   fully settled), click **Add to cart** on the first product you can interact with
3. Note whether the click registers immediately, is delayed, or needs to be repeated

**Expected Result:** Either the page fully blocks interaction until it's safely ready (e.g. a
visible loading state), or it's ready and a single click reliably registers once.

**Actual Result:** There's a window where the page appears interactive but a click doesn't
reliably produce an immediate, correct badge update — the user is left unsure whether the click
"took," which invites a repeat click and the same double-add risk as BUG-05.

**Why it's easy to miss:** Most testers either wait for the page to fully finish loading before
interacting (avoiding the window entirely) or don't test this account beyond confirming "it's
slow," without probing what happens if a real, impatient user clicks early.

**Impact:** Compounds with BUG-05's race condition risk specifically for the one account whose
entire purpose is simulating a slow, frustrating experience — exactly the scenario where a user
is most likely to click impatiently.

**Evidence:** [screen recording with visible timer overlay from login click to first successful add-to-cart]

---

## Summary Table

| ID | Title | Severity | Account |
|---|---|---|---|
| BUG-01 | Price sort doesn't reorder items | High | problem_user |
| BUG-02 | Cart badge desyncs after removing from cart page | High | problem_user |
| BUG-03 | Last Name / Zip fields reject input at checkout | Critical | problem_user |
| BUG-04 | Footer social links broken/placeholder | Low | all |
| BUG-05 | Rapid double-click adds item twice (race condition) | Medium | standard_user |
| BUG-06 | Inconsistent state on early click during glitch-user load | Medium | performance_glitch_user |
