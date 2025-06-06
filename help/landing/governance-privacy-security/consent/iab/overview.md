---
keywords: Experience Platform;home;IAB;IAB 2.0;consent;Consent
solution: Experience Platform
title: IAB TCF 2.0 Support in Experience Platform
description: Learn how to configure your data operations and schemas to convey customer consent choices when activating segments to destinations in Adobe Experience Platform.
role: Developer
feature: Consent
exl-id: af787adf-b46e-43cf-84ac-dfb0bc274025
---
# IAB TCF 2.0 support in Experience Platform

The [!DNL Transparency & Consent Framework] (TCF), as outlined by the [!DNL Interactive Advertising Bureau] (IAB) is an open-standard technical framework intended to enable organizations to obtain, record, and update consumer consent for the processing of their personal data, in compliance with the European Union's [!DNL General Data Protection Regulation] (GDPR). The second iteration of the framework, TCF 2.0, grants more flexibility for how consumers can provide or withhold consent, including whether and how vendors may use certain features of data processing, such as precise geolocation.

>[!NOTE]
>
>More information on TCF 2.0 can be found on the [IAB Europe website](https://iabeurope.eu/), including support materials and technical specifications.

Adobe Experience Platform is part of the registered [IAB TCF 2.0 vendor list](https://iabeurope.eu/vendor-list-tcf/), under the ID **565**. In compliance with TCF 2.0 requirements, Experience Platform allows you to collect customer consent data and integrate it into your stored customer profiles. This consent data can then be factored into whether profiles are included in exported audience segments, depending on their use case.

>[!IMPORTANT]
>
>Experience Platform is only able to comply with version 2.0 of the TCF (or greater). Previous versions of TCF are not supported.

This document provides an overview of how to configure your data operations and profile schemas to accept customer consent data generated by your Consent Management Platform (CMP). It also covers how Experience Platform conveys user consent choices when exporting segments.

## Prerequisites

To follow along with this guide, you must be using a CMP, either commercial or your own, that is integrated and compliant with the IAB TCF. See the [list of compliant CMPs](https://iabeurope.eu/cmp-list/) for more information.

>[!IMPORTANT]
>
>If the ID of your CMP is invalid, Experience Platform keeps processing your data as-is. To enforce TCF 2.0, you must confirm that your CMP has a valid ID that has been registered with IAB TCF 2.0 before sending data to Experience Platform.

This guide also requires a working understanding of the following Experience Platform services:

* [Experience Data Model (XDM)](/help/xdm/home.md): The standardized framework by which Experience Platform organizes customer experience data.
* [Adobe Experience Platform Identity Service](/help/identity-service/home.md): Solves the fundamental challenge posed by the fragmentation of customer experience data by bridging identities across devices and systems.
* [Real-Time Customer Profile](/help/profile/home.md): Uses [!DNL Identity Service] to create detailed customer profiles from your datasets in real time. [!DNL Real-Time Customer Profile] pulls data from the Data Lake and persists customer profiles in its own separate data store.
* [Adobe Experience Platform Web SDK](/help/web-sdk/home.md): A client-side JavaScript library that allows you to integrate various Experience Platform services into your customer-facing website.
    * [SDK consent commands](../../../../web-sdk/commands/setconsent.md): A use-case overview of the consent-related SDK commands shown in this guide.
* [Adobe Experience Platform Segmentation Service](/help/segmentation/home.md): Allows you to divide [!DNL Real-Time Customer Profile] data into groups of individuals that share similar traits and responds similarly to marketing strategies.

In addition to the Experience Platform services listed above, you should also be familiar with [destinations](/help/data-governance/home.md) and their role in the Experience Platform ecosystem.

## Customer consent flow summary {#summary}

The following sections describe how consent data is collected and enforced after the system has been properly configured.

### Consent data collection

Experience Platform allows you to collect customer consent data through the following process:

1. A customer provides their consent preferences for data collection through a dialog on your website.
1. Your CMP detects the consent preference change, and generates TCF consent data accordingly.
1. Using the Experience Platform Web SDK, the generated consent data (returned by the CMP) is sent to Adobe Experience Platform.
1. The collected consent data is ingested into a [!DNL Profile]-enabled dataset whose schema contains TCF consent fields.

In addition to SDK commands triggered by CMP consent-change hooks, consent data can also flow into Experience Platform through any customer-generated XDM data that is uploaded directly to a [!DNL Profile]-enabled dataset.

Any segments shared with Experience Platform by Adobe Audience Manager (through the [!DNL Audience Manager] source connector or otherwise) may also contain consent data if the appropriate fields have been applied to those segments through [!DNL Experience Cloud Identity Service]. For more information on collecting consent data in [!DNL Audience Manager], see the document on the [Adobe Audience Manager plug-in for IAB TCF](https://experienceleague.adobe.com/docs/audience-manager/user-guide/overview/data-privacy/consent-management/aam-iab-plugin.html).

### Downstream consent enforcement

Once TCF consent data has successfully been ingested, the following processes take place in downstream Experience Platform services:

1. [!DNL Real-Time Customer Profile] updates the stored consent data for that customer's profile.
1. Experience Platform processes customer IDs only if the vendor permission for Experience Platform (565) is provided for every ID in a cluster.
1. When exporting segments to destinations belonging to members of the TCF 2.0 vendor list, Experience Platform only includes profiles if the vendor permissions for both Experience Platform (565) *and* the individual destination are provided for every ID in a cluster.

The rest of the sections in this document provide guidance on how to configure Experience Platform and your data operations to fulfill the collection and enforcement requirements described above.

## Determine how to generate customer consent data within your CMP {#consent-data}

Since each CMP system is unique, you must determine the best way to allow your customers to provide consent as they interact with your service. A cookie consent dialog is a common way to attain customer consent. An example CMP dialog is seen below.

![An example Consent Management Platform dialog.](../../../images/governance-privacy-security/consent/iab/overview/cmp-dialog.png)

This dialog must allow the customer to opt in or out of the following:

| Consent option | Description |
| --- | --- |
| **Purposes** | Purposes define which ad tech purposes a brand can use a customer's data for. The following purposes must be opted into for Experience Platform to process customer IDs: <ul><li>**Purpose 1**: Store and/or access information on a device</li><li>**Purpose 10**: Develop and improve products</li></ul> |
| **Vendor permissions** | In addition to ad tech purposes, the dialog must also allow the customer to opt in or out of having their data used by specific vendors, including Adobe Experience Platform (565). |

### Consent strings {#consent-strings}

Regardless of the method you use to collect the data, the goal is to generate a string value based on the consent options chosen by the customer, called a consent string.

In the TCF specification, consent strings are used to encode relevant details about a customer's consent settings, in terms of specific marketing purposes as defined by policies and vendors. Experience Platform uses these strings to store the consent settings for each customer, and therefore a new consent string must be generated each time those settings change.

Consent strings may only be created by a CMP that is registered with the IAB TCF. For more information on how to generate consent strings using your particular CMP, refer to the [consent string formatting guide](https://github.com/InteractiveAdvertisingBureau/GDPR-Transparency-and-Consent-Framework/blob/master/TCFv2/IAB%20Tech%20Lab%20-%20Consent%20string%20and%20vendor%20list%20formats%20v2.md) in the IAB TCF GitHub repo.

## Create datasets with TCF consent fields {#datasets}

Customer consent data must be sent to datasets whose schemas contain TCF consent fields. Refer to the tutorial on [creating datasets for capturing TCF 2.0 consent](./dataset.md) for how to create the required profile dataset (and an optional Experience Event dataset) before continuing with this guide.

## Update [!DNL Profile] merge policies to include consent data {#merge-policies}

Once you have created a [!DNL Profile]-enabled dataset for collecting consent data, you must ensure that your merge policies have been configured to always include TCF consent fields in your customer profiles. This involves setting dataset precedence so that your consent dataset is prioritized over other potentially conflicting datasets.

For more information on how to work with merge policies, refer to the [merge policies overview](/help/profile/merge-policies/overview.md). When setting up your merge policies, you must ensure that your segments include all the required consent attributes provided by the [XDM privacy schema field group](./dataset.md#privacy-field-group), as outlined in the guide on dataset preparation.

## Integrate the Experience Platform Web SDK to collect customer consent data {#sdk}

>[!NOTE]
>
>The use of the Experience Platform Web SDK is required to process consent data directly in Adobe Experience Platform. [!DNL Experience Cloud Identity Service] is not supported.
>
>[!DNL Experience Cloud Identity Service] is still supported for consent processing in Adobe Audience Manager, however, and compliance with TCF 2.0 only requires that the library is updated to [version 5.0](https://github.com/Adobe-Marketing-Cloud/id-service/releases).

Once you have configured your CMP to generate consent strings, you must integrate the Experience Platform Web SDK to collect those strings and send them to Experience Platform. The Experience Platform SDK provides two commands that can be used to send TCF consent data to Experience Platform (explained in the subsections below). These commands should be used when a customer provides consent information for the first time, and anytime that consent changes thereafter.

**The SDK does not interface with any CMPs out of the box**. It is up to you to determine how to integrate the SDK into your website, listen for consent changes in the CMP, and call the appropriate command. 

### Create a datastream

In order for the SDK to send data to Experience Platform, you must first create a datastream for Experience Platform. Specific steps for how to create a datastream are provided in the [SDK documentation](/help/datastreams/overview.md).

After providing a unique name for the datastream, select the toggle button next to **[!UICONTROL Adobe Experience Platform]**. Next, use the following values to complete the rest of the form:

| Datastream field | Value |
| --- | --- |
| [!UICONTROL Sandbox] | The name of the Experience Platform [sandbox](/help/sandboxes/home.md) that contains the required streaming connection and datasets to set up the datastream. |
| [!UICONTROL Streaming Inlet] | A valid streaming connection for Experience Platform. See the tutorial on [creating a streaming connection](/help/ingestion/tutorials/create-streaming-connection-ui.md) if you do not have an existing streaming inlet. |
| [!UICONTROL Event Dataset] | Select the [!DNL XDM ExperienceEvent] dataset created in the [previous step](#datasets). If you included the [[!UICONTROL IAB TCF 2.0 Consent] field group](/help/xdm/field-groups/event/iab.md) in this dataset's schema, you can track consent-change events over time using the [`sendEvent`](#sendEvent) command, storing that data in this dataset. Keep in mind that the consent values stored in this dataset are **not** used in automatic enforcement workflows. |
| [!UICONTROL Profile Dataset] | Select the [!DNL XDM Individual Profile] dataset created in the [previous step](#datasets). When responding to CMP consent-change hooks using the [`setConsent`](#setConsent) command, collected data is stored in this dataset. Since this dataset is Profile-enabled, the consent values stored in this dataset are honored during automatic enforcement workflows. |

![](../../../images/governance-privacy-security/consent/iab/overview/edge-config.png)

When finished, select **[!UICONTROL Save]** at the bottom of the screen and continue following any additional prompts to complete the configuration.

### Making consent-change commands

Once you have created the datastream described in the previous section, you can start using SDK commands to send consent data to Experience Platform. The sections below provide examples of how each SDK command can be used in different scenarios.

#### Using CMP consent-change hooks {#setConsent}

Many CMPs provide out-of-the-box hooks that listen to consent-change events. When these events occur, you can use the [`setConsent`](/help/web-sdk/commands/setconsent.md) command to update that customer's consent data.

The `setConsent` command expects two arguments: 

1. A string that indicates the command type (in this case, "setConsent").
1. A payload that contains a `consent` array. The array must contain at least one object that provides the required consent fields. 

The `setConsent` command is displayed below:

```js
alloy("setConsent", {
  consent: [{
    standard: "IAB TCF",
    version: "2.0",
    value: "CLcVDxRMWfGmWAVAHCENAXCkAKDAADnAABRgA5mdfCKZuYJez-NQm0TBMYA4oCAAGQYIAAAAAAEAIAEgAA.argAC0gAAAAAAAAAAAA",
    gdprApplies: "true"
  }]
});
```

| Payload property | Description |
| --- | --- |
| `standard` | The consent standard being used. This value must be set to `IAB` for TCF 2.0 consent processing. |
| `version` | The version number of the consent standard indicated under `standard`. This value must be set to `2.0` for TCF 2.0 consent processing. |
| `value` | The base-64-encoded consent string generated by the CMP. |
| `gdprApplies` | A Boolean value that indicates whether the GDPR applies to the currently logged-in customer. For TCF 2.0 to be enforced for this customer, the value must be set to `true`. Defaults to `true` if not defined. |

The `setConsent` command should be used as part of a CMP hook that detects changes in consent settings. The following JavaScript provides an example of how the `setConsent` command can be used for OneTrust's `OnConsentChanged` hook:

```js
OneTrust.OnConsentChanged(function () {
  // Retrieve the TCF 2.0 consent data generated by the CMP, and pass it to Alloy. 
  __tcfapi("getTCData", 2, function (data, success) {
    if (success) {
      var tcString = data.tcString;
      var gdpr = data.gdprApplies;

      alloy("setConsent", {
        consent: [{
          standard: "IAB TCF",
          version: "2.0",
          value: tcString,
          gdprApplies: gdpr
        }]
      });
    }
  });
});
```

#### Using events {#sendEvent}

You can also collect TCF 2.0 consent data on every event triggered in Experience Platform by using the `sendEvent` command.

>[!NOTE]
>
>To use this method, you must have added the Experience Event Privacy field group to your [!DNL Profile]-enabled [!DNL XDM ExperienceEvent] schema. See the section on [updating the ExperienceEvent schema](./dataset.md#event-schema) in the dataset preparation guide for steps on how to configure this.

The `sendEvent` command should be used as a callback in appropriate event listeners on your website. The command expects two arguments: (1) a string that indicates the command type (in this case, `sendEvent`), and (2) a payload containing an `xdm` object that provides the required consent fields as JSON:

```js
alloy("sendEvent", {
  xdm: {
    "consentStrings": [{
      "consentStandard": "IAB TCF",
      "consentStandardVersion": "2.0",
      "consentStringValue": "CLcVDxRMWfGmWAVAHCENAXCkAKDAADnAABRgA5mdfCKZuYJez-NQm0TBMYA4oCAAGQYIAAAAAAEAIAEgAA.argAC0gAAAAAAAAAAAA",
      "gdprApplies": true
    }]
  }
});
```

| Payload property | Description |
| --- | --- |
| `xdm.consentStrings` | An array that must contain at least one object that provides the required consent fields. |
| `consentStandard` | The consent standard being used. This value must be set to `IAB` for TCF 2.0 consent processing. |
| `consentStandardVersion` | The version number of the consent standard indicated under `standard`. This value must be set to `2.0` for TCF 2.0 consent processing. |
| `consentStringValue` | The base-64-encoded consent string generated by the CMP. |
| `gdprApplies` | A Boolean value that indicates whether the GDPR applies to the currently logged-in customer. For TCF 2.0 to be enforced for this customer, the value must be set to `true`. Defaults to `true` if not defined. |

### Handling SDK responses

Many Web SDK commands return promises that indicate whether the call succeeded or failed. You can then use these responses for additional logic such as displaying confirmation messages to the customer. See [Command responses](/help/web-sdk/commands/command-responses.md) for more information.

## Export segments {#export}

>[!NOTE]
>
>Before you start exporting segments, you must ensure that your segments include all required consent fields. See the section on [configuring merge policies](#merge-policies) for more information.

Once you have collected customer consent data and have created audience segments containing the required consent attributes, you can then enforce TCF 2.0 compliance when exporting those segments to downstream destinations.

If the consent setting `gdprApplies` is set to `true` for a set of customer profiles, any data from those profiles that is exported to downstream destinations is filtered based on the TCF consent preferences for each profile. Any profile that does not meet the required consent preferences is skipped during the export process.

Customers must consent to the following purposes (as outlined by [TCF 2.0 policies](https://iabeurope.eu/iab-europe-transparency-consent-framework-policies/#Appendix_A_Purposes_and_Features_Definitions)) for their profiles to be included in segments that are exported to destinations:

* **Purpose 1**: Store and/or access information on a device
* **Purpose 10**: Develop and improve products

TCF 2.0 also requires that the source of data must check the destination's vendor permission before sending data to that destination. As such, Experience Platform checks if the destination's vendor permission is opted in to for all IDs in the cluster before including data bound to that destination.

>[!NOTE]
>
>Any segments that are shared with Adobe Audience Manager contain the same TCF 2.0 consent values as their Experience Platform counterparts. Since [!DNL Audience Manager] shares the same vendor ID as Experience Platform (565), the same purposes and vendor permission are required. See the document on the [Adobe Audience Manager plug-in for IAB TCF](https://experienceleague.adobe.com/docs/audience-manager/user-guide/overview/data-privacy/consent-management/aam-iab-plugin.html) for more information.

## Test your implementation {#test-implementation}

Once you have configured your TCF 2.0 implementation and have exported segments to destinations, any data that does not meet consent requirements will not be exported. To see whether the correct customer profiles were filtered during the export, you must manually check the data stores on your destinations to see if consent was properly enforced.

>[!IMPORTANT]
>
>If multiple IDs make up a cluster and TCF 2.0 applies, the entire cluster is excluded if even a single ID does not contain the correct purposes and vendor permission(s).

## Next steps

This document covered the process of configuring your Experience Platform data operations to meet your business obligations as outlined by the TCF 2.0. See the overview on [governance, privacy, and security](../../overview.md) for more information Experience Platform's privacy-related capabilities.
