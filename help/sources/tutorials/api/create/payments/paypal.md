---
keywords: Experience Platform;home;popular topics;PayPal connector;paypal;Paypal
solution: Experience Platform
title: Create a PayPal Base Connection Using the Flow Service API
type: Tutorial
description: Learn how to connect PayPal to Adobe Experience Platform using the Flow Service API.
exl-id: 5e6ca7b4-5e2f-4706-a339-ac159e2e0938
---
# Create a [!DNL PayPal] base connection using the [!DNL Flow Service] API

>[!WARNING]
>
>The [!DNL PayPal] source will be deprecated at the end of June 2025.

A base connection represents the authenticated connection between a source and Adobe Experience Platform.

This tutorial walks you through the steps to create a base connection for [!DNL PayPal] using the [[!DNL Flow Service] API](https://www.adobe.io/experience-platform-apis/references/flow-service/).

## Getting started

This guide requires a working understanding of the following components of Adobe Experience Platform:

* [Sources](../../../../home.md): [!DNL Experience Platform] allows data to be ingested from various sources while providing you with the ability to structure, label, and enhance incoming data using [!DNL Experience Platform] services.
* [Sandboxes](../../../../../sandboxes/home.md): [!DNL Experience Platform] provides virtual sandboxes which partition a single Experience Platform instance into separate virtual environments to help develop and evolve digital experience applications.

The following sections provide additional information that you will need to know in order to successfully connect to [!DNL PayPal] using the [!DNL Flow Service] API.

### Gather required credentials

In order for [!DNL Flow Service] to connect with [!DNL PayPal], you must provide values for the following connection properties:

| Credential | Description |
| ---------- | ----------- |
| `host` | The URL of the [!DNL PayPal] instance. (default: api.sandbox.paypal.com). |
| `clientId` | The client ID associated with your [!DNL PayPal] application. |
| `clientSecret` | The client secret associated with your [!DNL PayPal] application. |
| `connectionSpec.id` | The connection specification returns a source's connector properties, including authentication specifications related to creating the base and source connections. The connection specification ID for [!DNL PayPal] is: `221c7626-58f6-4eec-8ee2-042b0226f03b` |

For more information about getting started refer to [this PayPal document](https://developer.paypal.com/docs/api/overview/#get-credentials).

### Using Experience Platform APIs

For information on how to successfully make calls to Experience Platform APIs, see the guide on [getting started with Experience Platform APIs](../../../../../landing/api-guide.md).

## Create a base connection

A base connection retains information between your source and Experience Platform, including your source's authentication credentials, the current state of the connection, and your unique base connection ID. The base connection ID allows you to explore and navigate files from within your source and identify the specific items that you want to ingest, including information regarding their data types and formats.

To create a base connection ID, make a POST request to the `/connections` endpoint while providing your [!DNL PayPal] authentication credentials as part of the request parameters.

**API format**

```http
POST /connections
```

**Request**

The following request creates a base connection for [!DNL PayPal]:

```shell
curl -X POST \
    'https://platform.adobe.io/data/foundation/flowservice/connections' \
    -H 'Authorization: Bearer {ACCESS_TOKEN}' \
    -H 'x-api-key: {API_KEY}' \
    -H 'x-gw-ims-org-id: {ORG_ID}' \
    -H 'x-sandbox-name: {SANDBOX_NAME}' \
    -H 'Content-Type: application/json' \
    -d '{
        "name": "Paypal connection",
        "description": "Paypal connection",
        "auth": {
        "specName": "Client-Id-Secret Based Authentication",
        "params": {
            "host": "{HOST}",
            "clientId": "{CLIENT_ID}",
            "clientSecret": "{CLIENT_SECRET}"
            }
        },
        "connectionSpec": {
            "id": "221c7626-58f6-4eec-8ee2-042b0226f03b",
            "version": "1.0"
        }
    }'
```

| Property | Description |
| --------- | ----------- |
| `auth.params.host` | The URL of the [!DNL PayPal] instance. |
| `auth.params.clientId` | The client ID associated with your [!DNL PayPal] instance. |
| `auth.params.clientSecret` | The client secret associated with your [!DNL PayPal] instance. |
| `connectionSpec.id` | The [!DNL PayPal] connection specification ID: `221c7626-58f6-4eec-8ee2-042b0226f03b`. |

**Response**

A successful response returns the newly created connection, including its unique connection identifier (`id`). This ID is required to explore your data in the next tutorial.

```json
{
    "id": "24151d58-ffa7-4960-951d-58ffa7396097",
    "etag": "\"65015e9d-0000-0200-0000-5e89162d0000\""
}
```

## Next steps

By following this tutorial, you have created a [!DNL PayPal] base connection using the [!DNL Flow Service] API. You can use this base connection ID in the following tutorials:

* [Explore the structure and contents of your data tables using the [!DNL Flow Service] API](../../explore/tabular.md)
* [Create a dataflow to bring payments data to Experience Platform using the [!DNL Flow Service] API](../../collect/payments.md)
