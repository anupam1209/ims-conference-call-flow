# IMS Conference Call Flow — Wireshark-Based Reference

A single-page, print-friendly HTML reference document that visualizes the **complete SIP signaling flow** for a 3-way conference call in a IMS deployment. The flow is **derived directly from successful `CONFSIG` + `CONFMEDIA` Wireshark captures**, not from spec docs — so use this when the documented flow and the wire don't agree.

> **Bottom line:** The spec doc shows `183 Session Progress + PRACK/UPDATE` and an `INVITE with Replaces` pattern for joining UE2/UE3. The actual capture shows `200 OK directly` after `INVITE-Conf` and a `fresh out-of-dialog INVITE + in-dialog re-INVITE` pattern. The first `SUBSCRIBE` also gets a `403 Forbidden` — that's expected and passes on retry.

---

## What's inside

`index.html` renders one large SVG sequence diagram plus supporting editorial content:

- **Lifelines:** `UE1`, `UE2`, `UE3`, `P-CSCF`, `S-CSCF`, `TAS`, `CONF-AS-SIG`, `CONF-AS-Media`
- **Phases covered:**
  1. REGISTER / authentication
  2. SUBSCRIBE (including the expected first `403 Forbidden` and retry)
  3. UE1 originates `INVITE-Conf` → `200 OK` directly (no 183 / PRACK / UPDATE)
  4. Conference room creation via internal `CONF-AS-SIG ↔ CONF-AS-Media` REST/HTTP calls
  5. Adding UE2 — fresh out-of-dialog `INVITE` from CONF-AS-SIG
  6. Adding UE3 — fresh `INVITE` + in-dialog `re-INVITE` (not `INVITE w/ Replaces`)
  7. `NOTIFY sipfrag` (100 Trying / 200 OK) updates, `REFER` handling
  8. `BYE` teardown with `STR` to PCRF on the Rx interface for each leg
  9. Final state — conference terminated, room destroyed, all dialogs cleared
- **Doc-vs-capture call-outs** highlighting where the spec diverges from observed behavior
- **Debug cards** with quick-reference fields for live troubleshooting

## Architecture (actors)

| Node              | Role                                                                                                  |
| ----------------- | ----------------------------------------------------------------------------------------------------- |
| `UE1 / UE2 / UE3` | Conference participants                                                                               |
| `P-CSCF`          | Proxy CSCF — also sends `STR` to PCRF on teardown (Rx)                                                |
| `S-CSCF`          | Serving CSCF — primary signaling path: `UE1 → P-CSCF → S-CSCF → CONF-AS-SIG`                          |
| `TAS`             | Telephony Application Server                                                                          |
| `CONF-AS-SIG`     | Conference signaling AS                                                                               |
| `CONF-AS-Media`   | Conference media AS — talks to `CONF-AS-SIG` over **internal HTTP/REST** (not visible on SIP capture) |

## How to use

```bash
# Just open it — no build step, no dependencies.
open index.html               # macOS
xdg-open index.html           # Linux
start index.html              # Windows
```

Or serve it locally if you prefer:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

The page is also tuned for printing / PDF export (`@media print` strips backgrounds, avoids breaking the diagram across pages).

## Tech / dependencies

- Pure **HTML + inline CSS + inline SVG**. No JS, no framework, no bundler.
- Web fonts via Google Fonts: `Fraunces`, `Inter`, `JetBrains Mono` (requires internet on first load; falls back to system fonts otherwise).

## When to read this vs. the spec doc

Read **this** when:

- You're staring at a Wireshark `.pcap` and the documented flow doesn't match what you see.
- You're debugging conference setup, add-participant, or teardown on a live IMS box.
- You need to know which messages are SIP-on-the-wire vs. internal HTTP between `CONF-AS-SIG` and `CONF-AS-Media`.

Read **the spec doc** when:

- You need standards-compliant behavior or interop reasoning.
- You're reviewing what _should_ happen vs. what this deployment actually does.

## File layout

```
.
├── index.html   # The whole reference document (HTML + CSS + SVG)
└── README.md    # This file
```

## Status

`v1` — based on a single set of successful 3-way conference captures. Update the SVG and the doc-vs-capture call-outs as new captures (failure modes, larger conferences, codec re-negotiation, etc.) are collected.

---

_For IMS debugging reference only._
