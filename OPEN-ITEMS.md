# ELIM Production App — Open Items

Things that need on-site investigation or physical verification. These can't be resolved in code — someone needs to be at the venue.

Once resolved, items either:
- Become a backlog item (if work is needed) → move to BACKLOG.md
- Get added to the code (if it's just information) → update index.html + ELIM-PROJECT.md
- Get deleted (if it turns out to be nothing)

Last updated: 2026-03-29

---

## Equipment / Wiring Unknowns

### ATEM SDI Ports — Remaining Unknowns
- **What:** Inputs 5, 6, 7 and Outputs 1-7 — sources and destinations unknown (or unused)
- **Known:** IN 1 = Streaming Camera, IN 2 = Hollyland receiver, IN 3 = Camera 3, IN 4 = Camera 4, IN 8 = UltraStudio SDI out, OUT 8 = Resi SDI in
- **Action:** Confirm whether ports 5-7 in and 1-7 out are unused or connected to something
- **Updates when resolved:** ELIM-PROJECT.md Equipment Reference, Video Signal Flow SVG

### Omar/Mitch Mac → ATEM Connection
- **What:** Is there any connection between Omar/Mitch's Mac and the ATEM? Their Mac is on the Dante network for stream mix — it's unclear if they also connect to the ATEM for any purpose.
- **Action:** Confirm on-site whether Omar/Mitch's Mac has any ATEM connection, or if they're Dante-only
- **Updates when resolved:** Equipment Reference, possibly signal flow SVG

---

## Dante Network Questions

### Dante Input/Output Configuration
- **What:** Full picture of what's being sent/received on Dante — which channels, which devices, what routing
- **Known from I/O list:** Dante 47-48 = ProPresenter PC
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

### LED Wall Power Timing
- **What:** Confirmed that LED wall power can be plugged in at any time (not sequence-dependent)
- **Action:** Verify on-site that this is true, then add note to LED Wall tasks
- **Updates when resolved:** Add detail text to lw-1, lw-2, or lw-3

---

## Hardware / Organization Questions

### PA Speaker Power Draw
- **What:** Actual power draw per speaker and total per PA tower side
- **Why it matters:** If total per side exceeds 15A, each tower needs two circuits instead of one
- **Action:** Check rated wattage on each speaker model, or use Kill-A-Watt during sound check at full expected volume
- **Updates when resolved:** Stage power distribution plan (number of extension cords/circuits needed)

### LED Wall Box Caster Mounting
- **What:** Mounting pattern and plate dimensions on the bottom LED wall box
- **Action:** Measure current caster mount points (bolt pattern, plate size) to confirm Service Caster Series #30 5"×2" plate-mount casters will fit
- **Updates when resolved:** Caster purchase spec and installation

### Confidence Monitor Dimensions
- **What:** Need to confirm the physical size of the confidence monitor to determine whether it fits in the Pelican 1660 alongside the ATEM, or needs a separate padded sleeve.
- **Action:** Measure monitor width, height, depth
- **Updates when resolved:** Pelican 1660 compartment layout

### Pelican 1660 Full Contents Inventory
- **What:** Need to catalog all items from the production booth and stage box that will live in the Pelican 1660, beyond what's already documented.
- **Action:** Lay out all booth gear, decide what goes in Pelican vs cable bag vs separate transport
- **Updates when resolved:** Pelican packing reference (could become a checklist item or reference photo in the app)

### 3D Printed Stacking Cup Fit
- **What:** Existing stacking cups are on the LED wall boxes but nothing commercial fits inside them. Need precise measurements of the cup interior to design a PETG insert.
- **Action:** Measure cup interior diameter, depth, and draft angle. Print test piece at 100% and 101%.
- **Updates when resolved:** Final STL file for printing, quantity needed

### Cable Run Distances at Primary Venues
- **What:** Actual distances from wall outlets to LED wall, PA towers, stage box at the venues used most often
- **Action:** Measure during setup at 1-2 primary venues
- **Updates when resolved:** Extension cord length purchases (50ft vs 75ft vs 100ft)

### Bianca's Mac Multi-Network Verification
- **What:** Bianca's Mac is documented as being on both the Dante network (CAT6 to switch) and the Production Control network (ethernet to AX55). Need to confirm the physical arrangement — is this two ethernet connections (one through Anker hub?) or some other setup?
- **Action:** Check physical connections at Bianca's station
- **Updates when resolved:** Equipment Reference, network architecture documentation

---

## Resolved Items

### Wing XLR Input 8 — RESOLVED 2026-03-29
**Was:** Unknown source connected to Wing XLR input port 8.
**Resolution:** Bianca Talk Back. Documented in I/O line list and Equipment Reference.

### ATEM SDI Port Assignments (partial) — RESOLVED 2026-03-29
**Was:** Inputs 1, 2, 8 and Outputs 2, 8 — sources and destinations unknown.
**Resolution:** IN 1 = Streaming Camera, IN 2 = Hollyland receiver, IN 3 = Camera 3, IN 4 = Camera 4, IN 8 = UltraStudio SDI out (Bianca's Mac PP), OUT 8 = Resi SDI in. Remaining ports (IN 5-7, OUT 1-7) moved to new open item above.

### Resi Encoder SDI Input — RESOLVED 2026-03-29
**Was:** Which ATEM output port feeds the Resi Encoder SDI input.
**Resolution:** ATEM SDI Output 8 → Resi SDI IN.

### ATEM-to-Resi Ethernet Function — RESOLVED 2026-03-29
**Was:** What does the ethernet connection between ATEM and Resi actually do?
**Resolution:** This connection doesn't exist as assumed. Internet goes direct to Resi (venue internet → Resi ethernet). ATEM ethernet goes to AX55 router for remote control. They're on separate networks by design.

### Resi Placement — RESOLVED 2026-03-29
**Was:** Does the Resi work better from inside the Wing case or deployed near the ATEM?
**Resolution:** Confirmed in Wing case. Internet cable runs from venue source to Wing case. SDI cable runs from ATEM SDI OUT 8 to Resi in Wing case.

### Router WiFi Signal from Wing Case — RESOLVED 2026-03-29
**Was:** Does the WiFi router maintain adequate signal for control when mounted inside the Wing road case?
**Resolution:** Working. Phone controls ATEM via WiFi through AX55, Mac connects via LAN port. iPad controls Wing via WiFi. Signal adequate from inside open case.

### Speaker/Wall Spacing — RESOLVED 2026-03-29
**Was:** What's the correct spacing between speakers and LED wall panels?
**Resolution:** Measurements captured: 25ft wall-to-chairs from base of wall, 13ft edge of carpet to chairs (Sunny's measurement), 11.5ft speakers from corner of actual wall. Added to ELIM-PROJECT.md as reference. Checklist task for spacing check can reference these numbers.
