# Test Plan — Swag Labs (SauceDemo) E-Commerce Platform

**Application under test:** https://www.saucedemo.com
**Author:** [Your Name]
**Date:** [Date]
**Version:** 1.0

---

## 1. Objective

Verify that the core shopping flows of Swag Labs — authentication, product browsing,
cart management, and checkout — work correctly across the four provided user roles,
and identify functional/UI defects through structured and exploratory testing.

## 2. Scope

### In scope
- **Authentication:** login, logout, session handling, all four provided accounts, invalid credentials
- **Product catalog:** inventory listing, product detail view, images, prices, sorting (Name A-Z/Z-A, Price low-high/high-low)
- **Cart:** add/remove single and multiple items, cart badge count accuracy, cart persistence across navigation
- **Checkout:** shipping info form (validation, required fields), order overview (item list, tax, total math), order completion, "Cancel" flows at each checkout step
- **Cross-role behavior:** how each of the 4 seeded users behaves differently on the same flows
- **Negative/edge cases:** empty cart checkout, invalid form input, locked-out account, back-button navigation, direct URL access without auth

### Out of scope
- Backend/API testing (no public API is exposed)
- Payment gateway integration (checkout is a simulated/mock flow, no real payment)
- Load/performance testing beyond observing `performance_glitch_user` behavior
- Accessibility (WCAG) audit — noted as a risk area but not formally scored in this cycle
- Mobile native app testing (site is responsive web only)

## 3. Types of Testing

| Type | Application in this cycle |
|---|---|
| **Functional testing** | Verify login, add-to-cart, checkout, sort, and logout work as specified for each role |
| **UI / visual testing** | Check product images, layout, button states, and badge counters render correctly, especially for `problem_user` |
| **Negative testing** | Invalid login (wrong password, empty fields, locked user), incomplete checkout form, direct navigation to protected pages while logged out |
| **Edge case testing** | Empty cart checkout, adding/removing the same item repeatedly, sorting with 0 vs many items, browser back/forward during checkout, session/cart persistence after refresh |
| **Cross-browser / cross-device testing** | Same core flows run on Chrome, Firefox, and a mobile viewport (see Test Environment) |
| **Regression-style smoke testing** | The 3 automated flows (login, checkout, locked-out) act as a repeatable smoke suite |
| **Exploratory testing** | Unscripted, time-boxed session per user role to surface non-obvious bugs (see Bug Report) |

## 4. Test Environment

| Dimension | Coverage |
|---|---|
| **Browsers (desktop)** | Google Chrome (latest), Mozilla Firefox (latest) — primary; Microsoft Edge (latest) — secondary spot-check |
| **OS** | Windows 11 and macOS (at least one manual pass on each if available) |
| **Viewport / device** | Desktop (1920×1080), laptop (1366×768), mobile emulation (390×844, iPhone-class) via browser dev tools |
| **Automation environment** | Selenium WebDriver (Python) + WebDriver Manager, headless and headed Chrome, run locally and designed to be CI-ready |
| **Network conditions** | Default; `performance_glitch_user` used as a proxy for slow-network/slow-render behavior since no real network throttling is required to trigger it |

## 5. Test Data

| Username | Password | Purpose / Expected behavior |
|---|---|---|
| `standard_user` | `secret_sauce` | Baseline "happy path" user — everything should work as designed |
| `locked_out_user` | `secret_sauce` | Must be blocked at login with a clear, specific error message |
| `problem_user` | `secret_sauce` | Intentionally broken UI/behavior — used to surface bugs |
| `performance_glitch_user` | `secret_sauce` | Intentionally slow — used to check loading states, timeouts, and whether the app remains usable/doesn't error out under delay |
| Invalid credentials | e.g. `standard_user` / `wrong_password` | Negative login test |
| Empty fields | — | Required-field validation on login and checkout |

## 6. Test Cases

> Format: ID, Title, Preconditions, Steps, Expected Result, Priority

### TC-01 — Successful login with standard_user
- **Preconditions:** Browser at saucedemo.com login page, logged out
- **Steps:**
  1. Enter `standard_user` in Username
  2. Enter `secret_sauce` in Password
  3. Click **Login**
- **Expected Result:** User is redirected to `/inventory.html`; page title reads "Products"; 6 products are visible with images, names, descriptions, and prices
- **Priority:** Critical

### TC-02 — Login blocked for locked_out_user
- **Preconditions:** Logged out
- **Steps:**
  1. Enter `locked_out_user` / `secret_sauce`
  2. Click **Login**
- **Expected Result:** User remains on the login page; a red error banner appears reading "Epic sadface: Sorry, this user has been locked out."; no session/cart is created
- **Priority:** Critical

### TC-03 — Add item to cart and complete checkout (standard_user)
- **Preconditions:** Logged in as `standard_user`
- **Steps:**
  1. Click **Add to cart** on "Sauce Labs Backpack"
  2. Verify cart badge shows "1"
  3. Click cart icon → click **Checkout**
  4. Fill First Name, Last Name, Zip/Postal Code → **Continue**
  5. Review order summary (item price, tax, total) → **Finish**
- **Expected Result:** "Thank you for your order!" confirmation page is displayed; totals on the overview page equal item price + tax exactly; cart badge resets to empty after completion
- **Priority:** Critical

### TC-04 — Checkout form validation (missing required fields)
- **Preconditions:** Logged in as `standard_user`, 1 item in cart, on checkout Step One (Your Information)
- **Steps:**
  1. Leave Last Name and Zip Code blank
  2. Click **Continue**
- **Expected Result:** User stays on the same step; an inline error message specifies the missing field (e.g. "Error: Last Name is required"); no navigation occurs
- **Priority:** High

### TC-05 — Cart badge accuracy across add/remove actions
- **Preconditions:** Logged in as `standard_user`
- **Steps:**
  1. Add 3 different items to cart from the inventory page
  2. Remove 1 item from the inventory page (button should now read "Remove")
  3. Open the cart page and remove a 2nd item from there
- **Expected Result:** Badge count decreases by exactly 1 after each removal (3 → 2 → 1) and always matches the number of line items actually present in the cart page
- **Priority:** High

### TC-06 — Product sort behaves correctly (Price low→high / high→low, Name A→Z / Z→A)
- **Preconditions:** Logged in as `standard_user`, on inventory page
- **Steps:**
  1. Select "Price (low to high)" from the sort dropdown; record the displayed price sequence
  2. Select "Price (high to low)"; record the sequence
  3. Repeat for Name A→Z and Z→A
- **Expected Result:** Each sort mode produces a list that is fully and correctly ordered by the selected criterion, with no items out of place
- **Priority:** Medium

### TC-07 — problem_user exploratory pass (image and interaction integrity)
- **Preconditions:** Logged in as `problem_user`
- **Steps:**
  1. Visually compare each product's image against its name/description
  2. Attempt to add each of the 6 items to the cart individually and verify the badge and button state after each
  3. Attempt to fill out the checkout form (First Name, Last Name, Zip)
- **Expected Result:** Images correctly match their products; every "Add to cart" click updates the badge by exactly 1 and the button correctly toggles to "Remove"; all three checkout fields accept and retain typed text
- **Priority:** High (this is the primary bug-surfacing case)

### TC-08 — performance_glitch_user load-time and usability check
- **Preconditions:** Logged out
- **Steps:**
  1. Log in as `performance_glitch_user` and start a timer
  2. Wait for the inventory page to become interactive
  3. Attempt to add an item to cart as soon as the page appears usable
- **Expected Result:** Page eventually loads fully (document what the delay actually is); no JS errors in console during the delay; once loaded, add-to-cart works normally with no double-submission or stuck loading state
- **Priority:** Medium

### TC-09 — Direct URL access without authentication
- **Preconditions:** Logged out, no active session
- **Steps:**
  1. Navigate directly to `https://www.saucedemo.com/inventory.html` (or `/cart.html`, `/checkout-step-one.html`) without logging in
- **Expected Result:** User is redirected to the login page with an explanatory error ("You can only access '...' when you are logged in."), not shown a blank or broken page
- **Priority:** High (security/session-handling risk)

### TC-10 — Logout clears session and cart state
- **Preconditions:** Logged in as `standard_user` with 2 items in cart
- **Steps:**
  1. Open the burger menu → click **Logout**
  2. Log back in as `standard_user`
- **Expected Result:** After logout, user lands on the login page and back/forward navigation does not expose the inventory page; after logging back in, confirm and document whether the cart persisted (this is a common ambiguous/edge behavior worth flagging either way)
- **Priority:** Medium

## 7. Risk Assessment

| Area | Likelihood of defects | Impact if broken | Rationale |
|---|---|---|---|
| `problem_user` product images/interactions | Very High | Medium | Explicitly seeded as the "broken" account; highest bug density expected here |
| Cart badge / state sync | High | High | Small counters like this are easy to desync from actual cart contents and directly affect user trust in the checkout total |
| Checkout form validation | Medium | High | Any bypass of required-field validation could let malformed orders through |
| Sort functionality | Medium | Low–Medium | Cosmetic/ordering bug, unlikely to block a purchase but damages perceived quality |
| `performance_glitch_user` load handling | Medium | Medium | Could mask real errors as "still loading," or produce race conditions (e.g., double-clicks during slow load) |
| Session/auth boundary (direct URL access) | Low–Medium | High | If unauthenticated users can reach protected pages, that's a security-relevant defect, not just cosmetic |
| Cross-browser rendering | Low | Medium | Site is simple and mostly static; risk is concentrated in flexbox/grid layout edge cases at small viewports |
| Logout/cart persistence semantics | Low | Low | More of an ambiguous-requirement risk than a hard defect |

## 8. Entry / Exit Criteria

**Entry:** Site is reachable, all 4 test accounts authenticate (or correctly fail, in the locked-out case), and the automation environment (Python + Selenium + ChromeDriver) is installed and passing a smoke check.

**Exit:** All test cases in Section 6 executed on at least Chrome + Firefox; all Critical/High priority cases pass or have a filed, triaged bug; automated suite (3 flows) is green in the target environment; Bug Report delivered with ≥5 reproducible, non-duplicate defects.

## 9. Deliverables Produced From This Plan

1. This test plan
2. `BUG_REPORT.md` — defects found via the exploratory cases above (TC-07 in particular)
3. Automated Selenium/PyTest suite covering TC-01, TC-02, and TC-03
