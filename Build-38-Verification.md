# Candidate 38: source and acceptance checklist

**Not published. Corrected source and scoped package acceptance passed; publication pending.** The
earlier `9a27cd20` attempt was withheld for preventable preflight mutations.
The new source below includes the semantic preflight fix and has fresh
package verification. Publication still requires an explicit release decision.
The rolling feed still points to build 37. Do not describe this candidate as
verified from baseline evidence or from the presence of these instructions.

| Item | Candidate |
| --- | --- |
| Planned release tag | `extensions-2026-09-23-38` |
| Evaluations | `1.0.38-beta` |
| Dataset | `1.0.0-beta.26` |
| Required azd | `>=1.33.0` |
| Source commit for both extensions | [`e6f86f9605511c8448bff8114275ac1a160cf479`](https://github.com/m7md7sien/azure-dev/commit/e6f86f9605511c8448bff8114275ac1a160cf479) |
| Publication decision | Not yet approved; earlier attempt's results are not this source's acceptance |

Package-only version overrides are applied in an isolated checkout.
Source dependency manifests and changelogs are unchanged. Each extension is
built for Windows, Linux, and macOS, on amd64 and arm64. A built archive is not
proof that it has been executed on that platform.

## Included changes, not upstream merge claims

The source combines the original
[Azure/azure-dev#10113](https://github.com/Azure/azure-dev/pull/10113),
[Azure/azure-dev#10116](https://github.com/Azure/azure-dev/pull/10116), and
[Azure/azure-dev#10102](https://github.com/Azure/azure-dev/pull/10102)
with the following
[incremental source changes](https://github.com/m7md7sien/azure-dev/compare/0361f347eb28c5f2e756225115318393b1dad28a...e6f86f9605511c8448bff8114275ac1a160cf479):

- Validate dependencies before publication; retain successful versions and print
  recovery information after a partial failure.
- Validate local seed semantics, target query columns, and rubric parameter
  bounds before dependency reconciliation.
- Preserve service-issued registered dataset version identity, including for
  declared local files that have already been published.
- Scaffold static conversations or simulation explicitly, keep model roles
  separate, and reject invalid simulation limits through the actual config loaders.
- Separate requested simulation settings from observed results, and filtered
  page counts from full-run totals.

These are bundled changes, not statements that the PRs have merged upstream.
Deferred deploy-hint and meta-package changes are not included.

## Candidate authoring

Use the common environment and unique-resource setup in
[Bugbash-Instructions.md](./Bugbash-Instructions.md). These new flags require
candidate 38; they are not build 37 instructions.

For completed transcripts, each JSONL row holds a `messages` array. Static mode
must not invoke an agent:

```bash
azd ai eval init --conversation-mode static --dataset ./evals/datasets/<you>-recorded.jsonl --judge-model gpt-4.1-nano --evaluator builtin.task_completion --name <you>-static --no-prompt
azd ai eval create <you>-static
azd ai eval run start --eval <you>-static --no-prompt
```

For simulation, each seed row holds `test_case_description` and optionally
`desired_num_turns`, not `query`/`response`. Use independently deployed simulator
and judge models:

```bash
azd ai eval init --conversation-mode simulation --dataset ./evals/datasets/<you>-seeds.jsonl --target support-agent --simulation-model gpt-4o-mini --judge-model gpt-4.1-nano --evaluator builtin.task_completion --num-conversations 1 --max-turns 2 --name <you>-simulation --no-prompt
azd ai eval create <you>-simulation
azd ai eval run start --eval <you>-simulation --no-prompt
```

Static mode rejects `--target` and simulation-only flags. Simulation implies
dataset source and conversation level, requires an agent and simulator model,
and never substitutes the generation or judge model for that simulator.
Conversations per seed accept 1 through 5; maximum turns accept 1 through 20.
Omitting maximum turns preserves the service default. Explicit zero is invalid
in both flags and authored YAML, including the production `$ref` loading path.

The full YAML examples in the common instructions remain useful for deliberate
authoring beyond the scaffold. `init` stays add-only and does not generate data,
invoke an agent, or silently change existing evals. Generation declares artifacts;
follow its printed init/recovery commands rather than rerunning successful jobs.

## Important change from build 37: registered caps

| Input | Candidate 38 behavior to verify |
| --- | --- |
| Registered dataset, including a published `file:` declaration | Send the service-issued version identity. Never silently fall back to inline rows after lookup, authorization, or missing-identity errors. |
| Positive cap on a registered dataset | Reject explicitly because the current `file_id` source has no supported subset option. Remove the cap or deliberately publish a smaller dataset. |
| `--max-samples 0` | Explicitly overrides a configured positive cap. This differs from build 37. |
| Genuinely unregistered local file | Inline rows and positive caps remain available after the service confirms the dataset is absent. |
| Source-backed run or rerun selected by eval ID | Reject an explicit sample-cap flag that cannot change that source. Use trace-source bounds or select response IDs instead. |
| Simulation | Reject sample caps; use a small seed dataset, conversations per seed, and maximum turns. |

Do not use a different sample cap to distinguish two registered-dataset evals
in the CI scenario. Give the second eval a different dataset or evaluator.
No temporary subset dataset is automatically published.

## Acceptance record

| Gate | Status |
| --- | --- |
| Combined source checks at the pinned SHA | Parent-reported pass: build, full short suite with `NO_COLOR=1` and no skips, tagged vet, zero lint issues, and clean `go fix` diff |
| Source race-test run | [Evaluation command tests failed with a data race; dataset race tests passed](https://github.com/m7md7sien/azure-dev/actions/runs/35798951876). This is not passing package CI evidence. |
| Both extensions built from the pinned SHA | Passed in fresh source-specific staging; dependency manifests unchanged |
| Twelve archive layouts, manifests, entrypoints, SHA256 checks | Passed, including extracted binary VCS revision/platform and matching build bytes |
| Fresh isolated Windows installation and versions | Passed with azd 1.33.0 and exact version JSON |
| Matching-candidate live scenarios | Passed 17/17 measured negative cases without dataset-version/private-state changes; independent evaluator/eval absence checks; bounded one-seed simulation passed |
| Actual Linux CI run and installed versions | Pending |
| Anonymous pinned registry and all archive downloads | Pending publication |
| Anonymous rolling Latest registry | Still build 37 |

The earlier `9a27cd20` source could publish dependency assets for some invalid
seed/query inputs and was not approved for publication. The new source checks
non-whitespace seed descriptions, positive whole-number `desired_num_turns`,
turn ceilings, seed/message mode mixing, target query fields, whole-number rubric
weights from 1 to 10, and numeric pass thresholds from 0 to 1. The verifier's
expanded 17-case matrix exercised the exact new packages and confirmed rejection
without dataset-version or private-state changes. Independent checks found no
created custom rubric/eval resources for those cases. This closes the measured
preflight blocker; it is not a claim that every possible invalid input was tested.

Both new bundle hashes and installed Windows binary hashes matched the packaging
manifest, with exact JSON versions `1.0.38-beta` and `1.0.0-beta.26`.
The new package's one-seed run
`evalrun_b8d8f6d62a624d9f97e9f2ba2e658f66` completed with one passed evaluation
and zero errors in 20 seconds. No new 15-row generation was needed.

**Dataset download issue confirmed on the earlier `9a27cd20` package of
`1.0.0-beta.26`:** for a CLI-uploaded
single-file JSONL dataset, `download --output-file` exits 1 saying the dataset
holds one file but has no single path. The tested workaround is
`download --output-dir <directory>`: it downloaded one file whose SHA256 matched
the uploaded bytes. In this case `show` reports `isSingleFile: true` and
`type: uri_file`, while download JSON reports `singleFile: false`.
This discrepancy is a known issue, not evidence that the dataset is corrupt.
The new source changes evaluation preflight only, not dataset download code.
Other dataset shapes and output paths are not all verified.

**Earlier attempt only:** the verifier installed both `9a27cd20` local bundles, matched their SHA256 hashes
and both installed Windows binary hashes, and confirmed versions with azd
1.33.0. Package run `evalrun_9c33ef7f3a234d8c93bcbc7d71687ec5` completed with
one passed conversation evaluation and zero errors. The requested configuration
was one seed, one repetition, and maximum two turns. Generated/completed
conversation counters and actual turn counts were honestly **not reported**.
Separate inspection of this bounded run's transcript found two user messages
and two assistant messages; that does not make the CLI's missing aggregate
counters available.
These results remain evidence for the withheld `9a27cd20` packages, not acceptance
of the new source or every scenario, platform, output path, or negative case.

Record the exact candidate version, source SHA, scenario, observed result, and
sanitized evidence. A successful request submission is not a completed/scored
simulation. For simulation output, requested seeds/repetitions/turn ceilings
are not measured completed conversations or actual turns. Unsupported observed
counters must say **not reported**, not be estimated.

Service-side sample-count fixes have merged, but deployment to the test project
is unknown. The final GA simulation discriminator is unconfirmed. Privacy
signoff remains pending. None of those gates is satisfied by publishing this
unofficial feed. Do not include tokens, private prompts, customer rows, or full
raw service responses in public evidence.
