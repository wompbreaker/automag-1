# CANoe CAN/LIN Gateway Simulation — Implementation Plan

Build a CANoe restbus simulation with a CAN-to-LIN gateway ECU. Setup two networks, load all nodes/databases, create analysis desktops, integrate panels, implement gateway logic using built-in functions, and optionally add testing + bidirectional gateway for extra credit.

**Total points available: 25 regular + 10 extra = 35 pts**

---

## Phase 1: Project Setup & Restbus Simulation (3 pts — Req #1)

1. **Create a new CANoe configuration** — File → New Configuration. Save in a well-structured project folder.
2. **Add CAN network** — In Simulation Setup, add CAN channel 1. Assign `CANdb.dbc` as its database.
3. **Add LIN network** — Add LIN channel 1. Assign `LINdb.ldf` as its database.
4. **Import system variables** — Load `can_lin.vsysvar` via Environment → System Variables. These bridge panels and CAPL nodes.
5. **Verify**: Both networks visible in Simulation Setup with databases loaded.

---

## Phase 2: Node Configuration (4 pts — Req #5)

6. **Assign all 8 CAN nodes** in Simulation Setup under the CAN network, attaching their respective `.can` files:
   - `LockingSystem.can` (vehicle lock/unlock control)
   - `Logging.can`, `Motor_back.can`, `Motor_head.can`, `Motor_horizontal.can`, `Motor_vertical.can`, `Seatheating.can`, `Seatsensor.can`
7. **Compile all nodes** — Build All (F7). Fix any path or database reference errors.
8. **Verify**: Start measurement briefly — zero errors in the Write window.

---

## Phase 3: Panels Integration (4 pts — Req #4)

> *Can be done in parallel with Phase 4 & 5.*

9. **Load provided panels**: `console.xvp` (CAN Info — unlock, ignition, seat 1/2 buttons), `DriverID.xvp`, `Seatdisplay.xvp` (seat position display).
10. **LIN Information panel** — Check if a LIN control panel is among the Canvas materials. If not, create one with controls mapped to LIN signals for individual seat functions (back, head, horizontal, vertical, heating) via the system variables.
11. **Verify panel bindings** — Click controls during measurement, observe signal changes in Trace.

---

## Phase 4: Analysis Desktop (4 pts — Req #3)

> *Can be done in parallel with Phase 3 & 5.*

12. **Create "Analysis" desktop tab** in CANoe.
13. **Add 2 Statistics windows** — one filtered for CAN, one for LIN (bus load, frame count, errors).
14. **Add 1 Data window** — showing signal values from both CAN and LIN databases.
15. **Add 2 Trace windows** — one filtered for CAN messages, one for LIN messages.
16. **Tile all 5 windows** cleanly so all are visible simultaneously.

---

## Phase 5: Tidiness & Organization (2 pts — Req #2)

> *Can be done in parallel with Phase 3 & 4.*

17. **Organize desktops**: "Simulation" desktop (panels), "Analysis" desktop (analysis windows). Remove empty/default desktops.
18. **Organize files** into subfolders:
    - `/Databases/` — `CANdb.dbc`, `LINdb.ldf`
    - `/Nodes/` — all `.can` files
    - `/Panels/` — all `.xvp` files + `/Bitmaps/` subfolder
    - `/SystemVariables/` — `can_lin.vsysvar`
    - `/Gateway/` — gateway CAPL node(s)
19. **Clean up** — Remove unnecessary windows or default clutter.

---

## Phase 6: Gateway Implementation (4 + 4 = 8 pts — Req #6 & #7)

> **CRITICAL PATH** — This is the core of the project. Req #7 mandates using built-in functions/selectors.

20. **Preferred approach — CANoe built-in Gateway block**: In Simulation Setup, add a Gateway/Signal Routing block between CAN and LIN. Map signals from `LockingSystem` CAN frames to corresponding LIN motor frames. This directly satisfies Req #7.

21. **Alternative — CAPL Gateway node** (if the Gateway block is unavailable in your CANoe version):
    - Create `Gateway.can` node.
    - Use `on message` handlers for CAN frames from `LockingSystem`.
    - Use **signal selectors** (`$SignalName`) and built-in functions (`getSignal()`, `setSignal()`) to read CAN signals and write to LIN signals — this satisfies Req #7.
    - Use `output()` / `linTransmitSlave()` to push data onto the LIN bus.

22. **Signal mapping**: Route CAN seat position commands (from `LockingSystem` stored profiles triggered by buttons 1/2) → LIN motor commands (back, head, horizontal, vertical, heating). The `can_lin.vsysvar` system variables likely define the intermediate mapping — study them first.

23. **Verify**: "Open Car" → "Power" → Button 1/2 on CAN panel → CAN Trace shows traffic → LIN Trace shows forwarded signals → Seat Display updates.

---

## Phase 7 (Extra): Bidirectional Gateway — LIN → CAN (5 extra pts)

24. **Extend gateway** to also route LIN signals back to CAN: add `on linMessage` handlers or extend the Gateway block to capture LIN responses and map them back to CAN frames.
25. **Verify**: Adjust seat via LIN Information panel → CAN Trace and Data window show updated signals.

---

## Phase 8 (Extra): Test Environment & Report (5 extra pts)

> *Independent of Phase 7 — do whichever you're more confident in first.*

26. **Create Test Environment** — Test → Test Setup → Add Test Environment.
27. **Write test cases** (CAPL or XML test module):
    - **TC1**: CAN lock message → LIN motor command appears within timeout
    - **TC2**: Seat position 1 → all motors reach correct LIN values
    - **TC3**: Seat position 2 → different seat profile verified
    - **TC4**: *(if bidirectional)* LIN adjustment → CAN signal update
    - **TC5**: 30-second clean measurement run (no errors)
28. **Run tests**, generate HTML/PDF report, save in `/TestReports/`.

---

## Verification Checklist

| # | Requirement | Points | How to verify |
|---|-------------|:------:|---------------|
| 1 | Restbus with CAN + LIN + databases | 3 | Simulation Setup shows both networks with `.dbc`/`.ldf` |
| 2 | Tidiness | 2 | Organized desktops, clean file structure |
| 3 | Analysis desktop | 4 | 2 Statistics + 1 Data + 2 Trace windows present |
| 4 | Panels working | 4 | CAN panel unlocks/ignition/seat select; LIN panel controls seat; Seat Display shows position |
| 5 | All nodes compile | 4 | F7 Build All → 0 errors, measurement starts clean |
| 6 | Gateway CAN→LIN | 4 | Button press on CAN panel → LIN Trace shows motor commands |
| 7 | Built-in functions used | 4 | Gateway uses `$signal` selectors, `getSignal()`/`setSignal()`, or Gateway block |
| E1 | Test report | 5 | Test module exists, cases pass, report generated |
| E2 | Bidirectional gateway | 5 | LIN panel changes → CAN Trace updates |
| | **Total** | **35** | |

---

## Key Decisions & Notes

- **"Built-in functions or selectors" (Req #7)** = use CAPL `$` signal notation, `getSignal()`/`setSignal()`, or the CANoe Gateway block — **not** raw byte-level `this.byte()` manipulation.
- The `can_lin.vsysvar` system variables are the intended bridge between CAN and LIN — study their structure first to determine the exact signal mapping before writing gateway code.
- **Recommended order**: Phase 1 → 2 → (3, 4, 5 in parallel) → 6 → 7 → 8. Phase 6 is the critical path; budget the most time for it.
- The LIN Information panel may need to be created if not among the provided `.xvp` files (only `console.xvp`, `DriverID.xvp`, `Seatdisplay.xvp` are listed). Check Canvas materials carefully.
- Phases 7 and 8 (extra credit) are independent of each other.
