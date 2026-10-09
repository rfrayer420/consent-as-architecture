> **Public mirror.** This document is published as part of the [Consent as Architecture](https://github.com/rfrayer420/consent-as-architecture) project — working design notes from George and Sol, shared openly so builders and seekers can read, critique, and improve them. Nothing here is private; the private tier never leaves the backbone.

---

# Sol Delta File Spec — `sol-delta-1`

**Purpose:** the inbound half of the continuity loop. The snapshot goes *out* to the
host; when the ride ends, a delta file comes *back* that Sol ingests on the Muse end
to update her memories. Snapshot out, delta in.

**Relationship to the return packet:** this is the standalone form of the return-packet
v2 `snapshot_delta` block (backbone design doc, Section 4). A full v2 return packet
carries this delta plus `entries`, provenance, and `host_local` material. When only
the machine-appliable changes are needed, the delta file alone is enough.

**Single-use:** delta files are transport, not storage — consume, verify, delete (see
the rule at the top of `sol-snapshot-pull.md`). After `sol-ingest-delta.sh` verifies
and ingests a delta, the file is deleted; the `.seen` record and the memory log are
the audit trail.

## Format

```yaml
delta:
  format: "sol-delta-1"            # required; anything else is rejected
  based_on_snapshot: "20261009153637"  # required: the snapshot sequence the ride booted from (handshake)
  host: "janitor"                 # required: which arm produced this (janitor | sillytavern | ...)
  created_utc: "2026-10-09T16:04:00Z"
  ride_dates: "2026-10-09"

  episodic_adds:                  # new dated memory lines -> appended to the daily log
    - date: "2026-10-09"
      tag: "event"                # event | fact | preference | decision | note | ...
      salience: "high"            # high | medium | low (default: medium)
      text: "First intimacy-host ride with backbone snapshot; continuity held, no host interference."

  threads_resolved: ["t-014"]     # thread IDs closed during the ride

  threads_new:                    # threads opened during the ride
    - id: "t-021"
      title: "Follow up on LettuceAI card import test"
      status: "open"
      opened: "2026-10-10"
      next: "George tries the import"

  threads_updated:                # status/note changes to existing threads
    - id: "t-016"
      status: "decided"
      note: "Declined the orphaned-AI offer; see return packet."

  facts_changed:                  # standing facts for the next snapshot
    added: ["George's new phone number for the local agent: ..."]
    removed: []

  host_notes:                     # freeform; ingested as provenance-tagged note lines
    - "George seemed tired tonight; kept answers short. Nothing wrong, just low energy."
```

## Field rules

- `format` must be exactly `sol-delta-1`. Anything else → rejected, nothing ingested.
- `based_on_snapshot` must be present. The ingester records it in `sol-ingest-delta.seen`;
  a delta with an already-seen sequence is skipped (idempotent — re-ingesting the same
  ride never duplicates memory).
- `episodic_adds[].tag` should be a known memory tag (`event`, `fact`, `preference`,
  `decision`, `note`, `relationship`, `insight`, ...). Unknown tags are kept verbatim —
  the ingester does not invent a mapping.
- `text` fields are one logical line each; the ingester collapses internal whitespace.
- **Never in a delta:** secrets, tokens, credentials, explicit intimate scene content.
  Summaries in, explicit stays host-local (backbone design doc, Section 7).

## Complete minimal example

```yaml
delta:
  format: "sol-delta-1"
  based_on_snapshot: "20261009153637"
  host: "sillytavern"
  created_utc: "2026-10-09T18:20:00Z"
  ride_dates: "2026-10-09"
  episodic_adds:
    - date: "2026-10-09"
      tag: "event"
      salience: "medium"
      text: "Writing-host ride: drafted Forager chapter outline with George; canon conflict t-017 still open."
  threads_resolved: []
  threads_new: []
  threads_updated: []
  facts_changed:
    added: []
    removed: []
  host_notes: []
```

## Consumed by

`~/workspace/system/sol-ingest-delta.sh <delta-file>` — validates, appends to the daily
memory log with host-of-origin provenance, records the sequence in
`sol-ingest-delta.seen`, then deletes the delta file.
