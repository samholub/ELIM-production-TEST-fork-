# ELIM Production App — Project Brief for Cursor Audit & Development

## Purpose of This Document

This brief provides full context for an AI-assisted audit and continued development of the ELIM Production app. The app is a single-file React PWA (`index.html`, ~2,400 lines) used by ~16 church volunteers to manage weekly AV production setup/teardown at varying venues.

Read this alongside `.cursorrules` (architecture constraints and code patterns) and the project docs (`BACKLOG.md`, `OPEN-ITEMS.md`, `TESTING.md`, `CHANGELOG.md`).

---

## Phase 1: Code Audit

Before making any changes, audit the codebase for the following. Generate a findings report before implementing fixes.

### 1.1 Structural & Maintainability Issues

- **Component size:** Several components exceed 200 lines (`ChecklistView`, `IOListView`, `DashboardView`). Identify opportunities to extract sub-components without breaking the single-file architecture.
- **Prop drilling depth:** `ElimProductionApp` passes 15+ props through multiple layers. Map the prop chains and identify candidates for React Context (if the complexity warrants it — don't over-engineer for 20 components).
- **Code duplication:** The `JSON.parse(JSON.stringify())` deep clone pattern appears 8+ times. The edit toolbar pattern (search + edit toggle + reset) is duplicated across Checklist, I/O, and Repairs. Document all duplication sites.
- **Dead code:** `PlaceholderView` serves two nav cards (Equipment, Purchases) that lead to "Coming soon" screens. Identify any other dead code paths.
- **Inline style sprawl:** Styles are defined as objects inside render functions, recreated every render. Identify which style objects should be hoisted to module scope or memoized.

### 1.2 Performance Concerns

- **Re-render cascade:** When any `useStorage` value changes, Firebase listeners dispatch `elim-sync` events. Trace which components re-render when a single checkbox is toggled on another device. Are there unnecessary re-renders?
- **Photo storage:** Base64 photos stored in Firebase under `elim3-photos`. Each photo is auto-resized to 800×600 at 70% quality, but many photos could still mean large Firebase reads on initial load. Is the current approach sustainable for 52 weeks × multiple photos?
- **Timer updates:** The countdown timer triggers re-renders every second via `setInterval`. Does this cause unnecessary work in components that display the timer?
- **`ALL_ITEMS` computation:** `CHECKLIST_DATA.flatMap(...)` runs at module scope and again derived from `checklistData` prop in several components. Is this computed more often than necessary?

### 1.3 Reliability & Edge Cases

- **Offline behavior:** The app has Firebase → localStorage → in-memory fallback. Test: What happens when Firebase is unreachable during a write? Does localStorage get written? What about conflict resolution when two devices edit the same task offline and come back online?
- **Data migration:** When `CHECKLIST_DATA` changes in code and the app is redeployed, existing `elim3-checklist-data` in Firebase is never overwritten (by design — it only seeds on first load or after reset). Is there a migration path for structural changes that shouldn't require a full reset?
- **Race conditions in useStorage:** The `useStorage` hook reads Firebase, then sets state. If two rapid changes come in via the `elim-sync` event, could the second overwrite the first before React processes the batch?
- **Large roster:** The roster is unbounded. What happens with 50+ team members? Does the login screen, dashboard crew status, or assignment dropdown degrade?

### 1.4 Security & Data

- **Firebase rules are fully open** (`".read": true, ".write": true`). This is intentional for a small trusted team, but document the risk and what minimal rules would look like if needed (e.g., validate data shape, prevent deletion of root node).
- **API key in source:** Firebase API key is in the HTML. This is normal for Firebase (keys are not secret — security is in database rules), but flag it for awareness.
- **No input sanitization:** Task text, notes, repair descriptions are stored as-is. Could XSS be injected via Firebase that renders in another user's browser? React's default escaping likely handles this, but verify.

### 1.5 Accessibility

- **No ARIA attributes** on custom interactive elements (expandable sections, custom dropdowns, swipe gestures)
- **Color-only state indicators** — are the 4 visual states distinguishable for colorblind users?
- **Focus management** — keyboard navigation through the checklist, edit modes, modals
- **Screen reader support** — semantic HTML usage, heading hierarchy

### 1.6 Mobile-Specific

- **Touch target sizes** — verify all interactive elements meet 44×44px minimum
- **Swipe gesture conflicts** — does the custom `useBackSwipe` conflict with any native gestures on iOS or Android?
- **PWA behavior** — is the manifest complete? Does Add to Home Screen work correctly? What's missing for a proper offline-first experience?
- **Safe area handling** — is `env(safe-area-inset-*)` applied correctly for all notched phones?

---

## Phase 2: Priority Bug Fixes

These are documented in `BACKLOG.md` under Priority 0. Fix in this order:

1. **Dependency warning modal** — make backdrop opaque enough to read warning text
2. **Scroll position preservation** — store and restore scroll position per view
3. **Visible scrollbar** — hide via CSS
4. **Assignment dropdown styling** — custom dropdown matching dark theme, larger tap targets
5. **Populated notes surfacing** — show note preview below assignee without expanding
6. **Dante I/O field overflow** — constrain edit fields to screen width

Each fix should be a surgical edit to `index.html`. Run the relevant test cases from `TESTING.md` after each fix.

---

## Phase 3: UX Improvements

### 3.1 Navigation Model Rethink (Priority 1 — Biggest Impact)

The current flow is: Open app → Dashboard → Tap "Your Tasks" → See tasks. For the 15 of 16 team members who are individual contributors (not team leads), the dashboard is a speed bump.

**Recommended approach:** Make the app open directly to a personal task view. The dashboard becomes a secondary view accessible via a nav element. This requires:
- Changing the default `view` state from `"dashboard"` to `"mytasks"` (or a new combined view)
- Surfacing the timer and overall progress in the My Tasks header
- Adding navigation to the dashboard from My Tasks (a "Team" or "Overview" button)
- Preserving the crew-focused view (tap a name → see their tasks)

**Do not implement without reviewing the three options in BACKLOG.md Priority 1 and selecting one.**

### 3.2 Visual Noise Reduction

- Remove `PlaceholderView` and its two nav card entries (Equipment, Purchases)
- Reduce nav grid from 3×2 to 2×2
- Hide checklist search bar behind a search icon toggle
- Move checklist edit mode behind Settings or long-press (admin function)
- Remove "Last setup" stat from dashboard hero (it lives in Settings → Setup History)

### 3.3 Task Card Improvements

- Show assignee as plain text by default; dropdown only when unassigned or in assign mode
- Add "All" assignment option for communal tasks
- Brief check-off animation
- "You're done!" message when personal tasks complete but unassigned tasks remain

---

## Phase 4: Code Quality

### 4.1 Extract Utilities
```javascript
// Deep clone utility (replace 8+ instances)
const deepClone = (obj) => JSON.parse(JSON.stringify(obj));

// Edit toolbar shared component (Checklist, I/O, Repairs all use this pattern)
function EditToolbar({ searchValue, onSearch, editMode, onToggleEdit, onReset, itemCount, totalCount }) { ... }
```

### 4.2 Style Hoisting
Move frequently-used style objects to module scope:
```javascript
// Currently inside render functions, recreated every render
const cardBaseStyle = { borderRadius: 10, padding: "11px 12px", transition: "all 0.2s ease" };
const sectionHeaderStyle = { ... };
```

### 4.3 Component Extraction Candidates
- Timer display (used in Dashboard, Checklist header, My Tasks header)
- Section header (expanding/collapsing with counts, YOU badge, unassigned count)
- Task search/filter logic (duplicated between Checklist and Repairs)

### 4.4 Evaluate Signal Flow SVGs
~200 lines of hand-crafted SVG. Options:
- Keep as-is (always sharp, professional look, but manual to update)
- Replace with reference photos (easier to update, but lose resolution)
- Generate dynamically from I/O data (ambitious — eliminates dual source of truth but significant complexity)

---

## Phase 5: New Features (Build One at a Time)

Prioritized from `BACKLOG.md`:

1. **Teardown checklist** — reverse of setup with teardown-specific tasks; power sequence already has teardown guidance
2. **Equipment inventory** — serial numbers, condition tracking, who-has-what
3. **Setup time analytics** — trend visualization from existing history data
4. **Photo comparison** — this week vs reference photos side-by-side
5. **Full offline caching** — service worker for loading without connectivity
6. **Training mode** — expanded instructions + video links toggle per task

---

## Firebase Configuration Reference

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyAq8TGyBunq_8g1z_3lF-zO73taalYfGyg",
  authDomain: "elim-production.firebaseapp.com",
  databaseURL: "https://elim-production-default-rtdb.firebaseio.com",
  projectId: "elim-production",
  storageBucket: "elim-production.firebasestorage.app",
  messagingSenderId: "222489117430",
  appId: "1:222489117430:web:8364febc826417f0dda2d7"
};
```

Database rules (current — fully open):
```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

---

## Equipment & Team Context

### Key Team Members (AV Roles)
- **Sam** — ATEM operation and stream management (app owner/developer)
- **Bianca** — ProPresenter video control, bridges Dante audio and ATEM video
- **Tyler** — Live recording via Behringer Wing (USB-C direct, NOT on Dante)
- **Omar/Mitch** — Stream mix via Dante network
- **Glenn** — ProPresenter Mac operator

### Equipment Stack
- Behringer Wing (soundboard) + Midas DL251 stage box (AES50)
- Blackmagic ATEM Mini Pro (video switcher)
- Resi Mini (streaming encoder)
- 5-port Dante network switch
- 45-panel LED wall (9×5) via LED Transmitter
- WiFi router for iPad wireless control

### Physical Constraints
- Setup/teardown happens at varying venues, 1-2x per week
- Volunteers operate under time pressure (30-90 minute setup windows)
- Venue WiFi is unreliable — app must degrade gracefully
- Phones are the primary device — no laptops during setup

---

## Testing Protocol

After any change:
1. Open on a mobile device (or Chrome DevTools mobile emulation)
2. Test the specific feature/fix against the relevant section in `TESTING.md`
3. Verify no regressions in: checkbox sync, timer, back navigation, scroll behavior
4. Test with Firebase connected AND with Firebase blocked (localStorage fallback)
5. Test cross-device: make a change on one device, verify it appears on another within 1-2 seconds

---

## What NOT to Do

- **Don't split into multiple files.** The single-file architecture is intentional — it deploys to Firebase Hosting with zero build step.
- **Don't add a build system.** No webpack, no Vite, no esbuild. The CDN approach works and eliminates toolchain complexity for a non-developer maintainer.
- **Don't add authentication.** The no-auth model is intentional for a small trusted team. Adding auth would create a barrier for volunteers.
- **Don't add TypeScript.** The Babel CDN handles JSX transformation. TypeScript would require a build step.
- **Don't add CSS frameworks.** Tailwind, styled-components, etc. would either require a build step or add CDN weight. Inline styles work fine at this scale.
- **Don't refactor the navigation to React Router.** The custom navigation is simple and works. Router adds complexity and a dependency.
- **Don't over-architect.** This is a ~2,400 line app for 16 volunteers. Context providers, state management libraries, and abstraction layers are overkill unless they solve a concrete, measurable problem.
