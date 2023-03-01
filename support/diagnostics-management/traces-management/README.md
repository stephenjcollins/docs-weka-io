---
description: This page describes how to manage traces generated by the Weka processes.
---

# Traces management

Traces are low-level events generated by Weka processes and collected by an internal tracking tool. The traces are used as troubleshooting information for support purposes.

The tracking tool collects the traces continuously on the backends and clients. The number of traces retained depends on the specified free capacity on the backends and clients and the maximum capacity that traces can use. The tracking tool manages the retention of the traces as a FIFO queue.

If it is required to keep traces of a specific period for later investigation, you can freeze that period, and the traces within it are retained for a specified duration, enabling the support enough time to resolve the issue.

You can switch the verbosity level of the traces between low and high. The verbosity level affects the amount of information in the tracing data.

{% hint style="warning" %}
Do not disable the traces without specific instructions from the [Customer Success Team](../../getting-support-for-your-weka-system.md#contact-customer-success-team). Disabling the traces reduces the amount of troubleshooting information about the system and may affect the SLA if an issue occurs.
{% endhint %}

****

**Related topics**

[manage-traces-using-the-gui.md](manage-traces-using-the-gui.md "mention")

[manage-traces-using-the-cli.md](manage-traces-using-the-cli.md "mention")