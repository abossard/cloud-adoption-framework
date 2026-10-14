---
title: Health modeling for Azure landing zones
description: Model Azure landing zone capabilities and concerns with entities, signals, and dependencies, using connectivity as a worked domain.
author: abossard
ms.date: 10/14/2026
ms.topic: concept-article
ai-usage: ai-generated
---

# Health modeling for Azure landing zones

A health model represents the capabilities and operational concerns of your Azure landing zone. Its scope can be a shared capability, a specific situation, or a whole domain.

Choose entities, signal evidence, and dependencies around the questions platform owners need to answer. Those choices determine what the model's health states mean. Alerts are one response to those states, not the model's definition.

Use [Azure Monitor health models](/azure/azure-monitor/health-models/overview) to model the concerns described in the [Azure landing zone monitoring guidance](management-monitor.md). The examples below use connectivity as one ALZ domain. The same concepts apply to identity, management, security, and workload concerns.

> [!IMPORTANT]
> Azure Monitor health models are in preview. Review [supported setup and access](/azure/azure-monitor/health-models/create) before choosing the feature for your platform.

## Choose a capability or concern

Start with the capability or situation whose health matters to its owner.

The model can include broad capabilities and narrower situations that depend on them. Define the required behavior and the evidence that distinguishes success, reduced service, failure, and insufficient data.

| ALZ domain | Shared capability | Specific situation |
|---|---|---|
| Connectivity | Routing. | Traffic from one datacenter subnet to one Azure subnet. |
| Identity | Authentication. | Operator access to a required management service. |
| Management | Operational telemetry. | Incident data available for a particular subscription. |
| Security | Policy enforcement. | A named protection control behaves as intended. |
| Landing-zone workloads | Shared platform support. | A critical workload accesses its required platform service. |

These are examples of concerns, not predefined model templates. Choose whether one model covers a domain, a situation, or a broader set of services. See [health-model scope and domain models](/azure/azure-monitor/health-models/health-models-faq) and [health-model design principles](/azure/well-architected/design-guides/health-modeling).

## Use connectivity as a worked domain

A connection scope names the traffic the platform team needs to assess.

Start with an agreed owner and a concrete source, destination, and expected behavior. Use your [network topology and connectivity design](network-topology-and-connectivity.md) to identify the path's dependencies.

| Connection | What must work |
|---|---|
| Datacenter to Azure | Required Azure endpoints remain reachable from the datacenter. Routes and name resolution support that traffic. |
| Corporate egress | Required outbound traffic reaches its destination. Security policy still blocks disallowed traffic. |
| Azure to AWS | Named endpoints remain reachable across the agreed cross-cloud path. The model covers the dependencies you can observe. |

In this worked domain, one model can assess shared connectivity capabilities and several connection-specific situations. Define their scopes before selecting resources or signals.

## Model capabilities and specific situations

An entity represents something whose health you want to understand.

Use generic entities for both shared capabilities and specific situations. In the connectivity example, Routing summarizes shared route infrastructure. Datacenter-to-Azure routing depends on Routing and adds signals for one subnet-to-subnet path.

The overview separates what you assess from the observations that inform it. Solid links show entity dependencies. Dashed links show signal assignments. The blue input represents selected checks, not one query that every entity must share.

> [!NOTE]
> Diagram states and values are illustrative. Choose criteria and rollup rules for your connections. The capability structure is a starting design, not a built-in deployable model.

<!-- Diagram source: diagrams/alz-connectivity-health-model.md (repository root). Use its rendering command to refresh the SVGs. -->
:::image type="content" source="media/alz-connectivity-health-model/alz-connectivity-overview.svg" alt-text="Overview of shared connectivity capabilities and specific situations, with resource entities and signals informed by Azure resources or telemetry." lightbox="media/alz-connectivity-health-model/alz-connectivity-overview.svg" border="false":::

The capability and situation groups both contain generic entities. Their different colors identify their roles, not different product entity types or health states.

| Entity type | Connectivity example |
|---|---|
| Root entity | Connectivity represents this domain model's overall health. Each model has one root with a name that reflects its scope. |
| Generic entity | Routing and Datacenter-to-Azure routing describe a capability or situation. Either can have its own signals and child dependencies. |
| Azure resource entity | An entity represents Route Server or Azure Firewall. It interprets resource observations without containing or recreating the resource. |

See [entity types and relationships](/azure/azure-monitor/health-models/concepts#entities).

| Shared capability | What to assess | Resource guidance |
|---|---|---|
| Hybrid | The connection to the datacenter or another network. | [ExpressRoute](/azure/expressroute/expressroute-introduction). |
| Security services | Policy enforcement on the connection's path. | [Azure Firewall](/azure/firewall/overview). |
| Application delivery | The required ingress or delivery path. | [Application Gateway](/azure/application-gateway/overview). |
| Routing | Route exchange and forwarding to the destination. | [Azure Route Server](/azure/route-server/overview). |
| DNS | Name resolution for the connection. | [Azure DNS Private Resolver](/azure/dns/dns-private-resolver-overview). |

## Limit the impact of a scoped failure

A specific situation can fail while its shared capability remains healthy.

In this example, Route Server and Routing are Healthy. The path-specific check fails, so Datacenter-to-Azure routing is Unhealthy. Other capabilities remain Healthy in the illustrative snapshot.

:::image type="content" source="media/alz-connectivity-health-model/alz-connectivity-routing-situation.svg" alt-text="An unhealthy datacenter routing situation has Limited impact on a degraded Connectivity root while Routing and other shared capabilities remain healthy." lightbox="media/alz-connectivity-health-model/alz-connectivity-routing-situation.svg" border="false":::

Set **Limited** impact on the situation entity when its failure reduces the domain's service without making the whole domain unusable. Its Unhealthy state contributes Degraded to the Connectivity parent. The root uses **Worst of** dependencies in this example. The situation's failure does not roll down into Routing.

Choose **Standard** impact for a dependency whose failure should contribute its full state. Use Limited only when the reduced contribution matches the service contract. See [health rollup and impact](/azure/azure-monitor/health-models/concepts#health-propagation) and [rollup configuration](/azure/azure-monitor/health-models/rollup).

## Interpret one resource in different contexts

One Azure resource can support more than one concern.

Use separate representations when the same firewall needs different health criteria for corporate egress and a partner connection. Each Azure resource entity evaluates its signals independently. The physical resource stays the same.

:::image type="content" source="media/alz-connectivity-health-model/alz-connectivity-resource-concerns.svg" alt-text="One Azure Firewall resource has two representations that evaluate egress and partner-path signals independently within a health model." lightbox="media/alz-connectivity-health-model/alz-connectivity-resource-concerns.svg" border="false":::

Reuse one entity across parents when the health interpretation is the same. Create separate resource representations when the concerns need independent criteria. See [resource representations and model scope](/azure/azure-monitor/health-models/health-models-faq).

## Choose how signals determine health

A signal contributes health evidence to an entity.

Use observations that describe the entity's required behavior. Telemetry collection stays outside the health model. Metric and query signals apply model criteria to existing data. External health signals receive a state that another system has already evaluated.

:::image type="content" source="media/alz-connectivity-health-model/alz-connectivity-signal-sources.svg" alt-text="Metric, KQL, PromQL, and externally evaluated report paths contribute signals to entity health before it rolls up to a parent." lightbox="media/alz-connectivity-health-model/alz-connectivity-signal-sources.svg" border="false":::

Choose the inputs relevant to each entity.

| Signal type | Evidence or calculation |
|---|---|
| Azure resource | Platform metrics from the selected resource, compared with chosen criteria. |
| Log Analytics workspace | A numeric KQL result from the selected Log Analytics workspace (LAW). |
| Azure Monitor workspace | A PromQL result from the selected Azure Monitor workspace (AMW). |
| Azure Resource Health | Platform-reported availability of the represented resource. |
| External health | A health state evaluated outside the model and reported to a named signal. |
| Dependencies | Child entity states combined using the parent's dependency settings. |

An external health report can include the observed value and the external system's criteria, but its evaluated state is the input. The report updates a named signal on an existing entity. The entity still combines that signal with its other evidence and dependencies.

For a firewall example, assess a named connection that should succeed. Use its path-check result and [structured firewall logs](/azure/firewall/firewall-structured-logs) as evidence. A count of all policy denies also includes expected blocks and doesn't by itself show failure.

See [signals and data sources](/azure/azure-monitor/health-models/signals), [health states](/azure/azure-monitor/health-models/concepts#health-states), and [external health reports](/azure/azure-monitor/health-models/health-report-ingestion).

## Treat missing data as an evidence gap

Unknown means the model lacks enough evidence to determine health.

The timeline follows one path entity with one external signal and no other signals or child dependencies. Its report expires before the next observation arrives. Later reports show failure and recovery.

:::image type="content" source="media/alz-connectivity-health-model/alz-connectivity-missing-data.svg" alt-text="A path's health changes from Healthy to Unknown after its only report expires, then to Unhealthy after a failure report and Healthy after recovery." lightbox="media/alz-connectivity-health-model/alz-connectivity-missing-data.svg" border="false":::

Report expiry removes the external signal. The entity recalculates health from the remaining evidence. Expiry does not force every entity to Unknown when other signals or dependencies can still determine its state.

1. Check whether the source stopped reporting or the model lacks access to its telemetry. Review [setup and required access](/azure/azure-monitor/health-models/create).
2. Distinguish missing observations from an observed failed check. Keep each signal's refresh and reporting behavior clear.
3. Compare gaps with the [Graph, Timeline, and Entity details views](/azure/azure-monitor/health-models/analyze-health). Use [report refresh and expiry guidance](/azure/azure-monitor/health-models/health-report-ingestion#keep-the-report-current) for external signals.

## Compose a tenant-scale view

A parent model can combine domain and workload models without copying their internal topology.

Keep the detailed connectivity capabilities and situations in a connectivity model. A tenant-level parent can include that model alongside other shared services and landing-zone workloads. The arrangement is an example, not a built-in tenant model.

:::image type="content" source="media/alz-connectivity-health-model/alz-connectivity-tenant-scale.svg" alt-text="A tenant parent model groups nested connectivity, identity, security, management, and workload models under shared platform services and corporate or online landing zones." lightbox="media/alz-connectivity-health-model/alz-connectivity-tenant-scale.svg" border="false":::

In the parent, each nested model has an Azure resource representation. Its contribution is the state of the nested model's root. Shared platform services and the landing-zone groups are generic entities for aggregation, not extra roots.

Choose grouping and impact rules that preserve the meaning of each domain's failures. See [nested health models](/azure/azure-monitor/health-models/concepts#nested-health-models).

## Discover the existing estate

Start with an existing concern and its current monitoring. This process applies across ALZ domains and doesn't require the estate to have a Terraform or Bicep deployment.

1. Discover resources within the concern's agreed ownership and scope using Azure Resource Graph.
2. Add shared capabilities and specific situations. Attach the discovered resources to their intended parents and review the actual dependencies.
3. Choose useful signals or reuse existing criteria. Compare the resulting health with known incidents, then refine scope and interpretation.

Discovery doesn't infer every business dependency. Combine [discovery rules](/azure/azure-monitor/health-models/discoveries) with [manual model design](/azure/azure-monitor/health-models/designer). Recommended signals for supported resource types are a starting point. See the existing [ALZ brownfield monitoring guidance](management-monitor.md#brownfield-guidance).

## Reuse alert criteria

Existing resource alerts capture monitoring decisions the platform team can review for reuse.

Use **Import from alert rule** as a starting point for signal criteria. An imported rule doesn't become a live telemetry source. Health-model alerts target an entity's health state rather than each individual signal.

1. Compare existing alert criteria with the concern's health conditions. Keep [Azure Monitor baseline alerts](https://azure.github.io/azure-monitor-baseline-alerts/welcome/) as the resource-monitoring reference.
2. Choose which entity states need notifications. Keep existing monitoring during validation and review duplicate notifications before changing alert ownership.

See [signal assignment options](/azure/azure-monitor/health-models/signals) and [health-model alerts and migration](/azure/azure-monitor/health-models/alerts).

## Tune the model against observed failures

Review the model after incidents and planned changes. Keep entity scope aligned with ownership and the behavior the modeled concern requires.

1. Check whether the affected situation and its shared dependencies show different health states for a useful reason.
2. Review the impact of scoped failures on the domain root. In the connectivity example, a limited path failure doesn't mean every connection is unavailable.
3. Use health history to justify changes to signal criteria or dependency settings.

## Validate the concept in a sandbox

A nonproduction exercise checks the model's interpretation before the team relies on it operationally.

Choose an isolated capability or situation with existing evidence. For the connectivity example, use a test connection. Agree on expected observations rather than a provisioning template.

1. Observe a successful required connection. Compare its traffic evidence with the relevant signal and entity states.
2. Observe a failed check for a named path. Compare its scoped impact with the rest of the connectivity model.
3. Review a telemetry gap and recovery. Check how the signal reports insufficient data and how its dependencies affect parent health.

Use the [ALZ testing approach](../../enterprise-scale/testing-approach.md) for test boundaries. Use the [health-model creation guidance](/azure/azure-monitor/health-models/create) for supported setup and required access.
