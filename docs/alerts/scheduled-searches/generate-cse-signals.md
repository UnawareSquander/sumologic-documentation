---
id: generate-cse-signals
title: Generate Cloud SIEM Signals With a Scheduled Search
sidebar_label: Generate Cloud SIEM Signals With a Scheduled Search
description: You can generate a Cloud SIEM Signal with a scheduled search.
---

import useBaseUrl from '@docusaurus/useBaseUrl';

This page has information about creating a scheduled search that will trigger a Cloud SIEM Signal. Before you start using scheduled searches to create Cloud SIEM Signals, it is helpful to understand what Signals are, and how they relate to the generation of Cloud SIEM Insights. For information about how it all works see [Insight Generation Process](/docs/cse/get-started-with-cloud-siem/insight-generation-process/). 

:::note
For a more detailed description of the options you can configure for a scheduled search, see [Schedule a Search](schedule-search.md).
:::

## Requirements for the search query

This section describes the requirements for your scheduled search, which include a minimum set of fields to be returned, and renaming message fields as necessary to match attribute names in the selected Cloud SIEM Record type schema.  

### Required fields

There are several fields that your scheduled search must return to
enable Signal generation:

* `normalizedseverity`. This field must contain a value between (and including) 0 and 10. Signals generated by the scheduled search will have this severity value. SIgnal severity values are used by Cloud SIEM’s Insight generation algorithm, as described above. 
* `stage`. This field must contain a Tactic in the MITRE ATT&CK framework, one of the following:
  * Collection
  * Command and Control
  * Credential Access
  * Defense Evasion
  * Discovery
  * Execution
  * Exfiltration
  * Impact
  * Initial Access
  * Lateral Movement
  * Persistence
  * Privilege Escalation
  * Reconnaissance
  * Resource Development
    :::important
    If the `stage` field contains a Tactic that isn't in the MITRE ATT&CK framework, a Signal will not be generated, but a Record will be. 
    :::
* At least one entity field:

  * `device_ip`
  * `device_mac`
  * `device_natIp`
  * `dns_replyIp`
  * `dstDevice_hostname`
  * `dstDevice_ip`
  * `dstDevice_mac`
  * `dstDevice_natIp`
  * `fromUser_username`
  * `srcDevice_hostname`
  * `srcDevice_ip`
  * `srcDevice_mac`
  * `srcDevice_natIp`
  * `user_username`        

### Renaming message fields

When you configure a Scheduled Search to create Cloud SIEM Signals, you are prompted to select a [Cloud SIEM Record type](../../cse/schema/cse-record-types.md). The fields returned by your search must match an attribute in the Record
type you select. A field whose name does not match a Cloud SIEM attribute will not be populated in the Record created from the Schedule Search results. For more about Cloud SIEM attribute names, see [Attributes You Can Map to Records](../../cse/schema/attributes-map-to-records.md).

## Scheduling the search

1. After creating and saving your search, click the save icon.<br/><img src={useBaseUrl('img/alerts/save-as.png')} alt="Save the search" width="800"/>
1. The **Save Item** popup appears. 
   :::note
   The name of your scheduled search will appear as the signal name in Cloud SIEM.
   ::: 
   <br/><img src={useBaseUrl('img/alerts/save-item.png')} alt="Save as scheduled search" width="600"/>
1. Click **Schedule this search**.
1. The **Save Item** popup prompts you to select a run frequency.<br/> ![run-frequency.png](/img/alerts/run-freq-signal-gen.png)
1. Select a frequency from the pull-down list and click **Save**.  Scheduling a run frequency that matches your query time range will reduce overlapping searches and duplicate alerts. When you have a search scheduled to run over the same results as a previously scheduled search you would trigger an alert on the same data. 
1. The popup refreshes.<br/> ![options.png](/img/alerts/options.png)
1. **Time range for scheduled search.** Indicates the time range your query will use to execute, which impacts the results generated by the query.
   :::note
   This setting is different than the Time Range option configured for the Saved Search. The first time range is only used when you run the Saved Search from the Library. This Time Range applies to your Scheduled Search.
   :::
1. **Timezone for scheduled search**. Select the time zone you would like your scheduled search to use. The schedule's time is based on this time zone. This time zone is not related to the time zone of your data. If you don't make a selection, the scheduled search will use the time zone from your browser, which is the default selection.
1. **Send notification**. Select **If the following condition is met**, and enter an alert condition and the number of results that should trigger the alert.
1. **Alert Type**. Select **Cloud SIEM Signal**.
1. The popup refreshes.<br/> ![alert-type-selected.png](/img/alerts/alert-type-selected.png)
1. **Record Type**. Select a [Record Type](../../cse/schema/cse-record-types.md).
1. Click **Save**.

## View Signals in Cloud SIEM

To view Signals that were created from a scheduled search, run a keyword search on “CIP Scheduled Search” on the **Signals** page in the Cloud SIEM UI.

Below is a screenshot of a Signal that was created from a scheduled search. Note that:

* The **Mapping** section at the bottom of the page shows that the Signal was the result of a scheduled search.
* If the Signal is not part of an Insight, there’s a **Create Insight** link you can use to create an Insight for the Signal. For more information, see [Create an Insight from Signal](#create-an-insight-from-signal).
* You can click the **Full Details** link for more information about the Signal. See [View Signal details](#view-signal-details) below for a screenshot.

![ss-signal.png](/img/alerts/ss-signal.png)

## View Signal details

The **Full Details** tab displays details about the Signal.

![full-details.png](/img/alerts/full-details.png)

## Create an Insight from Signal

To create an Insight from a Signal generated from a scheduled search:

1. Navigate to a Signal that was generated from a scheduled search.
1. Click **Create Insight**. 
1. Click **Yes, Create Insight** when prompted whether you want to proceed. <br/> ![confirm-create.png](/img/alerts/confirm-create.png)
1. The new Insight is created and appears as a **Related Insight**. <br/> ![new-related-insight.png](/img/alerts/new-related-insight.png)
