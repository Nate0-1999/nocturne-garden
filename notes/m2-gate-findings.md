# M2 gate-day findings (owner spin, started 2026-08-05)

For the M2X scout/checklist and the M2J judge. Live findings from the
owner's first assembled-product startup:

1. VERSION-SKEW UX IS SILENT AND HOSTILE: a daemon newer than its remote
   Palace produced 200 lines of bare 503s and a timeout — no message
   ever said "your Palace is running older software than this app
   expects; run `nocturne deploy` (owner cloud) or update." The daemon
   authenticated the spine successfully, so it COULD have compared
   versions/contract and said so in one sentence. Wants: an explicit
   version handshake at startup with a plain-language, garden-free
   remedy line. (Root cause of the incident itself: the entire M2 wave
   was never deployed to the owner cloud — lawful, since no packet held
   cloud authority after F011's single-use grant; the owner is running
   `nocturne deploy` himself, first use with M2N receipts.)
2. IDLE-EXTRACTION RETRY NOISE: the extraction sweeper hit the missing
   endpoint (404) and logged a full traceback with "will retry" against
   a Palace that can never satisfy it. Retry loops against contract-
   level (not transient) failures should back off and quiet down after
   first diagnosis.
3. PHRASING: init's "Palace bearer:" prompt — "Palace access token"
   reads aloud better (vision §18); minor.
