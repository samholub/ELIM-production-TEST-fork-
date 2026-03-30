# ELIM Production App

Portable church production setup/teardown checklist app for ELIM Church (The Well / The ELIM Arrival). Progressive Web App built with React, Firebase Realtime Database for cross-device sync, offline-capable, no authentication required.

---

## Purpose

The app solves three problems that get worse as the team grows: (1) verification at scale — Sam can't physically watch every setup task across the venue while operating the ATEM and managing the stream, (2) knowledge transfer — as volunteers rotate in and out, the system of record for what connects to what, in what order, and why needs to live somewhere other than people's heads, and (3) consistency across venues — the checklist enforces the same standard regardless of room layout, power situation, or which crew members are present.

---

## Current Status: v5.8.0

**Live URL:** https://elim-production.web.app
**Test URL:** Replit preview (connected to production Firebase)
**Test repo:** https://github.com/samholub/ELIM-production-TEST-fork-

### What's Built
- **15 sections, 59 tasks** covering full setup flow from Pre-Setup through Post-Event
- **Photo documentation** — camera capture or upload per task, auto-resized, persistent
- **Troubleshooting guides** — expandable Q&A per task for common issues
- **Per-task notes** — persistent, shared across team; preview shown above the fold (hidden on checked tasks)
- **Task detail surfacing** — static reference text shown below assignee without expanding (hidden on checked tasks)
- **Custom assignment dropdown** — dark-themed dropdown replacing native select; click-outside-to-close, unassign option
- **Countdown timer** — configurable (30/45/60/90 presets or custom) with green→yellow→red transition, auto-pause at zero, syncs across devices
- **Weekly reset** — clears checkboxes/timer, preserves assignments/notes/photos; saves history (last 52 weeks); atomic with pending-reset flag
- **SVG brand mark** — geometric `ElimLogo` component (right-pointing chevrons + block E)
- **Signal Flow diagrams** — three static SVG diagrams (Audio, Video, Network/Dante) with color-coded sections
- **Crew focused views** — tap a crew member name on dashboard → focused task list
- **Unassigned task awareness** — dashboard row, dashed/italic styling, section header counts
- **Smart auto-expand** — sections with your incomplete tasks or in-progress auto-expand
- **Crew status dashboard** — per-person progress list (you sorted first) + unassigned row
- **Timer in header** — visible on Dashboard, Checklist, My Tasks, and Crew Tasks views
- **Follow-finger swipe-back gesture** — real-time content tracking, 35% threshold, header zone exclusion
- **Power sequence** — 10-step equipment power order with teardown reminder; badges on task cards
- **Editable I/O Line List** — edit channel labels, types, notes; add/remove channels; reset to defaults
- **Repairs tracker** — Firebase-persisted with full CRUD, search, priority levels
- **Checklist task editing** — search toggle (🔍 icon), edit mode (Sam only) with add/remove/edit tasks inline
- **View stack navigation** — proper breadcrumb-style back navigation with scroll position preservation
- **Default My Tasks view** — app opens to My Tasks; dashboard accessible via back navigation
- **Error boundary** — render crashes show "Tap to Reload" instead of permanent white screen
- **Dependency warnings** — prerequisite tasks must be complete (amber modal, user can always proceed)
- **Dynamic roster** — add/remove/reorder team members in Settings
- **Remember-me** — device remembers last selected user
- **Real-time cross-device sync** — Firebase Realtime Database with localStorage offline fallback
- **Resilient storage** — graceful degradation: Firebase → localStorage → in-memory
- **Firebase write retry** — failed writes tracked and retried; stale sync events ignored for pending keys
- **Edit mode restricted** — checklist edit mode (task add/delete) only visible to Sam
- **Task deletion confirmation** — `confirm()` dialog before removing any task
- **Atomic weekly reset** — pending-reset flag prevents partial state on interruption
- **Orphan cleanup** — task deletion and roster removal clean up all supporting data keys

---

## Design Philosophy: Warm Night

- **4 visual states:** Orange = yours, Green = done, Dashed/italic/dim = unassigned, Neutral = everything else
- **No per-person colors** in the main UI (only in Settings roster management)
- **Readable at a glance:** 16px task text, 28px checkboxes, solid triangle expand arrows — mobile thumb-sized
- **Quiet by default:** collapsed sections with separator lines, minimal badges, no timestamps
- **Your tasks stand out:** orange left border + "YOU" badge on sections with your incomplete work
- **Unassigned tasks stand out:** dashed border, lighter background, italic dimmed text
- **Crew focused views:** tap a name → see just their tasks, not the full checklist
- **Progressive disclosure:** details, photos, troubleshooting hidden until expanded
- **Calm urgency:** countdown timer uses color transition (green→yellow→red), not flashing

---

## Data Management

### Firebase / Code Sync Strategy

The hardcoded `CHECKLIST_DATA` constant in the code is the **canonical source of truth**. Firebase is the live store.

**How it works:**
- First app load (or after reset) → seeds Firebase from code constant
- After that → Firebase is the live data store
- If Firebase has data, the code constant is ignored

**Workflow:**
1. **Structural changes go through code first** — update `CHECKLIST_DATA`, deploy, then reset to pull in new baseline
2. **In-app edit mode is for live adjustments** — Sunday morning task description fix, quick task addition
3. **After any in-app edit worth keeping** — mirror it back to the code constant

**Same pattern applies to:** `DEFAULT_REPAIRS`, `DEFAULT_IO_LINE_LIST`

**Important:** In-app edits to checklist structure, I/O config, or repairs are stored in Firebase. If you deploy new code and reset, the code constant replaces Firebase data. Any in-app edits not mirrored to code will be lost on reset. Always mirror live edits to the code constant before the next deploy + reset cycle.

### Storage Keys
| Key | Content |
|-----|---------|
| `elim3-checks` | Checkbox states |
| `elim3-assign` | Task assignments |
| `elim3-comp` | Completion metadata |
| `elim3-notes` | Per-task notes |
| `elim3-photos` | Photo data (base64) |
| `elim3-roster` | Team roster |
| `elim3-timer` | Timer state |
| `elim3-history` | Setup history (last 52 weeks) |
| `elim3-io-config` | I/O line list overrides |
| `elim3-repairs` | Repairs list |
| `elim3-checklist-data` | Checklist structure (sections + tasks) |
| `elim3-pending-reset` | Transient flag for atomic reset (deleted after reset completes) |

### What Resets Clear vs Preserve
| Cleared on reset | Preserved across resets |
|---|---|
| Checkboxes | Assignments |
| Completions | Notes |
| Timer | Photos |
| | Roster |
| | I/O config |
| | Repairs |
| | Checklist structure |

---

## File Structure

```
workspace/                        ← Replit workspace (primary dev environment)
├── index.html                    ← Production app (single file, Firebase + React + app code)
├── firebase.json                 ← Firebase Hosting config
├── .firebaserc                   ← Firebase project link
├── database.rules.json           ← Realtime Database rules
│
├── (Claude project files)
│   ├── ELIM-PROJECT.md           ← This file — project overview, tech reference, deployment
│   ├── CHANGELOG.md              ← Version history
│   ├── TESTING.md                ← Pre-deployment testing checklist
│   ├── BACKLOG.md                ← All planned work, organized by category
│   └── OPEN-ITEMS.md             ← On-site investigations, open questions, unknowns
│
└── (GitHub backup: github.com/samholub/ELIM-production-TEST-fork-)
```

---

## Deployment

**Hosted on:** Firebase Hosting (free Spark plan)

**Development environment:** Replit (sole dev environment — live preview, terminal, Firebase CLI all in one place)
**Source backup:** GitHub (push once per version, not per edit — `git add . && git commit -m "v5.x.0" && git push`)

**To redeploy after changes:**
1. Edit `index.html` in Replit
2. Run `firebase deploy --only hosting` in Replit terminal
3. Live in ~10 seconds at the same URL

Firebase CLI is authenticated in Replit under `sammholub@gmail.com` with project `elim-production` linked.

**Data safety:** Redeploying only replaces `index.html`. All team data lives in Firebase Realtime Database and is never affected by code deployments.

**Firebase project:** `elim-production`
- Console: https://console.firebase.google.com/project/elim-production
- Database: https://elim-production-default-rtdb.firebaseio.com
- Database rules: open read/write (no auth required, permanent)

---

## Dashboard Layout
- **Header:** Logo (left) · Centered username with ▾ tap-to-switch · ⚙️ gear (right)
- **Hero:** Stacked progress zone (tappable → full checklist) + timer zone (with picker)
- **Your Tasks:** Card linking to My Tasks with remaining count
- **Crew Status:** Per-person progress rows, you sorted first, "Unassigned tasks" row at bottom
- **Nav Cards:** 2×2 grid — Power Sequence, I/O Line List, Signal Flow, Repairs

## Settings Layout
- **Reset Checklist** (top) — two-tap confirmation
- **Team Roster** — add/remove members with color assignment
- **Setup History** — last 52 sessions with date, progress bar, percentage, time
- **About** — app version and description
- **Equipment** — placeholder (coming soon)
- **Purchases** — placeholder (coming soon)

---

## Checklist Sections & Power Sequence

### Sections (in order)
1. 🙏 Pre-Setup (3 tasks)
2. 🖥️ LED Wall Setup (3 tasks)
3. 🎤 Stage Box Setup (6 tasks)
4. 🎛️ Wing Soundboard Setup (7 tasks)
5. 🌐 Dante Network Setup (6 tasks)
6. 🎵 ProTools Mac Setup (3 tasks)
7. 🖥️ Display Setup (3 tasks)
8. 📹 Camera Setup (3 tasks)
9. 💻 ProPresenter Mac Setup (5 tasks)
10. 📡 Streaming Setup (4 tasks)
11. ⚡ Final Power-On (4 tasks)
12. 🎙️ Mic Check (2 tasks)
13. 🔊 Sound Check (4 tasks)
14. ✅ Pre-Service Checks (4 tasks)
15. 📦 Post-Event (3 tasks)

### Power Sequence (on → 1-10, off → 10-1)
1. Stage Box
2. Wing Sound Board
3. WiFi Router
4. Dante Switch / Dante AVIO
5. ProTools Macs (Tyler + Omar/Mitch)
6. Confidence Monitor
7. Bianca's ProPresenter Mac
8. Streaming Camera
9. LED Transmitter
10. Speakers

---

## Equipment Reference

> **Note:** The app code and Signal Flow SVGs still say "ATEM Mini Pro" in several places. The actual device is an **ATEM Television Studio Pro 4K**. This is tracked for code correction.

### Audio / Stage

- **Stage Box**: Midas DL251 (or DL252) — connects to Wing AES50 Supermac port A
- **Soundboard**: Behringer Wing (with Dante Expansion card) — Wing ethernet left port to WiFi Router; Dante Expansion to port 5 on switch
- **Network Switch**: 5-Port Switch (Dante network) — port 5 = Wing Dante Expansion
- **WiFi Router**: TP-Link AX55 — ATEM/Mac control network + iPad wireless control. Router on 192.168.0.x subnet.
- **Bianca's IEM**: Wing XLR Output 8
- **Wing XLR Input 8**: Bianca Talk Back

### Video / ATEM

- **Video Switcher**: ATEM Television Studio Pro 4K — static IP 192.168.0.240; ethernet to AX55 LAN port for remote control; IEC C14 power (internal PSU)
- **ATEM SDI Inputs**: IN 1 = Streaming Camera, IN 2 = Hollyland wireless receiver, IN 3 = Camera 3, IN 4 = Camera 4, IN 8 = UltraStudio Monitor SDI out (from Bianca's Mac PP). Inputs 5-7 unused/unknown.
- **ATEM SDI Outputs**: OUT 8 = Resi SDI IN. Outputs 1-7 unknown/unused.
- **UltraStudio Monitor**: Blackmagic — SDI in from ATEM SDI 8; USB-C out to Bianca's Mac
- **Anker USB Hub**: ATEM HDMI control → hub → USB-C → Bianca's Mac
- **Hollyland Receiver**: Wireless camera receiver — SDI out to ATEM SDI IN 2
- **SDI/HDMI Converter**: 1× at production booth, short SDI cable to ATEM

### Streaming

- **Resi Encoder**: Streaming encoder — lives in Wing case; SDI in from ATEM SDI OUT 8; ethernet direct to venue internet (not through router)

### Computers

- **Tyler's Mac**: ProTools for live recording — USB-C direct to Wing (NOT on Dante network)
- **Omar/Mitch's Mac**: ProTools for stream mixing — CAT6 to 5-Port Switch (Dante)
- **Bianca's Mac** (labeled "Glenn's Mac" in some places): ProPresenter for video/presentation — ethernet to AX55 LAN port (192.168.0.100, static); HDMI to ATEM via Anker hub; USB-C from UltraStudio; HDMI to Stage Box 2 (LED wall transmitter); USB-C to Confidence Monitor. WiFi active simultaneously for internet, prioritized above ethernet in macOS service order.

### Display

- **LED Wall**: 9×5 panels (45 total) via LED Transmitter from Stage Box 2
- **Stage Box 2**: Production box — Bianca's HDMI in, LED Transmitter out
- **Confidence Monitor**: Stage display from Bianca's ProPresenter Mac (USB-C)

### Control

- **Wireless Control**: WiFi Router (AX55) → iPad (Wing control) + phone (ATEM control via ATEM Software Control at 192.168.0.240)

---

## Network Architecture (Three Separate Networks)

The production setup uses three independent networks that do not interconnect:

| Network | Devices | Purpose |
|---------|---------|---------|
| **Dante** (5-port switch) | Wing Dante Expansion, Omar/Mitch's Mac, Bianca's Mac (Dante audio), AVIO — 4 of 5 ports used | Audio routing |
| **Production Control** (AX55 router, 192.168.0.x) | ATEM (192.168.0.240 static), Bianca's Mac (192.168.0.100 wired), Phone (DHCP WiFi), Wing (WiFi) | ATEM remote control, Wing iPad control |
| **Internet** (direct line) | Resi encoder | Streaming to CDN |

**Key detail:** Bianca's Mac is on both the Dante network (CAT6 to switch for audio) and the Production Control network (ethernet to AX55 for ATEM control) simultaneously.

**Troubleshooting:** "Why can't X see Y?" → Check which network each device is on. Dante devices can't see internet devices. ATEM control devices can't see Dante audio. The Resi is isolated by design.

### ATEM Network Configuration (with revert procedure)

**Current working config:**
- ATEM: 192.168.0.240 / 255.255.255.0 / Gateway 192.168.0.1 (Static)
- Mac: 192.168.0.100 / 255.255.255.0 / no gateway (Manual, System Settings → Network → Ethernet → Details → TCP/IP)
- Phone: DHCP via AX55 WiFi

**To revert to previous state:**
- ATEM: Change to DHCP via front panel or ATEM Setup (was previously DHCP, resulting in link-local 169.254.67.254)
- Mac Ethernet: System Settings → Network → Ethernet → Details → TCP/IP → change to "Using DHCP"
- Mac Service Order: System Settings → Network → three-dot menu → Set Service Order → restore original

**Notes:** Ensure cables go in AX55 LAN ports, not WAN. If router is factory reset, subnet may change — verify via router admin. USB-C from ATEM to Mac works for ATEM Setup but had firmware mismatch with ATEM Software Control; ethernet control is used instead.

---

## Hardware Organization (Updated 2026-03-29)

### Transport & Deployment Strategy
No dedicated ATEM rack case. The ATEM transports in the Pelican 1660 and deploys on whatever table the venue provides. Improvements focus on cable management and organization rather than housing.

### Wing Road Case Contents
- Behringer Wing
- 5-port Dante switch (velcroed, pre-wired to Wing Dante expansion)
- TP-Link AX55 WiFi router (velcroed, pre-wired to switch)
- Resi encoder (confirmed placement — internet runs from venue source to Wing case)
- Switchless power strip (secured inside case)
- Short pre-wired CAT6 interconnects (permanent, never unplugged)
- Single umbilical cord from case to production table for laptop power

### Pelican 1660 Case Contents
- ATEM Television Studio Pro 4K (foam-protected)
- Short colored SDI cable bundle (1.5ft Belden 1855A)
- HDMI cable for ATEM control
- ATEM IEC power cable (labeled)
- SDI/HDMI converter
- Hollyland wireless camera receiver
- Anker USB hub
- AVIO Dante module (deploys near ATEM, powered via PoE CAT6 from Wing's switch)
- Spare short SDI and HDMI cables
- Small zippered pouch for adapters

### Cable Bundles (labeled with velcro one-wrap)
- **Bundle 1 "ATEM to converters"**: Short colored SDI cables + converter
- **Bundle 2 "ATEM to Resi + network"**: SDI (ATEM OUT 8 → Resi), ethernet (ATEM → AX55)
- **Bundle 3 "Bianca's Mac"**: HDMI to ATEM (via Anker hub), HDMI to Stage Box 2, USB-C to monitor, ethernet to AX55
- **Long runs**: Camera SDI, AES50, PoE CAT6 (AVIO), venue internet (to Resi) — separate cable bag

### Cable Color Code

Both ends of each setup cable get the same color marking (heat shrink or electrical tape). Matching color dot on the destination port.

| Color | Cable | From | To |
|-------|-------|------|----|
| Green | CAT6 | Switch | Omar/Mitch's Mac |
| Blue | CAT6 | Switch | Bianca's Mac (Dante) |
| Red | Analog pair | AVIO output | ATEM audio in |
| Yellow | Ethernet | Internet source | Resi |
| Orange | USB | ATEM | Bianca's Mac (via Anker hub) |
| White | HDMI | Bianca's Mac | Confidence Monitor |
| Purple | HDMI | Bianca's Mac | LED Transmitter (via Stage Box 2) |

Long runs (AES50 to stage box, CAT6 to LED wall, camera SDI) don't need color coding — only one destination possible.

### Table Cable Management
- Adhesive velcro loop strip behind ATEM (strain relief, cable organization)
- Velcro one-wrap ties anchor cables to strip
- Gaff tape loops at table edge for cable drops (no adhesive clips)
- Power cables separated from signal cables

### Wall Spacing Reference
- 25 feet from base of LED wall to chairs
- 13 feet from edge of carpet to chairs (Sunny's measurement)
- 11.5 feet from speakers to corner of actual wall

---

## Hardware Decisions Log

### Decisions Made
| Decision | Why |
|----------|-----|
| Router/switch/AVIO/Resi in Wing case | These devices connect to the Wing and Resi needs direct internet — grouping them reduces cable runs to production table |
| No inter-zone network cable | Dante switch and Resi internet are separate networks with no need to communicate |
| No laptops powered from ATEM area | Avoids cable runs to operator positions and single point of failure |
| Pelican transport, table deployment | Simpler than rack case for volunteers; no sliding shelf, no restraint mechanism, no thermal concerns |
| Velcro straps for cable bundling | Reusable, adjustable, tool-free, gentler on cables — better for weekly changes |
| ATEM static IP on router subnet | Enables remote control from Mac (wired) and phone (WiFi) via same router |

### Ideas Evaluated and Rejected
| Idea | Why Rejected |
|------|-------------|
| Dedicated ATEM rack case (3U, 4U, or 6U) | Too heavy/complex for the actual cable count and volunteer operation. Sliding shelf, transport restraint, and thermal management add points of failure for no real gain. |
| Formal 1U SDI patch panel | Overbuilt for 4 SDI connections. Flexible jumper tails solve cable-drag with fewer failure points. |
| EVA accessory case | Not enough loose gear to justify a separate padded case |
| Powering laptops/router from ATEM rack PDU | Creates cable clutter, single point of failure, exceeds capacity |
| Braided cable sleeve for cross-table backbone | More rigid than helpful, adds cost without real benefit |
| Non-slip shelf liner | Table surface has been fine without it |
| Adhesive cable clips on table edge | Gaff tape loops are faster, removable, and don't leave residue at varying venues |

---

## Team Members
Managed dynamically via Settings. Default roster: Glenn, Sunny, Omar, Mitch, Michael, Taylor, Seth, Sam, Bianca, Bri, Leo, Austin, Tyler, Kolton, Savanah, Emily

---

## Technical Notes

- **Framework:** React 18 (via CDN, no build step)
- **Styling:** Inline styles, no CSS framework
- **Fonts:** DM Sans (body) + JetBrains Mono (data/timers) via Google Fonts
- **Storage:** Firebase Realtime Database with `window.storage` API wrapper + `_ls` safe localStorage wrapper
- **Real-time sync:** Firebase `.on('value')` listeners dispatch `elim-sync` CustomEvents, consumed by `useStorage` hook
- **Resilience:** Firebase init in try/catch, all localStorage in `_ls` safe wrapper — app degrades gracefully
- **Photo handling:** Base64 JPEG, resized to max 800×600, 70% quality. Stored in Firebase as string values.
- **Theme:** "Warm Night" — 4-state color system (orange #D4763A / green #5CB87A / dashed-dim for unassigned / neutral), warm cream text (#F0E6D8), dark warm background (#111010)
- **Brand mark:** SVG `ElimLogo` component — two right-pointing chevrons + block E, sharp corners, scaled 82% inside circle
- **Countdown timer:** `timerColor()` helper returns green/yellow/red based on remaining %. Timer state: `{startedAt, accumulated, targetMinutes}`. Auto-pauses at zero.
- **Weekly reset:** Saves history before clearing. History record: `{date, pct, elapsed, tasks, total}`. Uses live `checklistData` for accurate `total`. Keeps last 52 weeks.
- **Signal Flow:** `SignalFlowView` component with three SVG sections using helper sub-components (Box, Arrow, Label, HLine, VLine). Audio viewBox 570×660. Video 480×510. Network 480×360. Color-coded: orange/purple/green.
- **Header positioning:** `position: fixed; top: 0` with `paddingTop: calc(env(safe-area-inset-top, 0px) + 8px)`. Body has `padding-top: 0`. Background extends behind iOS status bar. `overflow-x: clip` on body. Header spacer div: 48px height.
- **Back swipe:** `useBackSwipe` hook with follow-finger drag gesture. Real-time `translateX` via direct DOM manipulation (refs, not state). 35% screen width threshold. `data-no-swipe` attribute excludes photo scroll areas. Top-left zone (top 80px, left 120px) excluded.
- **Crew focused views:** `MyTasksView` accepts `targetUser` prop. `__unassigned__` special value filters to tasks with no assignee.
- **Unassigned styling:** `CheckItem` computes `isUnassigned` from `!assignee && !checked`, applies dashed border + dim italic. Section headers compute `unassignedCount`.
- **Card styling:** Computed via `cardStyle` function to avoid CSS property conflicts (especially `border` vs `borderLeft`).
- **Dependency warnings:** `dependsOn` property on task items. CheckItem checks prerequisite completion before toggling. Amber modal with Continue/Cancel.
- **No auth:** User selects name on launch, no passwords. Remember-me via localStorage.
- **Shared state:** All data shared (visible to all users) via Firebase
- **Hosting:** Firebase Hosting (free tier: 10GB bandwidth/month, 1GB database storage)
- **Error boundary:** `ErrorBoundary` class component wraps `ElimProductionApp` at mount point. Catches render errors, shows reload prompt. Prevents white-screen crashes from propagating.
- **Firebase write resilience:** `_pendingWrites` Set tracks keys with in-flight writes. `_startListener` ignores incoming `elim-sync` events for pending keys. Failed writes retry once after 5 seconds.
- **Atomic reset:** `newWeek` writes `elim3-pending-reset` flag before clearing state. On load, `useEffect` checks for flag and completes interrupted resets.
- **Performance:** `allItems` memoized via `useMemo(, [checklistData])`. `CheckItem` wrapped in `React.memo`. Photos excluded from initial load gate. Static style objects and arrays hoisted to module scope.
