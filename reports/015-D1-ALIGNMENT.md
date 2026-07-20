packet:            D1 — GCP deploy & remote verification alignment
session:           human+codex / 2026-07-20 / d1a4
status:            RETURNED_TODO
what exists now:   Live read-only audit confirms project n8-memory-palace is
                   ACTIVE and billed; Cloud SQL n8-memory-palace-db is
                   RUNNABLE, but only its default postgres database/user
                   exist. Cloud Run, Artifact Registry, Secret Manager, and
                   GCS resources are absent. PLAN now makes D1 relay-owned
                   under an exact cloud mutation fence; BOARD is reset TODO.
deviations:        SPEC D.2 033's premature "D1 executed" factual claim stays
                   immutable; D.2 045 explicitly supersedes only that status
                   clause and records the scoped delegation. This is an
                   operations/truth repair, not product-contract behavior or a
                   version bump.
                   D2 deployment and post-H5 human use remain human boundaries.
evidence:          Read-only gcloud inspection with explicit project flags:
                   project/billing ACTIVE; budget present; SQL Postgres 16
                   RUNNABLE in us-central1; only postgres DB/user; no Run
                   service, AR repository, Secret Manager setup, or bucket.
                   Local proxy and Docker are available. A local linux/amd64
                   dry build proved the legacy builder cannot cross-build and
                   Docker Buildx is absent; PLAN now names the single allowed
                   plugin/config repair. No image was pushed. Secret values
                   were never printed; required token/broker keys are present
                   in the ignored env, which this alignment tightened to 0600.
notes to the next agent:  This report supersedes reports 006–014 only where
                   they call D1 human-owned. Run the Boot Sequence, claim D1,
                   and obey its literal grant. Aligned product heads are Spine
                   3b4b80314bb2376a961168d56a2114287546ad902 and Harness
                   547f3210cba39154636688c38de28690db233766. Ambient gcloud
                   currently points at another project: every command must name
                   n8-memory-palace and us-central1. Preserve existing project,
                   budget, and SQL instance; perform no deletes. Create the
                   spine DB/user, protect and migrate it to 0002, use Secret
                   Manager plus dedicated spine-runtime identity, create only
                   the regional spine registry, install/configure only Buildx
                   if still absent, locally build/push the current Spine SHA
                   without Cloud Build, and deploy the exact named service.
                   Then leave redacted remote evidence and the URL in the
                   handoff. H5 remains blocked until D1 is actually DONE.
