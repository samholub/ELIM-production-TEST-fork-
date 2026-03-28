# ELIM Production App — Open Items

Things that need on-site investigation or physical verification. These can't be resolved in code — someone needs to be at the venue.

Once resolved, items either:
- Become a backlog item (if work is needed) → move to BACKLOG.md
- Get added to the code (if it's just information) → update index.html + ELIM-PROJECT.md
- Get deleted (if it turns out to be nothing)

Last updated: 2026-03-28

---

## Equipment / Wiring Unknowns

### Wing XLR Input 8
- **What:** Unknown source connected to Wing XLR input port 8
- **Not to be confused with:** Stage Box channel 8, which is Keys/Piano L
- **Action:** Identify what's plugged in, update I/O list
- **Updates when resolved:** `DEFAULT_IO_LINE_LIST` in code, I/O Line List view

### ATEM SDI Port Assignments
- **What:** Input 1, 2, 8 and Output 2, 8 — sources and destinations unknown
- **Action:** Document which camera/source goes to which ATEM input, which output goes where
- **Updates when resolved:** Video Signal Flow SVG, possibly camera setup tasks

### Omar/Mitch Mac → ATEM Connection
- **What:** Connection type between Omar/Mitch's Mac and ATEM is unknown (USB? SDI? HDMI?)
- **Action:** Check physical connection on-site
- **Updates when resolved:** Equipment Reference in ELIM-PROJECT.md, possibly signal flow SVG

### Resi Encoder SDI Input
- **What:** Which ATEM output port feeds the Resi Encoder SDI input
- **Action:** Check physical connection on-site
- **Updates when resolved:** Camera Setup tasks, Video Signal Flow SVG

---

## Dante Network Questions

### Dante Input/Output Configuration
- **What:** Full picture of what's being sent/received on Dante — which channels, which devices, what routing
- **Action:** Open Dante Controller on-site, document all active routes
- **Updates when resolved:** I/O Line List (Dante tab), Network Signal Flow SVG, Dante Network Setup tasks

### Dante Config Export
- **What:** Can the Dante configuration be exported/backed up? If so, where and how?
- **Action:** Check Dante Controller for export functionality
- **Updates when resolved:** Could become a backup task in Post-Event section or a doc reference

### Dante Device Inventory
- **What:** Complete list of devices visible on the Dante network
- **Action:** Open Dante Controller, screenshot or list all connected devices
- **Updates when resolved:** Equipment Reference, Network Signal Flow SVG

---

## Physical Setup Questions

### Stage Box Height
- **What:** Is the current stage box height appropriate, or does it need risers/adjustment?
- **Action:** Evaluate during setup — check cable strain, ergonomics, accessibility
- **Updates when resolved:** Possibly a note on sb-1 task, or a repairs item if work is needed

### Speaker/Wall Spacing
- **What:** What's the correct spacing between speakers and LED wall panels?
- **Action:** Measure and document the current setup, determine if a standard should be set
- **Updates when resolved:** New checklist task (already in BACKLOG.md), possibly a reference photo

### LED Wall Power Timing
- **What:** Confirmed that LED wall power can be plugged in at any time (not sequence-dependent)
- **Action:** Verify on-site that this is true, then add note to LED Wall tasks
- **Updates when resolved:** Add detail text to lw-1, lw-2, or lw-3

---

## Resolved Items

*Move items here with resolution date and what was done.*

*(none yet)*
