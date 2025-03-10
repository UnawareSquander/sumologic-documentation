---
id: azure-cosmos-db
title: Azure Cosmos DB
description: Learn about the Sumo Logic collection process for the Azure Cosmos DB service.
---

import useBaseUrl from '@docusaurus/useBaseUrl';

<img src={useBaseUrl('img/integrations/microsoft-azure/azure-cosmos-db.png')} alt="Thumbnail icon" width="50"/>

[Azure Cosmos DB](https://learn.microsoft.com/en-us/azure/cosmos-db/introduction) is a fully managed NoSQL and relational database for modern app development offering single-digit millisecond response times, automatic and instant scalability, along with guaranteed speed at any scale. This integration helps in monitoring the overall performance, failures, capacity, and operational health of all your Azure Cosmos DB resources.

The below instructions applies to the following [database APIs](https://learn.microsoft.com/en-us/azure/cosmos-db/choose-api):

* NoSQL
* MongoDB
* Cassandra
* Gremlin
* Table

## Log and Metric types

For Azure Cosmos DB, you can collect the following logs and metrics:

* **Resource logs**. To know more about the different resource log category types and schemas collected for Azure Cosmos DB, refer to the [Azure documentation](https://learn.microsoft.com/en-us/azure/cosmos-db/monitor-reference#resource-logs).
* **Platform Metrics for Azure Cosmos DB**. All metrics for Azure Cosmos DB are in the namespace `Azure Cosmos DB standard metrics`. For more information on supported metrics and dimensions, refer to the [Azure documentation](https://learn.microsoft.com/en-us/azure/cosmos-db/monitor-reference#metrics).

## Setup

Azure service sends monitoring data to Azure Monitor, which can then [stream data to Eventhub](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/stream-monitoring-data-event-hubs). Sumo Logic supports:

* Logs collection from [Azure Monitor](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/monitoring-get-started) using our [Azure Event Hubs source](/docs/send-data/hosted-collectors/cloud-to-cloud-integration-framework/azure-event-hubs-source/).
* Metrics collection using our [HTTP Logs and Metrics source](/docs/send-data/collect-from-other-data-sources/azure-monitoring/collect-metrics-azure-monitor/) via Azure Functions deployed using the ARM template.

You must explicitly enable diagnostic settings for each Azure Cosmos DB namespace you want to monitor. You can forward logs to the same event hub provided they satisfy the limitations and permissions as described [here](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/diagnostic-settings?tabs=portal#destination-limitations).

When you configure the event hubs source or HTTP source, plan your source category to ease the querying process. A hierarchical approach allows you to make use of wildcards. For example: `Azure/CosmosDB/Logs`, `Azure/CosmosDB/Metrics`.

### Configure metrics collection

In this section, you will configure a pipeline for shipping metrics from Azure Monitor to an Event Hub, on to an Azure Function, and finally to an HTTP Source on a hosted collector in Sumo Logic.

1. [Configure an HTTP Source](/docs/send-data/collect-from-other-data-sources/azure-monitoring/collect-metrics-azure-monitor/#step-1-configure-an-http-source).
2. [Configure and deploy the ARM Template](/docs/send-data/collect-from-other-data-sources/azure-monitoring/collect-metrics-azure-monitor/#step-2-configure-azure-resources-using-arm-template).
3. [Export metrics to Event Hub](/docs/send-data/collect-from-other-data-sources/azure-monitoring/collect-metrics-azure-monitor/#step-3-export-metrics-for-a-particular-resource-to-event-hub). Perform below steps for each Azure Cosmos DB namespace that you want to monitor.
   * Choose `Stream to an event hub` as destination.
   * Select `AllMetrics`.
   * Use the Event hub namespace created by the ARM template in Step 2 above. You can create a new Event hub or use the one created by ARM template. You can use the default policy `RootManageSharedAccessKey` as the policy name.

### Configure logs collection

In this section, you will configure a pipeline for shipping diagnostic logs from Azure Monitor to an Event Hub.

1. To set up the Azure Event Hubs cloud-to-cloud source in Sumo Logic portal, refer to our [Azure Event Hubs source documentation](/docs/send-data/hosted-collectors/cloud-to-cloud-integration-framework/azure-event-hubs-source/).
2. If you want to audit Azure Cosmos DB control plane operations, [disable the key based metadata write access](https://learn.microsoft.com/en-us/azure/cosmos-db/audit-control-plane-logs#disable-key-based-metadata-write-access).
3. To create the Diagnostic settings in Azure portal, refer to the [Azure documentation](https://learn.microsoft.com/en-us/azure/cosmos-db/monitor-resource-logs?tabs=azure-portal#create-diagnostic-settings). Perform below steps for each Azure Cosmos DB namespace that you want to monitor.
   * Choose `Stream to an event hub` as the destination.
   * Select your preferred log categories depending upon your database API.
   * Use the Event hub namespace and Event hub name configured in previous step in destination details section. You can use the default policy `RootManageSharedAccessKey` as the policy name.

## Troubleshooting

### Azure Event Hubs Source

Common error types are described [here](/docs/send-data/hosted-collectors/cloud-to-cloud-integration-framework/azure-event-hubs-source/#error-types).

You can try [restarting](/docs/send-data/hosted-collectors/cloud-to-cloud-integration-framework/azure-event-hubs-source/#restarting-your-source) the source for `ThirdPartyConfig` errors.

### HTTP Logs and Metrics Source used by Azure Functions

To troubleshoot metrics collection, follow the instructions in [Collect Metrics from Azure Monitor > Troubleshooting metrics collection](/docs/send-data/collect-from-other-data-sources/azure-monitoring/collect-metrics-azure-monitor/#troubleshooting-metrics-collection).
