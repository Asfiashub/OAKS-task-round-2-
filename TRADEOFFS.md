# SheetalTrack — Product Notes

## The core problem I designed for

The brief's real pain point isn't "log some data" — it's that paper
notebooks give each agent their own private, editable record, so when milk
sours nobody can agree on whose batch caused it or how long it actually
sat. A digital form that only one agent can see doesn't fix that; it just
digitizes the same dispute. So the design priority was: **one shared,
tamper-evident timeline that every agent looks at**, with the system (not
the agent) doing the risk judgment.

## Key product trade-offs

**Shared backend over offline-first.** I used Firestore instead of
per-device local storage specifically so agents across a co-op see the same
batch list in real time, with server-side timestamps that can't be
backdated from a phone's clock. The trade-off is that this version needs a
working internet connection to log or view batches — a real risk in some
villages. For a production version I'd add a local write queue (IndexedDB)
that syncs when connectivity returns; I scoped that out of the 48-hour
build to keep the shared-timeline guarantee solid rather than half-building
two data paths.

**No login system, by design.** Agents type a name once and it's
remembered on the device — there's no password or account. This matches
how a paper notebook actually gets used in the field (fast, no friction,
handed between people), and it's still enough to attribute every entry to
a name. The trade-off: nothing stops someone from typing a different name
than their own. That's an acceptable gap for a 48-hour prototype aimed at
day-to-day accountability, not fraud-proofing; a real deployment would add
lightweight phone-number verification per agent.

**Temperature bands instead of a raw number field.** Most collection
agents won't have a calibrated thermometer on them twice a day. Rather than
force a precise number (which invites made-up precision), the form uses
five qualitative bands plus an explicit "no thermometer — assume ambient"
option, so the tool degrades gracefully instead of blocking the log entry.

**A simplified spoilage model, stated as an assumption, not a certified
standard.** Remaining safe time is estimated from a lookup table (chilled
milk ≈ 24h safe window, down to ~1h for milk dropped off very hot), scaled
against elapsed time since drop-off. Real microbial spoilage kinetics are
more continuous and depend on initial bacterial load, not just temperature.
I chose a lookup table over a more "accurate"-looking formula because a
fake-precise number would be worse than an honest, coarse one — the write-up
is explicit that a real deployment should have these thresholds reviewed
against FSSAI / local dairy cooperative guidelines before they're trusted
operationally.

**Append-only events, not editable records.** A batch's status changes
(created, poured, disputed) are stored as an appended event list rather
than overwritten fields. Nobody can quietly edit history — only add to it.
This was the single highest-leverage trust decision in the build: it's
what actually resolves "whose batch spoiled" arguments, because the full
sequence of who-did-what-when is always visible, not just the current
state.

## Edge cases handled

- **No thermometer available** — explicit fallback band, not a blocked
  form.
- **Milk already at risk when the agent wants to pour** — pouring a batch
  flagged "At risk" or "Spoiled" requires a typed reason before it's
  accepted, and that reason is permanently attached to the batch. This
  turns a future argument ("why did you pour bad milk in?") into something
  answered by the record instead of memory.
- **Disagreement about a specific batch** — a dedicated "Dispute" action
  logs the disagreement without deleting or altering the original entry,
  so both the original claim and the dispute are visible side by side.
- **Multiple agents logging at the same time** — Firestore assigns each
  batch its own document ID, so simultaneous submissions from different
  phones can't collide or overwrite each other.
- **Page refresh / phone restart mid-shift** — because data lives in
  Firestore, not the page, refreshing or closing the browser doesn't lose
  any batches; only the un-submitted "add batch" form resets.
- **Idle countdowns going stale** — remaining time is recomputed from the
  stored collection timestamp on every render (every 15s) rather than
  ticking a counter down client-side, so it can't drift out of sync if a
  phone's tab is left open for hours.

## Trust mechanisms, summarized

1. **Shared visibility** — every agent sees every batch, in real time, not
   just their own.
2. **Server timestamps** — the clock the risk calculation uses comes from
   Firestore's server, not a phone that could be set wrong (by mistake or
   otherwise).
3. **Append-only audit trail** — every action is added to a batch's
   history, never overwrites it.
4. **System-computed risk, not self-reported** — the "Safe / Watch / At
   risk / Spoiled" flag is calculated from timestamp + temperature band by
   the app, removing the incentive to just say a batch is fine.
5. **Mandatory reason for risky overrides** — pouring a flagged batch
   requires a written justification captured at the moment of the
   decision, not reconstructed afterward when a dispute happens.

## Additions: tank monitor, SMS summary, farmer roster, sample data

**Vat capacity monitor.** The vat has its own state (`VAT_CAPACITY_L`,
currently 300 L — a placeholder for the hub's actual tank size) and its own
countdown, separate from any individual batch's countdown. The clock starts
on the *first* pour into an empty vat and resets only when an agent taps
"Tanker arrived — empty vat." I deliberately used an explicit reset action
rather than a calendar-day boundary, because collection cycles in the field
don't reliably align to midnight — a hub might run two full cycles in a
day, or one cycle spanning two days if a tanker is late. Historical batch
records are never touched by the reset; only the running fill/countdown
resets, so "who poured what" stays intact even across cycles.
`VAT_SAFE_HOURS` (6h) is a planning assumption, stated as such — the same
caveat as the per-batch model applies: it should be checked against real
tank performance before being trusted operationally.

**SMS-ready summary, not real SMS.** The brief's "no paid APIs" constraint
rules out an actual SMS gateway (Twilio etc.), so the generator composes
plain text and hands it off via a `sms:?body=` link (opens the phone's own
messaging app with the text pre-filled) plus a copy-to-clipboard fallback
for devices where that link doesn't behave predictably. The agent still
picks the driver's number — the tool doesn't guess or store it.

**Fixed farmer roster with IDs, plus an escape hatch.** Farmers now come
from a dropdown with stable IDs (`F001`, `F002`, …) instead of free-typed
names, which directly serves the trust goal: "Ramesh" typed two different
ways can no longer become two different people in a dispute. But a rural
co-op's farmer list changes (new farmers, one-off deliveries from someone
passing through), so I kept an "Other" option that falls back to free text
rather than blocking the log entry — that batch just won't have a stable
ID. A production version would let agents add a new farmer to the shared
roster on the spot instead.

**Pre-loaded sample data (15 deliveries).** Local demo mode seeds this
automatically. Shared/Firestore mode shows a one-time "Load sample data"
banner instead of auto-writing to a real project on every page load — a
judge (or anyone) opening the live URL repeatedly shouldn't silently
duplicate 15 fake batches into a real co-op's database. The seed data
intentionally covers every state (safe/watch/risk/spoiled, a normal pour,
an override pour, a dispute, a partially-full vat) so the whole feature set
is visible without needing to manually create each case.

## What I'd do next with more time

- Tighten the Firestore security rules beyond the current "anyone with the
  project's API key can read/write" test-mode-equivalent rule — e.g.
  requiring valid document shape on create and blocking edits to
  `farmer`/`liters` after creation, so the append-only guarantee is
  enforced by the database itself and not just by app-code convention.
- Offline queueing (IndexedDB + background sync) for patchy village
  connectivity.
- Lightweight per-agent verification (SMS OTP) instead of a free-text name.
- Photo attachment on dispute/override events for visual evidence.
- Replace the lookup-table spoilage model with one calibrated against real
  dairy cooperative data, ideally per-region (ambient temperature varies a
  lot across India).
- A simple daily summary export (CSV) for the co-op's records/accounting.
