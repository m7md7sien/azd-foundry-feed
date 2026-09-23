# Candidate 39: draft source and acceptance checklist

**Preparation only. Not built, published, or approved for publication.** The
exact source SHA is awaiting approval. Build 38 remains the published Latest
release, and its registry, packages, and historical findings are unchanged.

The live release/tag inventory and Latest registry were checked on 2026-09-23.
Build 38 currently carries evaluations `1.0.38-beta` and dataset
`1.0.0-beta.26`; no build 39 release or tag was present. The following names are
provisional, not reserved, and must be checked again before build/publication.

| Item | Draft value |
| --- | --- |
| Planned tag | `extensions-2026-09-23-39` |
| Evaluations | `1.0.39-beta` |
| Dataset | `1.0.0-beta.27` |
| Required azd | `>=1.33.0`, subject to checking the approved manifests |
| Source commit for both extensions | Pending exact integration SHA and BUILD approval |
| Registry/archive hashes | Not generated |
| Publication and Latest promotion | Separate pending gates |

No install URL is offered until real assets exist. Do not use this document as
evidence that a build 38 issue has been fixed in a published package.

## Planned scope, not shipped fixes

The coordinator is integrating follow-up work for:

- Single-file downloads in both `azd ai dataset` and `azd ai eval dataset`.
- Observed conversation-output counts, kept distinct from requested simulation
  settings and other service evaluation counts.
- Human-readable handoff guidance for unattended runs.
- Truly unregistered local files whose remote version listing is empty.
- Actual returned rubric dimension data and preservation of nested result/sample
  fields, without deriving missing values from rubric definitions.
- Human detail lookup identifiers while preserving JSON service identities.

Each claim must be checked against the frozen integrated source and the exact
new packages. Build 38's known issues remain documented in
[its acceptance record](./Build-38-Verification.md). A source fix, passing
worker test, or earlier candidate run does not by itself verify build 39.

## Packaging and publication gates

Both extensions must come from one approved immutable commit, fetched into a
new artifacts-owned `b39-<source-sha-prefix>` directory. Do not reuse a previous
`bin`, bundle, registry, or installed development extension.

Keep source `go.mod`, `go.sum`, and changelogs unchanged. Version overrides are
limited to each extension's `extension.yaml` and `version.txt` in isolated
staging. Use the established `azd x build --all --skip-install` and `azd x pack`
format: six archives per extension, each containing `extension.yaml` and its
platform-named entrypoint. Windows/macOS use ZIP and Linux uses tar.gz.

| Gate | Status |
| --- | --- |
| Exact source SHA and BUILD approval | Pending |
| Combined source checks and hosted race suites at that SHA | Pending |
| Twelve archive layouts, manifests, entrypoints, SHA256 hashes, extracted binary bytes and VCS/platform metadata | Not started |
| Fresh isolated Windows bundle installation and exact runtime versions | Not started |
| Candidate-specific regression/live acceptance | Not started |
| Separate PUBLISH approval | Pending |
| Complete new draft: individually uploaded assets, no overwrite, literal `registry.json` last | Not started |
| Non-Latest publication and anonymous pinned registry/all-asset verification | Not started |
| Fresh published-source installs and final Linux/Windows CLI/source CI with exact pins | Not started |
| Latest promotion, anonymous stable URL, and fresh stable-source install | Not started |

The release should contain 12 platform archives, `source-provenance.json`,
`SHA256SUMS`, and the literal `registry.json`. Preserve every older release and
asset. Synchronize the repository registry/docs only with real published assets.
Report publication separately from Latest promotion.

## Candidate-specific acceptance targets

These are pending targets, not pass claims. Record exact source, versions,
package hashes, commands, and sanitized outcomes for each result.

| Area | Required evidence |
| --- | --- |
| Standalone and embedded dataset download | The affected single-file `--output-file` path writes the exact expected bytes in both namespaces; existing directory downloads and invalid-input safeguards remain intact. |
| Truly unregistered local files | Exercise a direct declaration and a dataset override with confirmed remote absence and bounded rows. Verify no implicit dataset publication. Keep authorization/transport failures distinct from absence. |
| Registered dataset identity and caps | Retain service-issued version identity and explicit positive-cap rejection. Do not replace registered data with an inline copy or automatically publish a temporary subset. |
| Observed conversation outputs | Distinguish observed output/conversation/turn counts from requested seeds, repetitions, turn ceilings, and evaluation verdict totals. Do not treat missing counters as zero or synthesize unsupported counts. |
| Rubric detail and raw fields | Display dimension values only when actually returned. Verify applicable scores, weights and reasons against sanitized retained service evidence; preserve nested JSON fields rather than silently dropping them. |
| Detail lookup identifiers | A human-displayed lookup ID works with the detail command; JSON retains the service identity. |
| Unattended handoff | Confirm the human next-step hint and runnable reattachment command without contaminating JSON output or resubmitting work. |
| Existing behavior | Retain the prior semantic preflight/no-mutation, static-versus-simulation, pinning, cap-zero, and JSON-output guarantees using evidence tied to the candidate or an explicitly justified source boundary. |

Use small owned fixtures and bounded model calls; do not regenerate large
datasets merely to replace provenance. Delete only resources proved to belong
to the test. Prior build 38 runs and its 64-check-per-platform CI remain
historical evidence, not new build 39 executions.

## Remaining qualifications

Do not infer an all-scenarios fresh-user pass, authenticated live-cloud CI pass,
final GA simulation contract, privacy signoff, or service-fix deployment from
packaging or source checks. Each needs its own evidence. Keep unsupported or
unverified behavior explicit.

Public files must contain no credentials, private prompts or handoff material,
customer data, or full raw service responses. Publish sanitized field/shape
comparisons and precise verification scope instead.
