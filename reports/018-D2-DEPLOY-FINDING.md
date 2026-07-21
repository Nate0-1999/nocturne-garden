packet:            D2 deploy — live finding (human gate)
session:           human+claude / 2026-07-21
status:            RESOLVED — preflight fixed at the human gate (spine 335539f, DECISIONS 018); ready to arm
what exists now:   The D2 billing-breaker FUNCTION is built, unit-tested
                   (76 pass), and correct. Its deploy.sh preflight, however,
                   refuses to arm against a STANDARD GCP project. Three
                   refusals were hit live, in order:
                   (1) budget ownershipScope — human-caused: the $100 budget
                       created earlier in the session was project-scoped, not
                       BILLING_ACCOUNT-scoped. Fixed by creating a new
                       BILLING_ACCOUNT-scoped $100 budget
                       (82be62a3-3233-4b25-9b44-a239c9a3f6fd); the old
                       project-scoped budget (304a09a7-…) remains, alerts only.
                   (2) case-sensitive deployer email — genuine code bug: the
                       preflight compared IAM members to the deployer with
                       exact string equality, so `NDOswalt1@` (as stored in
                       billing IAM) != `ndoswalt1@` (as authenticated). FIXED
                       in spine commit 2233df2 / DECISIONS 017 (casefold the
                       email portion only).
                   (3) over-strict project-IAM audit — DESIGN defect: the
                       preflight enumerates every project principal holding
                       pubsub.topics.publish (or billing perms) and refuses
                       unless each is on a narrow allowlist. It thereby
                       refuses to arm against Google's OWN default identities
                       that exist in essentially every project:
                       `service-<pn>@containerregistry.iam.gserviceaccount.com`
                       and the default `<pn>-compute@developer.gserviceaccount.com`
                       holding roles/editor.
finding:           The breaker is over-engineered. Asked to "build a billing
                   circuit breaker," the relay produced a preflight so paranoid
                   it audits the whole project IAM graph and will not arm
                   against a default GCP posture — thoroughness crossed into
                   impracticality. This is confined to D2's deploy tooling; the
                   product (spine 158/160, harness 220/220, live contract +
                   10 UI states verified) is clean. Useful signal: the relay
                   errs maximally paranoid on security-flavored charges.
decision:          Owner chose to SIMPLIFY the preflight rather than harden the
                   project's IAM. Fixed directly at the human gate (no relay
                   packet needed): the project-IAM audit now recognizes Google's
                   default identities (Container Registry agent, default Compute
                   SA, gcp-sa-* agents) and trusts Google project service agents
                   except direct billing detach — while still refusing genuine
                   anomalies (users/groups/custom SAs). Every resource-level and
                   billing-account guard is preserved. Live project-IAM audit is
                   clean; 76 D2 + 160 spine tests pass. The human re-runs --apply
                   (unchanged command). See spine DECISIONS 018.
notes (resolved):  The fix (spine 335539f) kept every RESOURCE-level check (topic
                   publish policy = Google budget alerter only; function
                   private + no-retry; exact detach-role binding; budget shape;
                   trusted deployer via the 2233df2 casefold). DEMOTE the
                   project-wide principal audit (validate_role_access over
                   pubsub/billing publishers) from a hard refusal to an
                   informational WARNING that lists broad/default identities
                   but does not block. Add tests: a default-posture project
                   (containerregistry agent + default compute Editor SA
                   present) PASSES; tampering with the topic/function/detach
                   role still REFUSES. Destructive --apply stays human-only.
