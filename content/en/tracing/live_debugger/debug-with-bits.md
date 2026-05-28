---
title: Debug with Bits
description: Use Bits AI Dev Agent to create and manage Live Debugger sessions through a conversational interface.
further_reading:
- link: "/bits_ai/bits_ai_dev_agent/"
  tag: "Documentation"
  text: "Bits AI Dev Agent"
- link: "/tracing/live_debugger/"
  tag: "Documentation"
  text: "Live Debugger"
- link: "/dynamic_instrumentation/sensitive-data-scrubbing/"
  tag: "Documentation"
  text: "Sensitive Data Scrubbing"
---

{{< beta-callout url="https://www.datadoghq.com/product-preview/live-debugger/" >}}
Debug with Bits is in Preview. Request access to join the waiting list.
{{< /beta-callout >}}

## Overview

Debug with Bits lets you use [Bits AI Dev Agent][5] to inspect running services through a conversational interface. Instead of manually navigating the Live Debugger UI, you can describe what you want to investigate, and Bits places logpoints, retrieves captured data, and helps you interpret results.

All debugging activity runs through [Live Debugger][1], so the same [permissions][2], rate limits, auto-expiry rules, and [sensitive data scrubbing][3] apply regardless of whether you create logpoints manually or through Bits.

## Requirements

Before using Debug with Bits:

- [Live Debugger][1] must be enabled for the target service. See [Requirements and setup][1] for details.
- Your account needs the [permissions][2] required to use Live Debugger, including read, write, and variable-capture permissions for the target environment.
- [Bits AI Dev Agent][5] must be available in your organization.

## Available actions

Bits can perform the following Live Debugger actions during a debugging session:

| Action | Description |
|--------|-------------|
| Discover services | Find and validate services available for debugging in a given environment. |
| Create logpoints | Add logpoints to a running service at a specific code location. |
| List session logpoints | Show the logpoints active in a Debug Session. |
| Disable logpoints | Disable all logpoints in a session. |
| Retrieve snapshot data | Fetch captured variable values and execution context from an active logpoint. |

Logpoints created by Bits follow the same rules as manually created logpoints: they are read-only, non-blocking, and auto-expire after a configurable duration between 10 minutes and 48 hours (default: 60 minutes). Bits cannot modify application state or alter control flow.

## Get started

1. Navigate to the [Live Debugger page][4].
2. Open the Bits AI Dev Agent chat and describe the issue you want to investigate.
3. Bits identifies the relevant service and environment and proposes logpoint locations.
4. Review and confirm the proposed logpoints. Bits creates them and monitors for captured data.
5. When Bits retrieves snapshot data, review the captured variable values and execution context in the chat.
6. Ask Bits to disable the session, or let the logpoints expire automatically.

## Notes

**Multi-version environments**: When multiple code versions are deployed in the target environment and the target file differs between versions, Bits asks you to confirm the target version before placing a logpoint. This prevents logpoints from landing at incorrect line numbers.

**Language support**: Some features vary by language. For example, condition expressions are not supported for all runtimes. Bits notifies you when a requested feature is not available for the target service's language.

**Sensitive data**: The same [sensitive data scrubbing][3] that applies to manually created logpoints applies to logpoints created by Bits. In production environments, non-numeric and non-Boolean captured values are redacted by default.

## Further reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: /tracing/live_debugger/
[2]: /tracing/live_debugger/#permissions
[3]: /dynamic_instrumentation/sensitive-data-scrubbing/
[4]: https://app.datadoghq.com/debugging/sessions
[5]: /bits_ai/bits_ai_dev_agent/
