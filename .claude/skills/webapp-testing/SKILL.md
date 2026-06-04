---
name: webapp-testing
description: Toolkit for interacting with and testing local web applications using Playwright. Supports verifying frontend functionality, debugging UI behavior, capturing browser screenshots, and viewing browser logs.
license: Complete terms in LICENSE.txt
---

# Web Application Testing

To test local web applications, write native Python Playwright scripts.

**Helper Scripts:** The `with_server.py` script that manages server lifecycle is located in the sitebuilder project at:
`C:\Users\viren\source\sitebuilder\website-engine\skills\webapp-testing\scripts\with_server.py`

Run it with `--help` first to see usage. Do not read the source unless absolutely necessary — use it as a black box.

## Decision Tree: Choosing Your Approach

```
User task → Is it static HTML?
    ├─ Yes → Read HTML file directly to identify selectors
    │         ├─ Success → Write Playwright script using selectors
    │         └─ Fails/Incomplete → Treat as dynamic (below)
    │
    └─ No (dynamic webapp) → Is the server already running?
        ├─ No → Run: python <sitebuilder-path>/scripts/with_server.py --help
        │        Then use the helper + write simplified Playwright script
        │
        └─ Yes → Reconnaissance-then-action:
            1. Navigate and wait for networkidle
            2. Take screenshot or inspect DOM
            3. Identify selectors from rendered state
            4. Execute actions with discovered selectors
```

## Example: Using with_server.py

**Multiple servers (backend + frontend):**
```bash
python "C:\Users\viren\source\sitebuilder\website-engine\skills\webapp-testing\scripts\with_server.py" \
  --server "cd backend && python -m uvicorn main:app --port 8000" --port 8000 \
  --server "cd frontend && npm run dev" --port 5173 \
  -- python your_automation.py
```

To create an automation script, include only Playwright logic (servers are managed automatically):
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto('http://localhost:5173')
    page.wait_for_load_state('networkidle')  # CRITICAL: Wait for JS to execute
    # ... your automation logic
    browser.close()
```

## Reconnaissance-Then-Action Pattern

1. **Inspect rendered DOM**:
   ```python
   page.screenshot(path='/tmp/inspect.png', full_page=True)
   content = page.content()
   page.locator('button').all()
   ```

2. **Identify selectors** from inspection results

3. **Execute actions** using discovered selectors

## Common Pitfall

Do NOT inspect the DOM before waiting for `networkidle` on dynamic apps. Always call `page.wait_for_load_state('networkidle')` first.

## Best Practices

- Use `sync_playwright()` for synchronous scripts
- Always close the browser when done
- Use descriptive selectors: `text=`, `role=`, CSS selectors, or IDs
- Add appropriate waits: `page.wait_for_selector()` or `page.wait_for_timeout()`
- Screenshot first, then act — never guess at selectors in a dynamic app

## Golden Path to Test for the Booking Platform

Run these after each phase:

1. **Phase 3 (public pages):** Landing → click "Browse retreats" → see retreat card → click → see detail page
2. **Phase 4 (quiz + consultation):** Take dosha quiz → see recommendations → submit consultation form
3. **Phase 5 (booking):** Login → select dates → fill health intake → reach Stripe checkout
4. **Phase 7 (center admin):** Login as center_admin → see own retreats → not see other centers' data
5. **Phase 8 (platform admin):** Login as platform_admin → see pending centers → approve one
