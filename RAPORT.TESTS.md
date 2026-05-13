# Test Failures Analysis & Fix Instructions

**Date:** 2025-12-03
**Status:** 5 FAIL ❌, 4 PASS ✅
**Test Suite:** Playwright E2E Smoke Tests

---

## Executive Summary

**Root Cause:** Hugo multilingual configuration workaround is incomplete. While Polish content loads correctly, the site still uses `lang="en"` in HTML and loads English menu configuration instead of Polish.

**Current Workaround Issues:**
- `defaultContentLanguage = "en"` (should be "pl")
- `[en]` language config used instead of `[pl]` in `languages.toml`
- This causes Hugo to generate HTML with `lang="en"` attribute
- Menu loads from `menus.en.toml` instead of `menus.pl.toml`

**Impact:** 5 out of 9 tests failing, site works but has incorrect metadata and missing features.

---

## 🧪 Detailed Test Analysis

### ❌ FAILED TESTS (5)

#### 1. Homepage Title Mismatch
**Test:** `tests/e2e/smoke/homepage-content.spec.js:4` - "homepage loads correctly"
**File:** `tests/e2e/smoke/homepage-content.spec.js:8`

**Error:**
```
Expected: /Inteligentny Dom/
Received: "Inteligentne Mieszkanie"
```

**Root Cause:**
Homepage title comes from SEO metadata in `content/_index.md:14`:
```yaml
seo:
  title: Inteligentne Mieszkanie  # ← This overrides page title
title: Inteligentny Dom           # ← Test expects this
```

**Fix Options:**

**Option A: Update SEO title** (RECOMMENDED - 30 seconds)
```yaml
# File: content/_index.md
seo:
  title: Inteligentny Dom  # ← Change this line
```

**Option B: Update test expectations** (NOT RECOMMENDED)
```javascript
// Change test to accept either title
await expect(page).toHaveTitle(/Inteligentny Dom|Inteligentne Mieszkanie/);
```

**Difficulty:** 🟢 TRIVIAL
**Priority:** LOW (cosmetic issue)

---

#### 2. Latest Posts Section Missing
**Test:** `tests/e2e/smoke/homepage-content.spec.js:30` - "homepage has latest posts section"
**File:** `tests/e2e/smoke/homepage-content.spec.js:34-35`

**Error:**
```
Expected: locator('.latest-posts, .recent-posts') to be visible
Received: element(s) not found
```

**Root Cause:**
The "Latest Posts" partial renders using class `.section-related`, NOT `.latest-posts` or `.recent-posts`.

**Template Chain:**
1. `layouts/index.html:70` → `{{ partial "sidebar/latest-posts" . }}`
2. `layouts/partials/sidebar/latest-posts.html:11` → `{{ partial "sidebar/posts-section" ... }}`
3. `layouts/partials/sidebar/posts-section.html:14` → `<section class="section section-related container">`

**Fix Options:**

**Option A: Add class to template** (RECOMMENDED - 1 minute)
```html
<!-- File: layouts/partials/sidebar/posts-section.html:14 -->
<section class="section section-related latest-posts container">
  <!-- ↑ Add "latest-posts" class here -->
```

**Option B: Update test selector** (ACCEPTABLE - 30 seconds)
```javascript
// File: tests/e2e/smoke/homepage-content.spec.js:34
const latestPosts = page.locator('.section-related');
```

**Difficulty:** 🟢 TRIVIAL
**Priority:** MEDIUM (feature visibility)

---

#### 3. HTML lang Attribute is "en" Instead of "pl"
**Test:** `tests/e2e/smoke/multilingual-config.spec.js:4` - "site uses Polish as default language"
**File:** `tests/e2e/smoke/multilingual-config.spec.js:9`

**Error:**
```
Expected: lang attribute matching /pl/i
Received: "en"
```

**Root Cause:**
Hugo generates `<html lang="...">` based on `defaultContentLanguage` setting:

```toml
# File: config/_default/hugo.toml:22
defaultContentLanguage = "en"  # ← Workaround for Hugo bug
```

**Why This Exists:**
Hugo 0.150.1 has a breaking change where it defaults to "en" alphabetically when multiple i18n files exist, ignoring `defaultContentLanguage = "pl"`. The workaround was to accept this and set everything to "en" even though content is Polish.

**Fix Options:**

**Option A: Override in HTML template** (QUICK FIX - 2 minutes)
```html
<!-- File: layouts/_default/baseof.html (or create custom head partial) -->
<html lang="pl" ...>  <!-- ← Override Hugo's default -->
```

**Option B: Permanent fix via i18n migration** (PROPER SOLUTION - see task 005)
- Change `defaultContentLanguage = "pl"`
- Change `defaultContentLanguageInSubdir = true`
- All URLs get `/pl/` prefix
- Requires 301 redirects for 14 indexed pages

**Difficulty:** 🟡 MODERATE (Option A) / 🔴 COMPLEX (Option B)
**Priority:** HIGH (SEO impact - search engines see wrong language)

---

#### 4. Menu Missing Polish Labels
**Test:** `tests/e2e/smoke/multilingual-config.spec.js:19` - "Polish menu items match language"
**File:** `tests/e2e/smoke/multilingual-config.spec.js:24`

**Error:**
```
Expected: nav.navbar to contain text matching /Teoria|Poradniki|Usługi/
Received: element not found (nav.navbar doesn't exist)
```

**Root Cause:**
Hugo loads menu from `menus.en.toml` because `defaultContentLanguage = "en"`. However, BOTH `menus.en.toml` and `menus.pl.toml` have identical Polish labels!

**Evidence:**
```toml
# config/_default/menus/menus.en.toml:34-52
[[main]]
  name = "Teoria"       # ← Polish label in EN file!
  url = "/docs/systems/inteligentny-dom/"

[[main]]
  name = "Blog"
  url = "/blog/"

[[main]]
  name = "Poradniki"    # ← Polish label in EN file!
  url = "/tutorials/"

[[main]]
  name = "Usługi"       # ← Polish label in EN file!
  url = "/services/"
```

**Additional Issue:**
The test looks for `nav.navbar` but the actual navbar might use different classes. Need to inspect rendered HTML.

**Fix Options:**

**Option A: Fix test selector** (DIAGNOSTIC - 1 minute)
```bash
# First, check what navbar class is actually used:
pnpm run dev
# Open http://localhost:1313
# Inspect navbar element, find actual class
```

**Option B: Update menu config** (if navbar renders correctly)
- Menu labels are already in Polish in BOTH files
- Test selector might be wrong
- Check actual HTML output

**Difficulty:** 🟡 MODERATE (requires inspection)
**Priority:** HIGH (navigation is critical)

**QUESTION FOR PAWEŁ:**
❓ Can you open http://localhost:1313 in dev mode and:
1. Right-click the navigation menu
2. Select "Inspect Element"
3. Send screenshot or copy the HTML of the `<nav>` element?

This will help determine if the menu is rendering at all, or if it's just hidden/using different classes.

---

#### 5. Menu Item "Teoria" Not Found
**Test:** `tests/e2e/smoke/navigation-menu.spec.js:4` - "main menu has all 4 required items"
**File:** `tests/e2e/smoke/navigation-menu.spec.js:11`

**Error:**
```
Expected: menu item "Teoria" to be visible
Received: element(s) not found
```

**Root Cause:**
**Same as #4** - Either navbar doesn't render, or uses different CSS selector.

**Related Evidence:**
Test #3 in same file ("clicking menu items navigates") PASSES, which suggests menu IS clickable. This is contradictory!

**Possible Explanation:**
Test #3 (line 32-49) navigates successfully, suggesting menu EXISTS but:
- Test #1 uses `.nav-item` class (line 17) which might not exist
- Test #1 uses `:has-text()` selector which might be case-sensitive or whitespace-sensitive
- Navbar might use different structure than expected

**Fix Options:**

**Option A: Inspect and fix selectors** (RECOMMENDED - 5 minutes)
```javascript
// Current test (FAILING):
const menu = page.locator('nav.navbar');
await expect(menu.locator('a:has-text("Teoria")')).toBeVisible();

// Try alternative selectors:
const menu = page.locator('nav');  // Drop .navbar requirement
await expect(menu.locator('a', { hasText: 'Teoria' })).toBeVisible();
```

**Option B: Debug test with screenshots** (DIAGNOSTIC)
```javascript
// Add to test file:
await page.screenshot({ path: 'debug-navbar.png' });
console.log(await page.locator('nav').innerHTML());
```

**Difficulty:** 🟡 MODERATE
**Priority:** HIGH (blocks understanding of menu rendering)

**QUESTION FOR PAWEŁ:**
❓ Same as #4 - need HTML inspection of navbar to diagnose.

---

### ✅ PASSING TESTS (4)

1. ✅ **Homepage shows Polish content** - 6 feature boxes render correctly
2. ✅ **contentDir configuration correct** - Blog posts load from `/content/blog/`
3. ✅ **Teoria link URL correct** - Points to `/docs/systems/inteligentny-dom/`
4. ✅ **Menu navigation works** - Clicking navigates successfully

**Insight:** Tests #3 and #4 contradict each other:
- Test says "clicking menu works" (PASS)
- Test says "menu items not found" (FAIL)

This suggests:
- Menu EXISTS and is CLICKABLE
- But uses DIFFERENT CSS classes than expected
- Or has different text/whitespace

---

## 📋 Recommended Fix Priority

### Quick Wins (< 5 minutes total)

1. **Fix Test #1 (Homepage Title)** - 30 seconds
   - Edit `content/_index.md:14` → change SEO title to "Inteligentny Dom"

2. **Fix Test #2 (Latest Posts Class)** - 1 minute
   - Edit `layouts/partials/sidebar/posts-section.html:14`
   - Add `latest-posts` class to `<section>`

### Medium Priority (Requires Investigation)

3. **Debug Tests #4 & #5 (Menu Issues)** - 5-10 minutes
   - Run dev server: `pnpm run dev`
   - Inspect navbar HTML in browser
   - Update test selectors OR fix menu rendering
   - **Need Paweł's input:** Screenshot of navbar HTML

### Long-term Fix (Requires Planning)

4. **Fix Test #3 (HTML lang="pl")** - See `.claude/tasks/005-I18N_URL_MIGRATION.md`
   - **Quick workaround:** Override in baseof.html template (2 min)
   - **Permanent fix:** Execute i18n migration (2-3 days, blocked by content audit)

---

## 🚀 Action Plan for Mid-Level Developer

### Step 1: Quick Fixes (DO NOW - 5 minutes)

```bash
# 1. Fix homepage title
# Edit: content/_index.md
# Line 14: Change "Inteligentne Mieszkanie" → "Inteligentny Dom"

# 2. Fix latest posts class
# Edit: layouts/partials/sidebar/posts-section.html
# Line 14: Add "latest-posts" to class attribute

# 3. Run tests to verify
pnpm run test:smoke
```

**Expected Result:** 7 PASS ✅, 2 FAIL ❌ (only menu tests failing)

### Step 2: Debug Menu Issues (INVESTIGATE - 10 minutes)

```bash
# Start dev server
pnpm run dev

# Open browser: http://localhost:1313
# Right-click navigation → Inspect Element
# Copy HTML structure of <nav> element

# Add debug screenshot to test:
# Edit: tests/e2e/smoke/navigation-menu.spec.js
# Add after line 7:
await page.screenshot({ path: 'test-results/debug-navbar.png' });
const navHTML = await page.locator('nav').innerHTML();
console.log('Navbar HTML:', navHTML);

# Run test again
pnpm run test:smoke
```

**Expected Output:**
- Screenshot in `test-results/debug-navbar.png`
- Console log showing navbar HTML
- This reveals actual CSS classes and structure

### Step 3: Fix Menu Tests (BASED ON FINDINGS - 5 minutes)

**If navbar uses different classes:**
```javascript
// Update test selectors to match actual HTML
// Likely fix in tests/e2e/smoke/navigation-menu.spec.js:7
const menu = page.locator('nav.actual-class-here');
```

**If menu doesn't render at all:**
- Check theme configuration
- Verify menu config is loaded
- Ask Paweł for help

### Step 4: Long-term HTML lang Fix (OPTIONAL - 2 minutes)

**Quick workaround only:**
```bash
# Check if custom baseof.html exists
ls layouts/_default/baseof.html

# If not, copy from theme and modify
# Find theme baseof:
find node_modules/@hyas -name "baseof.html"

# Copy to project:
mkdir -p layouts/_default
cp node_modules/@hyas/*/layouts/_default/baseof.html layouts/_default/

# Edit: layouts/_default/baseof.html
# Change: <html lang="{{ site.Language.LanguageCode }}" ...>
# To:     <html lang="pl" ...>
```

**Note:** This is a TEMPORARY FIX. Permanent fix requires i18n migration (task 005).

---

## ❓ Questions for Paweł

### 1. Homepage Title (Test #1)
**Question:** Should the homepage SEO title be "Inteligentny Dom" or "Inteligentne Mieszkanie"?

**Context:**
- `content/_index.md` has both:
  - `title: Inteligentny Dom` (displayed as H1)
  - `seo.title: Inteligentne Mieszkanie` (used in `<title>` tag)
- Test expects "Inteligentny Dom" in browser title bar
- Currently shows "Inteligentne Mieszkanie"

**Recommendation:** Use "Inteligentny Dom" for both (consistency + SEO).

### 2. Navigation Menu Rendering (Tests #4 & #5)
**Question:** Can you inspect the navbar HTML and send a screenshot?

**Steps:**
1. Run: `pnpm run dev`
2. Open: http://localhost:1313
3. Right-click on navigation menu → "Inspect Element"
4. Send screenshot of HTML showing `<nav>` element

**Why:** Tests are contradictory - menu is clickable (test passes) but elements "not found" (test fails). Need to see actual HTML structure to fix test selectors.

### 3. HTML lang Attribute (Test #3)
**Question:** Should we do quick workaround or wait for full i18n migration?

**Options:**
- **Option A:** Quick fix - override `lang="pl"` in template (2 min)
  - Pros: Fixes test immediately, better for SEO
  - Cons: Temporary hack, still not proper multilingual setup
- **Option B:** Wait for task 005 (i18n migration)
  - Pros: Permanent solution, proper multilingual
  - Cons: Blocked by content audit, 2-3 weeks away

**Recommendation:** Do Option A now (2 min fix), then Option B later when ready for migration.

---

## 📊 Success Criteria

### After Quick Fixes (Step 1):
- ✅ 7 tests passing
- ❌ 2 tests failing (menu only)

### After Menu Debug (Steps 2-3):
- ✅ 9 tests passing (if selectors are the issue)
- OR clear diagnosis of menu rendering problem

### After HTML lang Fix (Step 4):
- ✅ All 9 tests passing
- 🎯 Site ready for production

---

## 🔗 Related Documentation

- **Migration Plan:** `.claude/tasks/005-I18N_URL_MIGRATION.md`
- **Test Spec:** `.claude/specs/002-smoke-tests.md`
- **ADR (Multilingual):** `.claude/adrs/001-multilingual-url-structure.md`
- **Project Status:** `STATUS.md`

---

## 📝 Notes

**Why Tests Contradict:**
- Test #3 (navigation clicks) uses direct `page.click('a:has-text("Blog")')` - works
- Test #1 (menu items) uses `menu.locator('a:has-text("Teoria")').toBeVisible()` - fails

**Hypothesis:** Menu EXISTS but `nav.navbar` selector is wrong, so:
- Direct click on `a:has-text("Blog")` finds link anywhere on page ✅
- `menu.locator(...)` fails because `menu` is empty (wrong selector) ❌

**Validation:** Debug output from Step 2 will confirm/refute this.

---

**Generated:** 2025-12-03
**Last Updated:** 2025-12-03
**Developer Level:** Mid-level (with debugging guidance)
**Estimated Total Time:** 20-30 minutes (Steps 1-3)
