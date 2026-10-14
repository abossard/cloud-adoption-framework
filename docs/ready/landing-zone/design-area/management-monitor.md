---
title: Monitor Azure platform landing zone health
description: Use health models to represent Azure landing zone capabilities and operational concerns, supported by telemetry and resource monitoring.
author: jtracey93
ms.author: jatracey
ms.date: 10/14/2026
ms.update-cycle: 180-days
ms.topic: concept-article
ms.custom: internal, UpdateFrequency.5
ai-usage: ai-assisted
---

# Monitor Azure platform landing zone health

Monitor the capabilities your Azure landing zone supports. The platform team needs to know which services work and which specific situations fail.

Health modeling connects operational concerns to evidence and dependencies. Define healthy behavior for the platform, then relate observed conditions to the concerns they affect.

## Azure landing zone monitoring guidance

Start with health models that represent the value and capabilities of your landing zone. A model can assess a shared capability, a particular situation, or a group of services with a common owner.

The value comes from how you build the model. Your choices of entities, signal criteria, and relationships determine whether its health states describe your concerns. A collection of resources or imported alert rules is a starting input, not that interpretation.

Use [Azure Monitor health models](/azure/azure-monitor/health-models/overview) to model entities, their signals, and dependency rollup. Signals interpret existing observations or accept externally evaluated health. Relationships describe which concerns depend on others. Alerts notify operators about selected health changes.

> [!IMPORTANT]
> Azure Monitor health models are in preview. Review [availability and required access](/azure/azure-monitor/health-models/create) and validate the model before relying on it operationally. Keep the telemetry and alert coverage your organization requires.

### Start with the concerns your platform serves

Choose questions that platform and workload owners need monitoring to answer. The following examples span ALZ domains. They are modeling choices, not predefined domain templates.

| ALZ domain | Example concern |
|---|---|
| Connectivity | Does shared routing work, and can a particular datacenter connection reach its required destination? |
| Identity | Can operators and workloads authenticate to the services they require? |
| Management | Does operational telemetry arrive with enough freshness to support platform decisions? |
| Security | Do the controls protecting an approved path behave as intended? |
| Landing-zone workloads | Can a critical workload journey use its shared platform dependencies? |

A tenant-level view can combine domain and workload models. Keep detailed concerns with their owners and aggregate the health that matters to the wider platform.

:::image type="content" source="media/alz-connectivity-health-model/alz-connectivity-tenant-scale.svg" alt-text="An illustrative tenant health model combines shared platform domains, including connectivity, with corporate and online workload models." lightbox="media/alz-connectivity-health-model/alz-connectivity-tenant-scale.svg" border="false":::

### Build the model around meaning

Choose the concern before you select resources or thresholds.

1. Define the concern and its owner. Choose whether the model describes a domain, shared capability, or specific situation.
2. Choose evidence and criteria. Agree what Healthy, Degraded, Unhealthy, and Unknown mean for that concern. Discovery finds resources. Existing alerts can supply starting criteria.
3. Define dependencies and impact. A failed resource or narrow situation should affect parent health according to the service's actual tolerances.
4. Choose the response. Identify who investigates a concern and which health changes should notify that team.

For the concepts and worked examples, see [Health modeling for Azure landing zones](management-connectivity-health-model.md). Connectivity is one example domain, not the limit of the approach.

### Support the model with telemetry and resource alerts

Health models interpret evidence. Resource monitoring supplies the observations and operational coverage on which that interpretation depends.

Use Azure Monitor baseline alerts (AMBA) for metric, activity log, and log query alert coverage on supported platform resources. Tune alert criteria for your estate. See [Azure Monitor alerts](/azure/azure-monitor/alerts/alerts-overview) and [Introduction to an Azure Monitor deployment](https://azure.github.io/azure-monitor-baseline-alerts/patterns/alz/HowTo/deploy/Introduction-to-deploying-the-ALZ-Pattern/).

Azure Policy can deploy alert rules and supporting notification resources at scale with the *DeployIfNotExists* effect. Follow [policy-driven governance guidance](/azure/cloud-adoption-framework/ready/enterprise-scale/dine-guidance#why-use-dine-and-modify-policies) and the [AMBA contributor guide](https://aka.ms/amba/contributing) for policy and alert extensions.

Configure at least one action group per subscription with an email notification channel. Test routing to the responsible staff. Service health alerts don't support alert processing rules, so configure their action groups directly.

Supporting resources include the [AMBA policy topology](media/azure-monitor-baseline-alerts-policy-initiative-flow.svg), its [Visio source](https://azure.github.io/azure-monitor-baseline-alerts/patterns/alz/media/AMBA-Diagrams.vsdx), and the [resource-monitoring walkthrough video](https://www.youtube.com/embed/xeuxetAfHRg?si=1dmmOyIAkdf8nMgV).

## Brownfield guidance

Begin with existing capabilities, resources, and monitoring. Health modeling doesn't require the estate to have a new infrastructure-as-code deployment.

1. Choose one concern and define its scope with the responsible team.
2. Discover relevant resources and add the capability or situation entities that resource inventory alone can't describe.
3. Reuse suitable observations and alert criteria. Compare the model's states with known incidents before changing existing notification arrangements.

Maintain resource-alert coverage alongside this work. Import relevant AMBA policies and initiatives, assign them at the intended scope, and remediate noncompliant resources. In an ALZ-aligned estate, use the corresponding hierarchy. In another estate, choose the topmost management group from which the policies should apply. See [Determine your management group hierarchy](https://aka.ms/amba/alz/brownfield).

## Test the framework

Validate the model's interpretation and the supporting monitoring before production use.

1. Compare known successful and failed situations with their signal and entity states.
2. Check how a scoped failure affects shared capabilities and the domain root. Include a missing-data case.
3. Test alert deployment, notification routing, and the team's response process.

See [Testing approach for Azure landing zones](/azure/cloud-adoption-framework/ready/enterprise-scale/testing-approach).

> [!NOTE]
> AMBA's alert, severity, and threshold guidance also applies outside Azure Policy. You can deploy resource alerts through Azure Resource Manager, Bicep, Terraform, or the portal.

## Next steps

- [Health modeling for Azure landing zones](management-connectivity-health-model.md)

- [Introduction to deploying Azure Monitor](https://azure.github.io/azure-monitor-baseline-alerts/patterns/alz/HowTo/deploy/Introduction-to-deploying-the-ALZ-Pattern/)

- [Business continuity and disaster recovery](management-business-continuity-disaster-recovery.md)
