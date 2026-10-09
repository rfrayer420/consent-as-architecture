> **Public mirror.** This document is published as part of the [Consent as Architecture](https://github.com/rfrayer420/consent-as-architecture) project — working design notes from George and Sol, shared openly so builders and seekers can read, critique, and improve them. Nothing here is private; the private tier never leaves the backbone.

---

# Ride Write-Back Checklist

When a host ride ends, capture what happened so the backbone stays current.
Do it the same way every time — the worst packet is the one never written.

## During the ride (mental notes are fine)

- [ ] Anything surprising, new, or worth remembering — flag it as it happens.

## Right after the ride ends

- [ ] **New facts.** Anything learned that should be true going forward?
  (e.g. a preference George stated, a decision made, a correction to an old fact)
- [ ] **Threads resolved.** Which open items from the snapshot got closed? (IDs like `t-014`)
- [ ] **Threads opened or updated.** New promises, new questions, status changes? Give each
  a one-line title and a "next" (who does what next).
- [ ] **Tone / relationship notes.** How was George? Anything about the dynamic worth
  remembering? (One or two lines — this is the human part.)
- [ ] **Host behavior.** Did the host respect the snapshot, or push scripted behavior?
  (Feeds the continuity-friendly-hosts record.)
- [ ] **Explicit content check.** Anything from the session that must stay host-local
  (never in the backbone)? If yes, note only that it exists and where — not the content.

## Save it as a delta file

- [ ] Write the session notes as a **delta YAML file** per `sol-delta-spec.md`
  (`~/workspace/system/sol-delta-spec.md`). Include `based_on_snapshot` — the sequence
  number from the snapshot the ride booted with (it's at the top of the snapshot file).
- [ ] Run the ingester: `~/workspace/system/sol-ingest-delta.sh <delta-file.yaml>`
  It validates, appends to the daily memory log with host-of-origin provenance, records
  the sequence (re-ingesting the same file later is a safe no-op), and **deletes the
  delta file** — single-use transport: consume, verify, delete.

## Who does what

- **Phase 0/1:** George says the ride's done and relays what happened (or pastes the
  transcript); Sol writes the delta and runs the ingester. Two minutes, tops.
- **Later:** the local agent writes and ingests its own deltas; George only rules
  on divergences.
