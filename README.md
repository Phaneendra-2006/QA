# SauceDemo QA Assignment

QA + automation deliverables for the Swag Labs (https://www.saucedemo.com) offline interview
assignment.

## Repo structure

```
saucedemo-qa-assignment/
├── README.md                      <- you are here
├── test-plan/
│   └── TEST_PLAN.md                <- scope, test types, environment, test data, 10 test cases, risk assessment
├── bug-report/
│   └── BUG_REPORT.md               <- 6 non-obvious bugs, with repro steps, severity, and impact
└── automation/
    ├── requirements.txt
    ├── pytest.ini
    ├── conftest.py                 <- WebDriver fixtures (webdriver-manager, headless toggle)
    ├── pages/                      <- Page Object Model
    │   ├── base_page.py
    │   ├── login_page.py
    │   ├── inventory_page.py
    │   └── cart_and_checkout_page.py
    └── tests/
        ├── test_login.py           <- valid login + invalid password
        ├── test_checkout_flow.py   <- add to cart -> full checkout, plus a validation negative case
        └── test_locked_out_user.py <- locked_out_user error + direct-URL bypass attempt
```

## Design notes / why it's built this way

- **Page Object Model**, not scripts full of raw `driver.find_element` calls. Each page (login,
  inventory, cart, checkout step 1/2, checkout complete) is its own class in `pages/`, so a
  selector change only needs to be fixed in one place.
- **Selectors use SauceDemo's `data-test` attributes** wherever available (e.g.
  `[data-test='username']`) instead of CSS classes or link text, since `data-test` attributes are
  put there specifically for automation and are far less likely to change than styling classes.
- **No `time.sleep()` anywhere.** All waits go through `BasePage`'s `WebDriverWait` helpers
  (`find`, `find_clickable`, `is_visible`), so the suite is resilient to normal render timing.
- **Assertions check both UI state and business logic** — e.g. `test_checkout_flow.py` doesn't
  just check that the confirmation page loads, it also asserts `item_total + tax == total`, which
  is the kind of check that would have caught a real rounding/math bug if one existed.
- Tests are grouped by user flow, matching the 3 flows required by the assignment:
  1. `test_login.py::test_login_with_valid_credentials`
  2. `test_checkout_flow.py::test_add_to_cart_and_complete_checkout`
  3. `test_locked_out_user.py::test_locked_out_user_cannot_log_in`
  Extra tests (invalid password, missing-last-name validation, direct-URL bypass) are included
  as a bonus but are not required to satisfy the 3-flow minimum.

## Prerequisites

- Python 3.9+
- Google Chrome installed locally (the suite uses `webdriver-manager` to auto-download a matching
  `chromedriver`, so you do **not** need to install ChromeDriver manually)

## Setup

```bash
cd automation
python3 -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## Running the tests

Run the whole suite, headed (a visible Chrome window):

```bash
pytest
```

Run headless (e.g. for CI):

```bash
HEADLESS=1 pytest
```

Run only the critical-path smoke tests (the 3 required flows):

```bash
pytest -m critical
```

Run a single file:

```bash
pytest tests/test_checkout_flow.py -v
```

## Known limitations / what I'd add with more time

- No parallelization (`pytest-xdist`) — fine for 5 tests, would matter at scale.
- No CI config included (e.g. GitHub Actions) — straightforward to add given the `HEADLESS=1`
  toggle already in `conftest.py`.
- `problem_user` and `performance_glitch_user` are exercised manually (see `bug-report/`) rather
  than automated, since their failures are exactly the kind of unscripted, "does this look right"
  defects exploratory testing is better suited to catch than a fixed assertion.
- Cross-browser runs (Firefox) are described in the test plan but only implemented for Chrome
  here; swapping in `webdriver-manager`'s Firefox support would be a small, mechanical change to
  `conftest.py`.

## Related deliverables

- Test plan: [`test-plan/TEST_PLAN.md`](test-plan/TEST_PLAN.md)
- Bug report: [`bug-report/BUG_REPORT.md`](bug-report/BUG_REPORT.md)
- Video 1 (test plan & automation walkthrough): **[add Loom link]**
- Video 2 (manual bug discovery demo): **[add Loom link]**
