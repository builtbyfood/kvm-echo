---
name: Bug report
about: Something in kvm-echo did not work the way it should
title: ''
labels: bug
assignees: ''
---

<!--
Before filing: check the "Known issues" section of the User Guide first —
docs/USER-GUIDE.md#known-issues-check-here-first — a couple of these have known
workarounds and are already being tracked.

If your problem involves a capture or an export, the jobId is the single most
useful thing you can include. It appears on the result panel, in the Captures
table row, and in every related log line on both the appliance and the host.
-->

## Environment

| | |
|---|---|
| **Plugin version** | <!-- e.g. 0.7.8 --> |
| **Morpheus version** | <!-- e.g. 9.0.4 --> |
| **KVM host OS** | <!-- e.g. Ubuntu 24.04 --> |

## What I did

<!-- The sequence of clicks that got you there. -->

## What I expected

## What happened instead

<!-- A screenshot of the panel is very helpful here. -->

## jobId

<!-- The 12-character string from the result panel or the Captures table row.
     Leave blank if the problem isn't capture- or export-related. -->

## Attached

<!-- Tick what you're including. -->

- [ ] Screenshot(s)
- [ ] Appliance log lines for the jobId:
      `sudo grep '<jobId>' /var/log/morpheus/morpheus-ui/current | tail -50`
- [ ] Host wrapper log, if capture-related — SSH to the KVM host:
      `sudo cat /dev/shm/echo/<jobId>.log`
      `sudo cat /dev/shm/echo/<jobId>.status`

<!--
Please skim logs before attaching. Capture metadata can include interface names,
BPF filters and hostnames from your environment.
-->
