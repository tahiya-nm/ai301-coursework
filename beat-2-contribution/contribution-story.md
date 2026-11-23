# Contribution: Recognise well-known OpenAI-compatible providers (Groq, Mistral, Together) without listing them in `gateways`

Path: `beat-2-contribution/contribution-story.md`

**Contribution Number:** 1
**Student:** Tahiya Mahin
**GitHub Username:** tahiya-nm
**Issue:** https://github.com/open-telemetry/opentelemetry-ebpf-instrumentation/issues/4200
**Pull Request:** https://github.com/open-telemetry/opentelemetry-ebpf-instrumentation/pull/4322

---

## Toolkit

**Unit 5: Issue search, done mostly by hand with Claude and the Issue Selection site**

I found this issue by searching GitHub and the AI301 Issue Selection site, with Claude helping to shortlist and check candidates. The search fell short twice: Claude suggested an issue that a merged PR had already fixed, and it understated the competition on another. I now read each thread, linked PR and PR body myself before claiming an issue. I did not run the issue-scout skill on #4200 before choosing it, so there is no scout verdict for this pick.

**Unit 6: Not used for this contribution**

I did not use a voice-guard or repo-memory file while working on this issue. I wrote my PR description and claim comment myself and asked Claude to review them against the repo's AI policy, then disclosed that AI use in the PR.

**Unit 7: Not used for this contribution**

I did not run a ship-gate or gate manifest on this PR. My pre-submission checks were the repo's own make targets (`lint`, `test`, `generate-config-schema`, `lint-markdown`) and a manual read of `CONTRIBUTING.md`, `AI-POLICY.md`, `AGENTS.md` and `VERSIONING.md`.

**Unit 8: Not used for this contribution**

I did not use a PR-monitor or playbook. The PR was opened as a draft on 2026-11-23 and has no maintainer feedback yet, so there was nothing for one to track.

---

## Why I Chose This Issue

I wanted an issue in an AI/ML-adjacent project that I could finish alone with limited time, that maintainers would review quickly, and that would look credible on a resume. My first pick, aibrix #3100, was a call for testing that several people were already working on, so I dropped it. I then searched GitHub issue lists and the AI301 Issue Selection site, and checked each candidate's thread, linked PRs and PR bodies for "Closes #N". Several well-known AI repos had good-first-issues that were already taken or already fixed (for example WeatherGenerator #2059 was fixed by a merged PR). #4200 in OpenTelemetry eBPF Instrumentation (OBI) fit: it is about GenAI observability, the change is small and self-contained, and the issue names the providers, the files and the tests to add.

The issue carried a maintainer note reserving it for the KubeCon NA ContribFest on Nov 10. I did not start work. After the event passed I commented asking whether it was free and whether the maintainers wanted built-in defaults. A maintainer (mmat11) replied "I think we can start with some defaults", the issue author (NimrodAvni78) agreed, and the issue was assigned to me.

Claim comment: https://github.com/open-telemetry/opentelemetry-ebpf-instrumentation/issues/4200#issuecomment-6811484109

I did not run my issue-scout on #4200 before choosing it, so I have no scout verdict to report. I chose it from my own reading of the thread, the maintainers' replies and the code, which is also why I could say what the change would touch before I claimed it.

**Toolkit**

- Tool: Claude (a chat session with a browser pane and a sandbox) plus the AI301 Issue Selection site and GitHub issue search. I did not use issue-scout for this pick.
- What it produced: a ranked shortlist, and for each candidate a check of the thread, linked PRs and maintainer activity; #4200 came out as the best fit.
- Difficulties: Claude suggested two issues without checking enough: WeatherGenerator #2059 was already fixed by a merged PR whose body said "Closes #2059", and it understated the competition on aibrix #3100. I caught both by reading the threads and PR bodies myself. GitHub's `-linked:pr` filter also misses claims made in comments and "Closes #N" in a PR body.
- What changed: I stopped trusting search filters alone and read each thread, the linked PRs and the code before claiming.

---

## Understanding the Issue

**Problem Description**

OBI can turn HTTP calls to OpenAI-compatible APIs into GenAI spans, but only for hosts the user lists under `openai_compatible.gateways`. Groq, Mistral and Together serve the OpenAI chat completions API from their own hosts and do not send the `Openai-*` headers that OBI's OpenAI detector looks for. Their traffic therefore shows up as plain HTTP spans unless the user lists each host by hand, even though the parser already handles their responses.

**Expected Behavior**

With `openai_compatible` extraction enabled, calls to `api.groq.com`, `api.mistral.ai` and `api.together.xyz` produce GenAI spans with a provider name, with no `gateways` config needed. A host the user configures still takes priority.

**Current Behavior**

Those calls are reported as plain HTTP client spans (for example `POST /openai/v1/chat/completions` with `http.*` attributes). Adding a `gateways` entry for the host turns the same call into a `chat <model>` span with `gen_ai.*` attributes, as the issue's reproduction shows.

**Affected Components**

- `pkg/ebpf/common/http/openai_compatible.go`: `OpenAICompatibleSpan`, where the host is matched against the gateways list
- `pkg/ebpf/common/http/openai_compatible_test.go`: existing gateway tests, where the new ones go
- `pkg/ebpf/common/http_transform.go`: `postProcessHTTPSpan`, which calls `OpenAICompatibleSpan`
- `pkg/config/payload_extraction.go`: the `OpenAICompatibleConfig` structs and the `Gateways` description
- `devdocs/config/CONFIG.md` and `config-schema.json`: generated from that description
- `devdocs/features.md` and `SUPPORT_MATRIX.md`: list the supported OpenAI-compatible gateways
- `pkg/appolly/app/request/span.go`: the `gen_ai.provider.name` value set

**Toolkit**

- Tool: Claude, reading the issue and the code through a browser pane and a sandbox clone of the repo.
- What it produced: a map of where host matching happens and how the span reaches the exporters.
- Difficulties: the issue says the change belongs in `pkg/config/payload_extraction.go` lines 177-190, but that file now only defines structs; the matching logic is in `openai_compatible.go`. I found this by searching for where `Gateways` is used.
- What changed: I wrote the plan against the real file and said so in my comment on the issue.

---

## Reproduction Process

**Environment Setup**

macOS on Apple Silicon (arm64). Go 1.26 or newer (the repo's `go.mod` requires 1.26.0). Docker Desktop. I forked `open-telemetry/opentelemetry-ebpf-instrumentation`, cloned my fork, and added the upstream remote. The repo needs generated eBPF Go bindings before anything compiles, so I ran `make docker-generate BPF_TARGETS=arm64` (it pulls the `obi-generator` image). The first `go test` failed with `undefined: BpfHttpRequestTraceT` and similar errors until the bindings were generated; after that the baseline `go test -short -count=1 ./pkg/ebpf/common/http/ -run OpenAICompatible` passed. A later full `make verify` run failed in `./scripts` with `declare: -A: invalid option`, because macOS ships bash 3.2 and the script needs bash 4 or newer; installing bash with Homebrew fixed it and `go test ./scripts/...` then passed. OBI's eBPF probes cannot run on macOS, so I reproduced the bug at unit level instead of running the container setup from the issue.

**Steps to Reproduce**

1. Check out `main` (without the fix) and generate the bindings as above.
2. Add `TestOpenAICompatibleSpan_DefaultGateways` to `pkg/ebpf/common/http/openai_compatible_test.go`. It builds a chat-completions request to `http://api.groq.com/openai/v1/chat/completions`, calls `OpenAICompatibleSpan` with no configured gateways, and expects the span to match.
3. Run `go test -short -count=1 ./pkg/ebpf/common/http/ -run TestOpenAICompatibleSpan_DefaultGateways`.
4. Observed result: all three subtests fail with `Should be true` at the line that requires `OpenAICompatibleSpan` to match, because the function returns "not matched" for the host when no gateway is configured.

**Reproduction Evidence**

- **Commit showing reproduction:** https://github.com/tahiya-nm/opentelemetry-ebpf-instrumentation/commit/f90f3df6b (a test-only commit on branch `repro/4200`, based on upstream `main`)
- **Screenshots/logs:**

```text
--- FAIL: TestOpenAICompatibleSpan_DefaultGateways (0.00s)
    --- FAIL: TestOpenAICompatibleSpan_DefaultGateways/groq (0.00s)
        openai_compatible_test.go:571:
                Error:          Should be true
    --- FAIL: TestOpenAICompatibleSpan_DefaultGateways/mistral (0.00s)
        openai_compatible_test.go:571:
                Error:          Should be true
    --- FAIL: TestOpenAICompatibleSpan_DefaultGateways/together_has_no_enum_member (0.00s)
        openai_compatible_test.go:571:
                Error:          Should be true
FAIL    go.opentelemetry.io/obi/pkg/ebpf/common/http    0.837s
```
- **My findings:** At unit level the bug reproduces: with no configured gateways, `OpenAICompatibleSpan` returns "not matched" for Groq, Mistral and Together, so the call stays a plain HTTP span. I did not reproduce the issue's end-to-end behavior with real containers, because that needs a Linux host with eBPF support; I rely on the issue's own reproduction (OBI v1.0.0) for that part.

**Toolkit**

- Tool: Claude to explain the code and the repo's test setup; Docker for the generator image.
- What it produced: the commands for generating bindings and a test that targets the bug.
- Difficulties: missing generated bindings on a fresh clone; no way to run eBPF on macOS.
- What changed: I reproduced at unit level and said so plainly here.

---

## Solution Approach

**Analysis**

`OpenAICompatibleSpan` converts a request into an OpenAI-compatible span only when its host (case-insensitive, optional port) matches an entry in the user's gateways list; otherwise it returns the base HTTP span. OBI's own OpenAI detector keys on `Openai-*` response headers, which Groq, Mistral and Together do not send. The issue shows that adding a single gateway entry already produces correct GenAI spans, so the parsing is fine and only the host list is missing. `postProcessHTTPSpan` calls this function whenever `openai_compatible` is enabled, whatever the list contains, so defaults will be reached.

**Proposed Solution**

Add a built-in default list (`api.groq.com` as `groq`, `api.mistral.ai` as `mistral_ai`, `api.together.xyz` as `together_ai`) that is checked after the user's gateways, so a configured entry for the same host wins. Not changing: the config format or schema, the provider value set, or the parsing. `together_ai` is not in the `gen_ai.provider.name` set, so Together is reported as `custom`; I raise this as a question in the PR. I am also not adding a way to turn the defaults off, and I ask the maintainers whether they want one.

**Implementation Plan**

1. In `openai_compatible.go`, add the default list and move the matching loop into a helper, `matchOpenAICompatibleGateway`, so it runs over the configured list first and then the defaults.
2. In `openai_compatible_test.go`, add a table test for the three hosts and a test that a configured gateway overrides a default.
3. Reword the `Gateways` description (it said "opt-in allowlist") in `payload_extraction.go`, and regenerate `CONFIG.md` and `config-schema.json` with `make generate-config-schema`.
4. Add the three providers to the gateway lists in `devdocs/features.md` and `SUPPORT_MATRIX.md`.

Deviations: the issue pointed at `payload_extraction.go` for the logic, so I moved the code change to `openai_compatible.go`. I only found the features and support-matrix docs (step 4) after reading the PR template and compatibility rules, so I added them late.

**Toolkit**

- Tool: Claude, to explain the code and draft the patch and tests; I reviewed the diff and ran everything locally.
- What it produced: a patch of about 86 added lines across five files, later plus the two docs edits.
- Difficulties: the first draft missed the features and support-matrix docs, and I initially described the change as purely additive, which is not quite right: for these hosts the plain HTTP span and metrics are replaced by GenAI ones.
- What changed: I added the docs and wrote the compatibility caveat in the PR.

---

## Testing Strategy

Observable checks:

1. **Before and after on the reproduction.** `TestOpenAICompatibleSpan_DefaultGateways` fails on `main` (no match for the host) and passes with the change, with subtests `groq`, `mistral` and `together_has_no_enum_member`. It asserts the span subtype, the stored provider name (`groq`, `mistral_ai`, `together_ai`) and the emitted attribute (`groq`, `mistral_ai`, `custom`).
2. **Precedence.** `TestOpenAICompatibleSpan_ConfiguredGatewayOverridesDefault` configures `api.groq.com` with provider `litellm` and checks that the span reports `litellm`.
3. **The repo's own checks.** `go test -short -count=1 ./pkg/ebpf/common/http/ ./pkg/config/...` passes (`ok` for both packages), `go test ./scripts/...` passes after the bash fix, `make lint` reports no issues, `make generate-config-schema` leaves only the intended one-line description changes, and `make lint-markdown` reports 0 issues.

Not run as of 2026-11-23: a complete `make verify` (my first run stopped at the macOS bash problem described above and I have not re-run it to completion yet), `make check-config-schema`, and the integration tests, which need a Linux host with eBPF support. Some Linux-only unit tests may be skipped on macOS. I will update this section when `make verify` finishes.

**Toolkit**

- Tool: the repo's `make` targets and Go test; Claude to explain test failures and review my reasoning.
- What it produced: passing tests and clean lint.
- Difficulties: `make fmt` changed an unrelated file (`pkg/export/otel/metrics_shutdown_test.go`, an import-grouping blank line already present on `main`).
- What changed: I reverted that file so the PR contains only my changes.

---

## Implementation Notes

**2026-11-11**

Searched issue lists and the AI301 site, checked threads and PR bodies, dropped aibrix #3100, picked #4200, claimed it on the site, and commented on the issue asking whether the ContribFest reservation was over and whether defaults were wanted.

**Between 2026-11-11 and 2026-11-22: maintainer reply**

mmat11 replied "I think we can start with some defaults", NimrodAvni78 agreed, the ContribFest label was removed, and the issue was assigned to me.

**2026-11-22**

Forked and cloned the repo, got Go and Docker working, generated the eBPF bindings, and confirmed the baseline test passed. Found that the issue's file pointer was out of date and moved the change to `openai_compatible.go`. Drafted the patch and tests with AI help, then applied it and ran the tests.

**2026-11-23**

Ran `make lint` and regenerated the config schema, reverted an unrelated formatter change, added the features and support-matrix doc edits, committed, and pushed to my fork. Pushed a separate test-only commit (`repro/4200`) as evidence of the bug. A first `make verify` run failed in `./scripts` because macOS bash is too old; I installed a newer bash and that package now passes. Opened draft PR #4322 against `open-telemetry/opentelemetry-ebpf-instrumentation`. The EasyCLA bot flagged my commits as unsigned, and I need to sign the CLA before the PR can be reviewed.

**Toolkit**

- Tool: Claude, plus the repo's make targets.
- What it produced: the working patch, test output and a record of each decision.
- Difficulties: environment setup took the longest, not the code change itself.
- What changed: nothing in the plan after the docs addition.

---

## Pull Request

**PR Link:** https://github.com/open-telemetry/opentelemetry-ebpf-instrumentation/pull/4322

**PR Description:** Built-in default gateways so Groq, Mistral and Together are recognized as OpenAI-compatible providers without any `gateways` config. Configured gateways are checked first, then the defaults; the matching loop moved into a helper so it runs over both lists. The description lists the tests (a new table test for the three hosts that fails on `main`, and an override test), the checks I ran and did not run, a compatibility section, two open questions for reviewers (whether `together_ai` should become a member of the `gen_ai.provider.name` enum, and whether the defaults need an off switch), `Fixes #4200`, and an AI-use disclosure. The release-level line in the compatibility section says I am still checking the classification against `VERSIONING.md` and will update it before marking the PR ready.

**Maintainer Feedback:**

- 2026-11-23: none yet. The PR was just opened as a draft, and I still need to sign the EasyCLA.

**Status:** Draft (opened 2026-11-23; CLA not yet signed as of this entry)

**Toolkit**

- Tool: Claude, to review my PR description draft against `AI-POLICY.md`, `AGENTS.md`, `CONTRIBUTING.md` and `VERSIONING.md`. I did not use a ship-gate or PR-monitor.
- What it produced: a list of changes to my draft: a compatibility section, open questions for reviewers, and the AI-use disclosure.
- Difficulties: the repo requires human-written GitHub text, so I rewrote the description in my own words rather than pasting generated text. The release-level classification is still unresolved because the defaults cannot be turned off.
- What changed: I added a compatibility section that says plainly that GenAI spans replace HTTP ones for these three hosts, and I will not mark the PR ready until the CLA is signed, `make verify` has passed and the release-level line is settled.

---

## Learnings and Reflections

**Technical Skills Gained**

How OBI turns captured HTTP requests and responses into GenAI spans and routes them to GenAI metrics. Writing table-driven Go tests that follow a package's existing style. Generating eBPF bindings with `bpf2go` in Docker. Regenerating a config schema and docs from struct comments. Applying a project's semver compatibility policy to a change in default behavior.

**Challenges Overcome**

Getting a build environment working for a project that needs generated code and a newer Go than I expected. Working out which issue was safe to claim when several were already fixed or in progress. Tracing the real code path when the issue's file pointer had gone stale. Following a repo's AI policy, which means understanding every line of the change and writing the PR text myself.

**What I'd Do Differently Next Time**

Read the repo's contributing and AI policy files before choosing an issue. Check PR bodies for "Closes #N" and the code itself before trusting that an issue is open. Check every file pointer in an issue against the current code. Start the environment setup early, because it was the longest step.

---

## Resources Used

- The issue thread: https://github.com/open-telemetry/opentelemetry-ebpf-instrumentation/issues/4200
- OBI `CONTRIBUTING.md`, `AI-POLICY.md` and `AGENTS.md`, which set the rules for the PR
- OBI `VERSIONING.md` and `TELEMETRY.md`, for the compatibility assessment
- The Makefile targets `docker-generate`, `generate-config-schema`, `lint` and `lint-markdown`
- `pkg/ebpf/common/http/openai_compatible_test.go`, whose existing gateway tests I followed
