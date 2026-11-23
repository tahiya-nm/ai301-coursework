# Teaching OpenTelemetry eBPF Instrumentation to recognise Groq, Mistral and Together out of the box

## Overview

OpenTelemetry eBPF Instrumentation (OBI) watches applications from the Linux kernel and reports what they do as traces and metrics. It can turn calls to OpenAI-style AI APIs into GenAI spans, but only for hosts a user lists by hand under `openai_compatible.gateways`. Calls to Groq, Mistral and Together, which all serve the OpenAI API from their own hosts, showed up as plain HTTP calls with no model or token data. I took issue [#4200](https://github.com/open-telemetry/opentelemetry-ebpf-instrumentation/issues/4200), got the maintainers' agreement to add built-in defaults, and opened a draft pull request: [#4322](https://github.com/open-telemetry/opentelemetry-ebpf-instrumentation/pull/4322). As of November 23, 2026 it is still a draft. It is not merged, I have not yet signed the project's contributor agreement, and a few checks are still open.

## What happened

**Claiming the issue.** The issue was reserved for a conference contribution day, so I waited until it passed and then asked whether it was free and whether the maintainers wanted built-in defaults. A maintainer replied, "I think we can start with some defaults." The issue author agreed and the issue was assigned to me.

**Reproducing the bug.** OBI's eBPF probes cannot run on macOS, so I reproduced the bug at unit level instead of with the issue's container setup. I wrote a test that sends a chat-completions request to each provider's host with no gateways configured and expects a match. On `main` it fails for all three hosts (commit `f90f3df6b` on my fork):

```text
--- FAIL: TestOpenAICompatibleSpan_DefaultGateways (0.00s)
    --- FAIL: TestOpenAICompatibleSpan_DefaultGateways/groq (0.00s)
        openai_compatible_test.go:571:
                Error:          Should be true
    --- FAIL: TestOpenAICompatibleSpan_DefaultGateways/mistral (0.00s)
    --- FAIL: TestOpenAICompatibleSpan_DefaultGateways/together_has_no_enum_member (0.00s)
FAIL    go.opentelemetry.io/obi/pkg/ebpf/common/http    0.837s
```

That confirmed the diagnosis: the parser already handles these responses, and the only thing missing is the host list.

**The fix.** The issue pointed at `payload_extraction.go`, but that file now only defines config structs, so I traced where `Gateways` is used and made the change in `openai_compatible.go`. I added a built-in list, checked after the user's own gateways so a configured entry for the same host still wins:

```go
var defaultOpenAICompatibleGateways = []config.OpenAICompatibleGateway{
	{Host: "api.groq.com", Provider: "groq"},
	{Host: "api.mistral.ai", Provider: "mistral_ai"},
	{Host: "api.together.xyz", Provider: "together_ai"},
}
```

The matching loop moved into a helper that runs over the configured list first and then the defaults. With the fix, the same test passes. It checks the span type, the stored provider name, and the emitted attribute. Together is the interesting row: `together_ai` is not in the `gen_ai.provider.name` value set, so the emitted attribute is `custom`, and the test says so in its name. I also added a test that a configured `api.groq.com` entry with provider `litellm` overrides the default.

**A change I made.** My first description of the change called it purely additive. Then I read the project's `VERSIONING.md`, which allows changed defaults in a minor release only if the old behaviour remains available. Here it does not: the defaults cannot be turned off, and for these three hosts GenAI spans and metrics now replace the plain HTTP ones. I rewrote the compatibility section of the PR to say that, and I left the release-level line as "still checking" instead of claiming a classification I had not confirmed. I also raised two questions for reviewers: whether `together_ai` should join the enum, and whether the defaults need an off switch. A smaller correction came from the same reading: my first patch missed the two docs that list supported gateways, so I added them, and I reworded the config description from "opt-in allowlist", which was no longer true:

```text
Before: Opt-in allowlist of gateway destinations ...
After:  Additional gateway destinations to match by host (case-insensitive) with optional port
        and provider name, checked before the built-in Groq, Mistral and Together hosts
```

**What landed.** The maintainers agreed to the approach before I wrote code, and the issue was assigned to me. That is where the result stands. I have no review comments on the PR yet.

## Limitations and what's next

The work has real gaps. There is no end-to-end test, because the probes need a Linux host with eBPF support, so I rely on the issue's own reproduction for that part. The defaults cannot be disabled, which is why the release level is unresolved. Together is reported as `custom` until the enum is extended. Beyond the work itself, the PR is a draft with the contributor agreement unsigned, and I ran the targeted tests and lint but have not finished a full `make verify`.

Next I will sign the agreement, finish `make verify` and `make check-config-schema`, and record the results in the PR. Then I will settle the release-level line with the maintainers, starting from the off-switch question, and mark the PR ready for review only after that. If they want an off switch, it would be a new optional config field with the defaults on by default, which keeps the old behaviour available.
