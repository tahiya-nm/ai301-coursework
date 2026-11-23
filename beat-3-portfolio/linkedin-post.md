I opened a fix for an open source tool so it recognises calls to three popular AI services (Groq, Mistral and Together) without any manual setup. The tool is OpenTelemetry eBPF Instrumentation, which reports what running software is doing, and before my change those AI calls appeared as generic web traffic with no model or usage details.

I wrote a test that failed on the existing code, made the change, and watched it pass. The maintainers agreed to the approach before I wrote any code. While preparing the pull request I reread the project's versioning rules and realised my first description of the change, "purely additive", was wrong, so I rewrote it and asked the maintainers a question instead of guessing.

Today the pull request is still a draft awaiting my final checks and review. It taught me to check a change against a project's own rules before calling it safe.

https://github.com/open-telemetry/opentelemetry-ebpf-instrumentation/pull/4322

#OpenSource #Observability #AI
