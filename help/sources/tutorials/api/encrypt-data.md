---
title: Encrypted Data Ingestion
description: Learn how to ingest encrypted files through cloud storage batch sources using the API.
exl-id: 83a7a154-4f55-4bf0-bfef-594d5d50f460
---
# Encrypted data ingestion

You can ingest encrypted data files to Adobe Experience Platform using cloud storage batch sources. With encrypted data ingestion, you can leverage asymmetric encryption mechanisms to securely transfer batch data into Experience Platform. Currently, the supported asymmetric encryption mechanisms are PGP and GPG.

The encrypted data ingestion process is as follows:

1. [Create an encryption key pair using Experience Platform APIs](#create-encryption-key-pair). The encryption key pair consists of a private key and a public key. Once created, you can copy or download the public key, alongside its corresponding public key ID and Expiry Time. During this process, the private key will be stored by Experience Platform in a secure vault. **NOTE:** The public key in the response is Base64-encoded and must be decoded prior to using.
2. Use the public key to encrypt the data file that you want to ingest.
3. Place your encrypted file in your cloud storage.
4. Once the encrypted file is ready, [create a source connection and a dataflow for your cloud storage source](#create-a-dataflow-for-encrypted-data). During the flow creation step, you must provide an `encryption` parameter and include your public key ID. 
5. Experience Platform retrieves the private key from the secure vault to decrypt the data at the time of ingestion.

>[!IMPORTANT]
>
>The maximum size of a single encrypted file is 1 GB. For example, you can ingest 2 GBs worth of data in a single dataflow run, however, any individual file in that data cannot exceed 1 GB.

This document provides steps on how to generate a encryption key pair to encrypt your data, and ingest that encrypted data to Experience Platform using cloud storage sources.

## Get started {#get-started}

This tutorial requires you to have a working understanding of the following components of Adobe Experience Platform:

* [Sources](../../home.md): Experience Platform allows data to be ingested from various sources while providing you with the ability to structure, label, and enhance incoming data using Experience Platform services.
  * [Cloud storage sources](../api/collect/cloud-storage.md): Create a dataflow to bring batch data from your cloud storage source to Experience Platform.
* [Sandboxes](../../../sandboxes/home.md): Experience Platform provides virtual sandboxes which partition a single Experience Platform instance into separate virtual environments to help develop and evolve digital experience applications.

### Using Experience Platform APIs

For information on how to successfully make calls to Experience Platform APIs, see the guide on [getting started with Experience Platform APIs](../../../landing/api-guide.md).

### Supported file extensions for encrypted files {#supported-file-extensions-for-encrypted-files}

The list of supported file extensions for encrypted files are:

* .csv
* .tsv
* .json
* .parquet
* .csv.gpg
* .tsv.gpg
* .json.gpg
* .parquet.gpg
* .csv.pgp
* .tsv.pgp
* .json.pgp
* .parquet.pgp
* .gpg
* .pgp

>[!NOTE]
>
>Encrypted file ingestion in Adobe Experience Platform Sources supports openPGP and not any specific proprietary version of PGP.

## Create encryption key pair {#create-encryption-key-pair}

>[!IMPORTANT]
>
>Encryption keys are specific to a given sandbox. Therefore, you must create new encryption keys if you want to ingest encrypted data in a different sandbox, within your organization.

The first step in ingesting encrypted data to Experience Platform is to create your encryption key pair by making a POST request to the `/encryption/keys` endpoint of the [!DNL Connectors] API.

**API format**

```http
POST /data/foundation/connectors/encryption/keys
```

**Request**

+++View example request

The following request generates an encryption key pair using the PGP encryption algorithm.

```shell
curl -X POST \
  'https://platform.adobe.io/data/foundation/connectors/encryption/keys' \
  -H 'Authorization: Bearer {{ACCESS_TOKEN}}' \
  -H 'x-api-key: {{API_KEY}}' \
  -H 'x-gw-ims-org-id: {{ORG_ID}}' \
  -H 'x-sandbox-name: {{SANDBOX_NAME}}' \
  -H 'Content-Type: application/json' 
  -d '{
      "name": "acme-encryption",
      "encryptionAlgorithm": "PGP",
      "params": {
          "passPhrase": "{{PASSPHRASE}}"
      }
  }'
```

| Parameter | Description |
| --- | --- |
| `name` | The name of your encryption key pair. |
| `encryptionAlgorithm` | The type of encryption algorithm that you are using. The supported encryption types are `PGP` and `GPG`. |
| `params.passPhrase` | The passphrase provides an additional layer of protection for your encryption keys. Upon creation, Experience Platform stores the passphrase in a different secure vault from the public key. You must provide a non-empty string as a passphrase. |

+++

**Response**

+++View example response

A successful response returns your Base64-encoded public key, public key ID, and the expiry time of your keys. The expiry time automatically sets to 180 days after the date of key generation. Expiry time is currently not configurable.

```json
{
    ​"publicKey": "{PUBLIC_KEY}",
    ​"publicKeyId": "{PUBLIC_KEY_ID}",
    ​"expiryTime": "1684843168"
}
```

| Property | Description |
| --- | --- |
| `publicKey` | The public key is used to encrypt the data in your cloud storage. This key corresponds with the private key that was also created during this step. However, the private key immediately goes to Experience Platform. |
| `publicKeyId` | The public key ID is used to create a dataflow and ingest your encrypted cloud storage data to Experience Platform. |
| `expiryTime` | The expiry time defines the expiration date of your encryption key pair. This date is automatically set to 180 days after the date of key generation and is displayed in unix timestamp format. |

+++

### Retrieve encryption keys {#retrieve-encryption-keys}

To retrieve all encryption keys in your organization, make a GET Request to the `/encryption/keys` endpoit=nt.

**API format**

```http
GET /data/foundation/connectors/encryption/keys
```

**Request**

+++View example request

The following request retrieves all encryption keys in your organization.

```shell
curl -X GET \
  'https://platform.adobe.io/data/foundation/connectors/encryption/keys' \
  -H 'Authorization: Bearer {{ACCESS_TOKEN}}' \
  -H 'x-api-key: {{API_KEY}}' \
  -H 'x-gw-ims-org-id: {{ORG_ID}}' \
```

+++

**Response**

+++View example response

A successful response returns your encryption algorithm, name, public key, public key ID, key type, and the corresponding expiry time of your keys.

```json
{
    "encryptionAlgorithm": "{ENCRYPTION_ALGORITHM}",
    "name": "{NAME}",
    "publicKeyId": "{PUBLIC_KEY_ID}",
    "publicKey": "{PUBLIC_KEY}",
    "keyType": "{KEY_TYPE}",
    "expiryTime": "{EXPIRY_TIME}"
}
```

+++

### Retrieve encryption keys by ID {#retrieve-encryption-keys-by-id}

To retrieve a specific set of encryption keys, make a GET request to the `/encryption/keys` endpoint and provide your public key ID as a header parameter.

**API format**

```http
GET /data/foundation/connectors/encryption/keys/{PUBLIC_KEY_ID}
```

**Request**

+++View example request

```shell
curl -X GET \
  'https://platform.adobe.io/data/foundation/connectors/encryption/keys/{publicKeyId}' \
  -H 'Authorization: Bearer {{ACCESS_TOKEN}}' \
  -H 'x-api-key: {{API_KEY}}' \
  -H 'x-gw-ims-org-id: {{ORG_ID}}' \
```

+++

**Response**

+++View example response

A successful response returns your encryption algorithm, name, public key, public key ID, key type, and the corresponding expiry time of your keys.

```json
{
    "encryptionAlgorithm": "{ENCRYPTION_ALGORITHM}",
    "name": "{NAME}",
    "publicKeyId": "{PUBLIC_KEY_ID}",
    "publicKey": "{PUBLIC_KEY}",
    "keyType": "{KEY_TYPE}",
    "expiryTime": "{EXPIRY_TIME}"
}
```

+++

### Create customer managed key pair {#create-customer-managed-key-pair}

You can optionally create a sign verification key pair to sign and ingest your encrypted data.

During this stage, you must generate your own private key and public key combination and then use your private key to sign your encrypted data. Next, you must encode your public key in Base64 and then share it to Experience Platform in order for Experience Platform to verify your signature.

### Share your public key to Experience Platform

To share your public key, make a POST request to the `/customer-keys` endpoint while providing your encryption algorithm and your Base64-encoded public key.

**API format**

```http
POST /data/foundation/connectors/encryption/customer-keys
```

**Request**

+++View example request

```shell
curl -X POST \
  'https://platform.adobe.io/data/foundation/connectors/encryption/customer-keys' \
  -H 'Authorization: Bearer {{ACCESS_TOKEN}}' \
  -H 'x-api-key: {{API_KEY}}' \
  -H 'x-gw-ims-org-id: {{ORG_ID}}' \
  -H 'x-sandbox-name: {{SANDBOX_NAME}}' \
  -H 'Content-Type: application/json' 
  -d '{
      "name": "acme-sign-verification-keys"
      "encryptionAlgorithm": {{ENCRYPTION_ALGORITHM}},       
      "publicKey": {{BASE_64_ENCODED_PUBLIC_KEY}},
      "params": {
          "passPhrase": {{PASS_PHRASE}}
      }
    }'
```

| Parameter | Description |
| --- | --- |
| `encryptionAlgorithm` | The type of encryption algorithm that you are using. The supported encryption types are `PGP` and `GPG`. |
| `publicKey` | The public key that corresponds to your customer managed keys used for signing your encrypted. This key must be Base64-encoded.|

+++

**Response**

+++View example response

```json
{    
  "publicKeyId": "e31ae895-7896-469a-8e06-eb9207ddf1c2" 
} 
```

| Property | Description |
| --- | --- |
| `publicKeyId` | This public key ID is returned in response to sharing your customer managed key with Experience Platform. You can provide this public key ID as the sign verification key ID when creating a dataflow for signed and encrypted data. |

+++

### Retrieve customer managed key pair

To retrieve your customer managed keys, make a GET request to the `/customer-keys` endpoint.

**API format**

```http
GET /data/foundation/connectors/encryption/customer-keys
```

**Request**

+++View example request

```shell
curl -X GET \
  'https://platform.adobe.io/data/foundation/connectors/encryption/customer-keys' \
  -H 'Authorization: Bearer {{ACCESS_TOKEN}}' \
  -H 'x-api-key: {{API_KEY}}' \
  -H 'x-gw-ims-org-id: {{ORG_ID}}' \
```

+++

**Response**

+++View example response

```json
[
    {
        "encryptionAlgorithm": "{ENCRYPTION_ALGORITHM}",
        "name": "{NAME}",
        "publicKeyId": "{PUBLIC_KEY_ID}",
        "publicKey": "{PUBLIC_KEY}",
        "keyType": "{KEY_TYPE}",
    }
]
```

+++

## Connect your cloud storage source to Experience Platform using the [!DNL Flow Service] API

Once you have retrieved your encryption key pair, you can now proceed and create a source connection for your cloud storage source and bring your encrypted data to Experience Platform. 

First, you must create a base connection to authenticate your source against Experience Platform. To create a base connection and authenticate your source, select the source you would like to use from the list below:

* [Amazon S3](../api/create/cloud-storage/s3.md)
* [[!DNL Apache HDFS]](../api/create/cloud-storage/hdfs.md)
* [Azure Blob](../api/create/cloud-storage/blob.md)
* [Azure Data Lake Storage Gen2](../api/create/cloud-storage/adls-gen2.md)
* [Azure File Storage](../api/create/cloud-storage/azure-file-storage.md)
* [Data Landing Zone](../api/create/cloud-storage/data-landing-zone.md)
* [FTP](../api/create/cloud-storage/ftp.md)
* [Google Cloud Storage](../api/create/cloud-storage/google.md)
* [Oracle Object Storage](../api/create/cloud-storage/oracle-object-storage.md)
* [SFTP](../api/create/cloud-storage/sftp.md)

After creating a base connection, you must then follow the steps outlined in the tutorial for [creating a source connection for a cloud storage source](../api/collect/cloud-storage.md) in order to create a source connection, a target connection, and a mapping.

## Create a dataflow for encrypted data {#create-a-dataflow-for-encrypted-data}

>[!NOTE]
>
>You must have the following, in order to create a dataflow for encrypted data ingestion:
>
>* [Public key ID](#create-encryption-key-pair)
>* [Source connection ID](../api/collect/cloud-storage.md#source)
>* [Target connection ID](../api/collect/cloud-storage.md#target)
>* [Mapping ID](../api/collect/cloud-storage.md#mapping)

To create a dataflow, make a POST request to the `/flows` endpoint of the [!DNL Flow Service] API. To ingest encrypted data, you must add an `encryption` section to the `transformations` property and include the `publicKeyId` that was created in an earlier step.

**API format**

```http
POST /flows
```

>[!BEGINTABS]

>[!TAB Create a dataflow for encrypted data ingestion]

**Request**

+++View example request

The following request creates a dataflow to ingest encrypted data for a cloud storage source.

```shell
curl -X POST \
  'https://platform.adobe.io/data/foundation/flowservice/flows' \
  -H 'x-api-key: {{API_KEY}}' \
  -H 'x-gw-ims-org-id: {{ORG_ID}}' \
  -H 'x-sandbox-name: {{SANDBOX_NAME}}' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "ACME Customer Data",
    "description": "ACME Customer Data (Encrypted)",
    "flowSpec": {
        "id": "9753525b-82c7-4dce-8a9b-5ccfce2b9876",
        "version": "1.0"
    },
    "sourceConnectionIds": [
        "655f7c1b-1977-49b3-a429-51379ecf0e15"
    ],
    "targetConnectionIds": [
        "de688225-d619-481c-ae3b-40c250fd7c79"
    ],
    "transformations": [
        {
            "name": "Mapping",
            "params": {
                "mappingId": "6b6e24213dbe4f57bd8207d21034ff03",
                "mappingVersion":"0"
            }
        },
        {
            "name": "Encryption",
            "params": {
                "publicKeyId":"311ef6f8-9bcd-48cf-a9e9-d12c45fb7a17"
            }
        }
    ],
    "scheduleParams": {
        "startTime": "1675793392",
        "frequency": "once"
    }
}'
```

| Property | Description |
| --- | --- |
| `flowSpec.id` | The flow spec ID that corresponds with cloud storage sources. |
| `sourceConnectionIds` | The source connection ID. This ID represents the transfer of data from source to Experience Platform. |
| `targetConnectionIds` | The target connection ID. This ID represents where the data lands once it is brought over to Experience Platform. |
| `transformations[x].params.mappingId` | The mapping ID.|
| `transformations.name` | When ingesting encrypted files, you must provide `Encryption` as an additional transformations parameter for your dataflow. |
| `transformations[x].params.publicKeyId` | The public key ID that you created. This ID is one half of the encryption key pair used to encrypt your cloud storage data. |
| `scheduleParams.startTime` | The start time for the dataflow in epoch time. |
| `scheduleParams.frequency` | The frequency at which the dataflow will collect data. Acceptable values include: `once`, `minute`, `hour`, `day`, or `week`. |
| `scheduleParams.interval` | The interval designates the period between two consecutive flow runs. The interval's value should be a non-zero integer. Interval is not required when frequency is set as `once` and should be greater than or equal to `15` for other frequency values. |

+++

**Response**

+++View example response

A successful response returns the ID (`id`) of the newly created dataflow for your encrypted data.

```json
{
    "id": "dbc5c132-bc2a-4625-85c1-32bc2a262558",
    "etag": "\"8e000533-0000-0200-0000-5f3c40fd0000\""
}
```

+++

>[!TAB Create a dataflow to ingest encrypted and signed data]

**Request**

+++View example request

```shell
curl -X POST \
  'https://platform.adobe.io/data/foundation/flowservice/flows' \
  -H 'x-api-key: {{API_KEY}}' \
  -H 'x-gw-ims-org-id: {{ORG_ID}}' \
  -H 'x-sandbox-name: {{SANDBOX_NAME}}' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "ACME Customer Data (with Sign Verification)",
    "description": "ACME Customer Data (with Sign Verification)",
    "flowSpec": {
        "id": "9753525b-82c7-4dce-8a9b-5ccfce2b9876",
        "version": "1.0"
    },
    "sourceConnectionIds": [
        "655f7c1b-1977-49b3-a429-51379ecf0e15"
    ],
    "targetConnectionIds": [
        "de688225-d619-481c-ae3b-40c250fd7c79"
    ],
    "transformations": [
        {
            "name": "Mapping",
            "params": {
                "mappingId": "6b6e24213dbe4f57bd8207d21034ff03",
                "mappingVersion":"0"
            }
        },
        {
            "name": "Encryption",
            "params": {
                "publicKeyId":"311ef6f8-9bcd-48cf-a9e9-d12c45fb7a17",
                "signVerificationKeyId":"e31ae895-7896-469a-8e06-eb9207ddf1c2"
            }
        }
    ],
    "scheduleParams": {
        "startTime": "1675793392",
        "frequency": "once"
    }
}'
```

| Property | Description |
| --- | --- |
| `params.signVerificationKeyId` | The sign verification key ID is the same as the public key ID that was retrieved after sharing your Base64-encoded public key with Experience Platform. |

+++

**Response**

+++View example response

A successful response returns the ID (`id`) of the newly created dataflow for your encrypted data.

```json
{
    "id": "dbc5c132-bc2a-4625-85c1-32bc2a262558",
    "etag": "\"8e000533-0000-0200-0000-5f3c40fd0000\""
}
```

+++

>[!ENDTABS]

### Delete encryption keys {#delete-encryption-keys}

To delete your encryption keys, make a DELETE request to the `/encryption/keys` endpoint and provide your public key ID as a header parameter.

**API format**

```http
DELETE /data/foundation/connectors/encryption/keys/{PUBLIC_KEY_ID}
```

**Request**

+++View example request

```shell
curl -X DELETE \
  'https://platform.adobe.io/data/foundation/connectors/encryption/keys/{publicKeyId}' \
  -H 'Authorization: Bearer {{ACCESS_TOKEN}}' \
  -H 'x-api-key: {{API_KEY}}' \
  -H 'x-gw-ims-org-id: {{ORG_ID}}' \
```

+++

**Response**

A successful response returns HTTP status 204 (No Content) and a blank body.

### Validate encryption keys {#validate-encryption-keys}

To validate your encryption keys, make a GET request to the `/encryption/keys/validate/` endpoint and provide the public key ID that you want to validate as a header parameter.

```http
GET /data/foundation/connectors/encryption/keys/validate/{PUBLIC_KEY_ID}
```

**Request**

+++View example request

```shell
curl -X GET \
  'https://platform.adobe.io/data/foundation/connectors/encryption/keys/validate/{publicKeyId}' \
  -H 'Authorization: Bearer {{ACCESS_TOKEN}}' \
  -H 'x-api-key: {{API_KEY}}' \
  -H 'x-gw-ims-org-id: {{ORG_ID}}' \
```

+++

**Response**

A successful response returns either a confirmation that your IDs are valid, or invalid. 

>[!BEGINTABS]

>[!TAB Valid]

A valid public key ID returns a status of `Active` along with your public key ID.

```json
{
    "publicKeyId": "{PUBLIC_KEY_ID}",
    "status": "Active"
}
```

>[!TAB Invalid]

An invalid public key ID returns a status of `Expired` along with your public key ID.

```json
{
    "publicKeyId": "{PUBLIC_KEY_ID}",
    "status": "Expired"
}

```

>[!ENDTABS]


## Restrictions on recurring ingestion {#restrictions-on-recurring-ingestion}

Encrypted data ingestion does not support ingestion of recurring or multi-level folders in sources. All encrypted files must be contained in a single folder. Wildcards with multiple folders in a single source path are also not supported.

The following is an example of a supported folder structure, where the source path is `/ACME-customers/*.csv.gpg`.

In this scenario, the files in bold are ingested into Experience Platform.

* ACME-customers
  * **File1.csv.gpg**
  * File2.json.gpg
  * **File3.csv.gpg**
  * File4.json
  * **File5.csv.gpg**

The following is an example of an unsupported folder structure where the source path is `/ACME-customers/*`.

In this scenario, the flow run will fail and return an error message indicating that data cannot be copied from the source.

* ACME-customers
  * File1.csv.gpg
  * File2.json.gpg
  * Subfolder1
    * File3.csv.gpg
    * File4.json.gpg
    * File5.csv.gpg
* ACME-loyalty
  * File6.csv.gpg


## Next steps

By following this tutorial, you have created an encryption key pair for your cloud storage data, and a dataflow to ingested your encrypted data using the [!DNL Flow Service API]. For status updates on your dataflow's completeness, errors, and metrics, read the guide on [monitoring your dataflow using the [!DNL Flow Service] API](./monitor.md).