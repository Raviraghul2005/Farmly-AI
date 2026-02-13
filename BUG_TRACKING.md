# Bug Tracking Document
**Project:** Farmly AI  
**Version:** 1.0.0 MVP  
**Last Updated:** February 13, 2026

---

## Bug Priority Levels

- **P0 (Critical):** Blocks demo, must fix immediately
- **P1 (High):** Major feature broken, should fix before demo
- **P2 (Medium):** Minor issue, fix if time permits
- **P3 (Low):** Enhancement or cosmetic issue, post-demo

---

## Active Bugs

### P0 - Critical (Demo Blockers)

*No P0 bugs currently identified*

---

### P1 - High Priority (Pre-Demo)

#### BUG-001: Authentication Redirect on Page Reload
- **Status:** 🔍 Under Investigation
- **Priority:** P1 (High)
- **Severity:** High
- **Component:** Frontend - Authentication
- **Reported:** February 13, 2026
- **Reporter:** QA Team
- **Assigned:** Frontend Team

**Description:**
When a logged-in user refreshes the page (F5 or browser reload), the authentication state is lost and the user is redirected to the login page, even though the auth data exists in localStorage.

**Steps to Reproduce:**
1. Login to the app with valid credentials
2. Navigate to any protected page (e.g., /home)
3. Press F5 or click browser reload
4. **Expected:** User stays on the same page
5. **Actual:** User is redirected to login page

**Technical Details:**
- Auth state is stored in localStorage via Zustand persist middleware
- Data IS visible in browser DevTools under localStorage
- Issue appears to be a timing problem between:
  - React Router initialization
  - Zustand store rehydration from localStorage
- localStorage operations are synchronous in browsers

**Investigation Notes:**
- Initial hypothesis: Race condition during page reload
- Finding: Zustand persist middleware loads synchronously
- Attempted Fix #1: Added `_hasHydrated` tracking → FAILED (infinite loading)
- Attempted Fix #2: Reverted to simple implementation
- Current Status: Investigating router timing

**Files Involved:**
- `frontend/src/store/authStore.ts`
- `frontend/src/App.tsx`
- `frontend/src/components/common/ProtectedRoute.tsx`

**Related Documentation:**
- See `frontend/BUGFIXES.md` lines 12-154 for detailed investigation

**Workarounds:**
1. **For Demo:** Don't refresh the page during demonstration
2. **For Testing:** Use navigation links instead of browser reload
3. **Demo Mode:** Set `VITE_DEMO_MODE=true` to bypass auth

**Proposed Solutions:**
1. Add React Router loader functions to check auth before route loads
2. Move auth check to a layout component instead of route-level
3. Add small delay before initial routing decision (not ideal)
4. Use sessionStorage instead of localStorage (loses auth on browser close)

**Impact:**
- **User Experience:** High - Users lose their session unexpectedly
- **Demo Impact:** Medium - Can be avoided by not refreshing
- **Data Loss:** None - Data is preserved in localStorage

**Next Steps:**
1. Add detailed logging to track initialization sequence
2. Test localStorage availability on page load
3. Review React Router documentation for auth patterns
4. Consider moving to React Router v6.4+ data APIs

**Updates:**
- [2026-02-13] Bug identified during E2E testing
- [2026-02-13] Infinite loading fix reverted, back to simple implementation

---

### P2 - Medium Priority (Nice to Fix)

#### BUG-002: Circular Dependency Warning in Build
- **Status:** ℹ️ Informational
- **Priority:** P2 (Medium)
- **Severity:** Low
- **Component:** Frontend - Build System
- **Reported:** February 13, 2026
- **Reporter:** Build System

**Description:**
Vite build generates a warning about circular chunk dependencies between `vendor` and `react-vendor` chunks.

**Build Output:**
```
Circular chunk: vendor -> react-vendor -> vendor. 
Please adjust the manual chunk logic for these chunks.
```

**Technical Details:**
- Occurs during production build (`pnpm run build`)
- Does not prevent build from completing
- Does not affect runtime functionality
- Related to manual chunk splitting in `vite.config.ts`

**Files Involved:**
- `frontend/vite.config.ts` (lines 20-80 - manualChunks configuration)

**Impact:**
- **Build:** Warning message only, build succeeds
- **Runtime:** No impact on app functionality
- **Performance:** No measurable impact on load times

**Proposed Solution:**
Adjust manual chunk configuration to prevent circular dependency:
```typescript
manualChunks: {
  'react-vendor': ['react', 'react-dom'],
  'router-vendor': ['react-router-dom'],
  'query-vendor': ['@tanstack/react-query'],
  // Don't mix overlapping dependencies
}
```

**Priority Justification:**
- Not user-facing
- No functional impact
- Can be addressed post-demo
- Build optimization, not a bug

---

### P3 - Low Priority (Post-Demo)

*No P3 bugs currently identified*

---

## Resolved Bugs

### ✅ BUG-003: Scrolling Not Working (FIXED)
- **Status:** ✅ Fixed
- **Priority:** P0 → Resolved
- **Component:** Frontend - CSS
- **Reported:** February 13, 2026
- **Resolved:** February 13, 2026
- **Resolution Time:** < 1 hour

**Description:**
Mouse wheel scrolling did not work on any page. Scrollbar dragging did work, indicating CSS blocking wheel events.

**Root Cause:**
1. `overscroll-behavior-y: contain` in inline styles (MAIN CULPRIT)
2. `overflow-y: auto` instead of `overflow-y: scroll`
3. Complex height inheritance preventing natural scroll

**Fix Applied:**
- Removed `overscroll-behavior-y: contain` from `index.html` and `index.css`
- Changed `overflow-y: auto` to `overflow-y: scroll` in `index.css`
- Changed `min-height: 100%` to `min-height: 100vh`
- Removed `height: 100%` from html element
- Removed unnecessary `position: relative` from body

**Files Modified:**
- `frontend/index.html`
- `frontend/src/index.css`

**Verification:**
- ✅ Mouse wheel scrolling works on all pages
- ✅ Scrollbar still works
- ✅ Horizontal scroll prevented (as intended)
- ✅ Smooth scrolling on mobile devices
- ✅ No layout shifts or overflow issues

**Reference:** `frontend/BUGFIXES.md` lines 9-114

---

### ✅ BUG-004: Infinite Loading Spinner (FIXED)
- **Status:** ✅ Fixed
- **Priority:** P0 → Resolved
- **Component:** Frontend - Authentication
- **Reported:** February 13, 2026
- **Resolved:** February 13, 2026
- **Resolution Time:** < 30 minutes

**Description:**
After attempting to fix auth persistence with hydration tracking, app showed infinite loading spinner and never became interactive.

**Root Cause:**
Added asynchronous hydration tracking (`_hasHydrated` flag) for a synchronous operation (localStorage reading). This introduced a deadlock where the app waited for a callback that never fired correctly.

**Fix Applied:**
Reverted the hydration tracking code and returned to simple Zustand persist implementation. localStorage operations are synchronous in browsers, so tracking is unnecessary.

**Lesson Learned:**
Don't overcomplicate synchronous operations with async tracking patterns.

**Reference:** `frontend/BUGFIXES.md` lines 204-218

---

## Known Limitations (Not Bugs)

### LIMIT-001: Demo Mode OTP
- **Type:** Intentional Limitation
- **Component:** Backend - Authentication
- **Description:** Demo accounts use fixed OTP `123456` when `DEMO_MODE=true`
- **Impact:** Only affects demo/development environment
- **Production:** Set `DEMO_MODE=false` for real OTP via Twilio
- **Reference:** `backend/DEMO_ACCOUNTS.md`

### LIMIT-002: ML Service Accuracy
- **Type:** Model Limitation
- **Component:** ML Service - Disease Detection
- **Description:** Disease detection accuracy varies by image quality and crop type
- **Current Accuracy:** ~85-95% on test set
- **Target:** >98% for trained diseases
- **Mitigation:** Provide confidence scores, allow user feedback
- **Future:** Continuous model training with user feedback

### LIMIT-003: Market Price Data Availability
- **Type:** External API Limitation
- **Component:** Backend - Market Service
- **Description:** Agmarknet API may have delays or missing data for some markets
- **Fallback:** Uses Gemini AI to generate estimated prices when API unavailable
- **Impact:** Price recommendations may be less accurate in some regions
- **Reference:** `backend/MARKET_INTEGRATION.md`

---

## Testing Gaps

### GAP-001: Multilingual Voice Testing
- **Status:** Not Tested
- **Impact:** High
- **Description:** Voice recognition not tested with real native speakers
- **Recommendation:** Test with at least 2-3 native speakers per language
- **Priority:** P1 for demo

### GAP-002: Real Device Camera Testing
- **Status:** Partially Tested
- **Impact:** Medium
- **Description:** Camera functionality tested on emulators, not real devices
- **Recommendation:** Test on actual iOS and Android devices
- **Priority:** P1 for demo

### GAP-003: Poor Network Conditions
- **Status:** Not Tested
- **Impact:** Medium
- **Description:** App not tested under slow 3G or intermittent connectivity
- **Recommendation:** Use Chrome DevTools network throttling
- **Priority:** P2

### GAP-004: Load Testing
- **Status:** Not Tested
- **Impact:** Low
- **Description:** API endpoints not tested under concurrent load
- **Recommendation:** Use Apache JMeter or similar tool
- **Priority:** P3 (post-demo)

---

## Bug Statistics

| Priority | Open | In Progress | Resolved | Total |
|----------|------|-------------|----------|-------|
| P0 | 0 | 0 | 2 | 2 |
| P1 | 1 | 0 | 0 | 1 |
| P2 | 1 | 0 | 0 | 1 |
| P3 | 0 | 0 | 0 | 0 |
| **Total** | **2** | **0** | **2** | **4** |

---

## Bug Velocity

| Date | Bugs Opened | Bugs Closed | Net Change |
|------|-------------|-------------|------------|
| 2026-02-13 | 4 | 2 | +2 |

---

## Action Items

### Immediate (Before Demo)
1. [ ] **BUG-001:** Investigate and fix auth persistence issue
   - Owner: Frontend Team
   - Deadline: Before demo
   - Workaround documented for demo

2. [ ] **GAP-001:** Test voice recognition with native speakers
   - Owner: QA Team
   - Deadline: Before demo
   - At least Hindi and Tamil

3. [ ] **GAP-002:** Test camera on real devices
   - Owner: QA Team
   - Deadline: Before demo
   - iOS and Android

### Post-Demo
1. [ ] **BUG-002:** Fix circular dependency warning
   - Owner: Frontend Team
   - Priority: P2
   - No urgency

2. [ ] **GAP-003:** Network throttling tests
   - Owner: QA Team
   - Priority: P2

3. [ ] **GAP-004:** Load testing
   - Owner: Backend Team
   - Priority: P3

---

## Bug Report Template

When reporting new bugs, use this template:

```markdown
#### BUG-XXX: [Title]
- **Status:** 🔍 New / 🔧 In Progress / ✅ Fixed
- **Priority:** P0 / P1 / P2 / P3
- **Severity:** Critical / High / Medium / Low
- **Component:** Frontend / Backend / ML Service / Database
- **Reported:** YYYY-MM-DD
- **Reporter:** Name
- **Assigned:** Team/Person

**Description:**
[Clear description of the bug]

**Steps to Reproduce:**
1. Step 1
2. Step 2
3. **Expected:** [What should happen]
4. **Actual:** [What actually happens]

**Technical Details:**
[Any relevant technical information]

**Files Involved:**
- file/path.ts

**Impact:**
- **User Experience:** High / Medium / Low
- **Demo Impact:** High / Medium / Low
- **Data Loss:** Yes / No

**Proposed Solution:**
[How to fix it]
```

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-13 15:45 | Initial bug tracking document created | QA Team |
| 2026-02-13 15:45 | Added BUG-001 through BUG-004 | QA Team |
| 2026-02-13 15:45 | Added known limitations and testing gaps | QA Team |

---

**Next Review:** Before demo rehearsal  
**Document Owner:** QA Team  
**Contact:** qa@farmly.ai
