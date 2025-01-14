# Indexes API

## 1. Summary

This specification describes the indexes API endpoints. The endpoint gives the possibility to get, get all, create, update and delete Meilsearch indexes.

## 2. Motivation
N/A

## 3. Functional Specification

Indexes contain a set of documents in which to search and have their specific settings.

See [Documents API specification](0124-documents-api.md) and [Settings API specification](0123-settings-api.md) for more details.


### 3.1. `index` API Resource Definition

| Field                         | Type            | Required |
|-------------------------------|-----------------|----------|
| [uid](#311-uid)               | string          | True     |
| [name](#312-name)             | string          | False    |
| [primaryKey](#313-primaryKey) | string / `null` | False    |
| [createdAt](#314-createdAt)   | string          | False    |
| [updatedAt](#315-updatedAt)   | string          | False    |

#### 3.1.1. `uid`

- Type: string
- Required: true

A unique identifier for the index.

This field is mandatory when creating an index and cannot be changed afterwards.

The field `uid` can be an integer or a string containing only alphanumeric characters, hyphens (-) and underscores (_).

#### 3.1.2. `primaryKey`

- Type: string
- Required: false
- Default: `null`

The primary key is the attribute in a document whose value is unique amongst all the other documents.

This field allows bypassing the auto-inference mechanism of the document identifiers.

By default, the `primaryKey` will be chosen by the auto-inference mechanism by the engine when a first document is indexed.

Specifying this field tells the engine to use the document attribute specified in `primaryKey` and bypasses this mechanism.

When the index is empty, it is possible to modify the `primaryKey`.

#### 3.1.3. `createdAt`

- Type: string
- Required: false

The creation date on which the index has been created.

Automatically generated by the engine at the creation of an index.

Represented with the `RFC 3339` format.

#### 3.1.4. `updatedAt`

- Type: string
- Required: false

The latest date on which the index has been updated.

Automatically generated by the engine at the creation/update of an index.

Represented wih the `RFC 3339` format.

### 3.2. API Endpoints Definition

Manipulate indexes of a Meilisearch instance.

- [3.2.1. `GET` - `indexes`](#321-get---indexes)
- [3.2.2. `GET` - `indexes/:index_uid`](#322-get---indexesindexuid)
- [3.2.3. `POST` - `indexes`](#323-post---indexes)
- [3.2.4. `PATCH` - `indexes/:index_uid`](#324-patch---indexesindexuid)
- [3.2.5. `DELETE` - `indexes/:index_uid`](#325-delete---indexesindexuid)

#### 3.2.1. `GET` - `indexes`

List all indexes of a Meilisearch instance.

The results are sorted in ascending alphanumeric order from the `uid` field.

##### 3.2.1.1. Query Parameters

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `offset`                 | integer / `null`         | false    |
| `limit`                  | integer / `null`         | false    |

###### 3.2.1.1.1. `offset`

- Type: Integer
- Required: False
- Default: `0`

Sets the starting point in the results, effectively skipping over a given number of indexes.

###### 3.2.1.1.2. `limit`

- Type: Integer
- Required: False
- Default: `20`

Sets the maximum number of indexes to be returned by the current request.

##### 3.2.1.2. Response Definition

An object containing all the indexes.

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `results`                | Array[Index]             | true     |
| `offset`                 | integer                  | true     |
| `limit`                  | integer                  | true     |
| `total`                  | integer                  | true     |

###### 3.2.1.2.1. `results`

- Type: Array[Index]
- Required: True

An array containing the fetched indexes.

###### 3.2.1.2.2. `offset`

- Type: Integer
- Required: True

Gives the `offset` parameter used for the query.

> See [3.2.1.1.1. `offset`](#32111-offset) section.

###### 3.2.1.2.3. `limit`

- Type: Integer
- Required: True

Gives the `limit` parameter used for the query.

> See [3.2.1.1.2. `limit`](#32112-limit) section.

###### 3.2.1.2.3. `total`

- Type: Integer
- Required: True

Gives the total number of indexes that can be browsed.

#### 3.2.2. `GET` - `indexes/:index_uid`

Fetch an index of a Meilisearch instance.

##### 3.2.2.1. Response Definition

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `uid`                    | string                   | true     |
| `primaryKey`             | string / `null`          | true     |
| `createdAt`              | string                   | true     |
| `updatedAt`              | string                   | true     |

##### 3.2.2.2. Errors

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.2.3. `POST` - `indexes`

Creates an index.

##### 3.2.3.1. Request Payload Definition

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `uid`                    | string                   | true     |
| `primaryKey`             | string / `null`          | false    |

##### 3.2.3.2. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.2.3.3. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending a value with a different type than `string` for `uid` will return a [bad_request](0061-error-format-and-definitions.md#bad_request) error.
- 🔴 Sending an invalid `uid` returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 Sending a value with a different type than `string` or `null` for `primaryKey` will return a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

###### 3.2.3.3.1. Async Errors

- 🔴 When Meilisearch is secured by a master key, if the API Key used do not have the `indexes.create` action defined, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related asynchronous `task` resource. See [3.2.2.2. Response Definition](#3222-response-definition).
- 🔴 Sending a `uid` that already exists returns an [index_already_exists](0061-error-format-and-definitions.md#index_already_exists) error.

#### 3.2.4. `PATCH` - `indexes/:index_uid`

Updates an index.

The `primaryKey` field can be updated when the index is empty. If the `primaryKey` is not defined, the indexing process will try to auto-infer the `primaryKey` by searching the first attribute containing `id` in the first document payload to index.

##### 3.2.4.1. Request Payload Definition

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `primaryKey`             | string / `null`          | False    |

##### 3.2.4.2. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.2.4.3. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.

###### 3.2.4.3.1. Async Errors

- 🔴 When updating the `primaryKey`, if the previous `primaryKey` value has already been used for a document, the API returns an [index_primary_key_already_exists](0061-error-format-and-definitions.md#index_primary_key_already_exists) error.
- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.2.5. `DELETE` - `indexes/:index_uid`

Deletes an index.

##### 3.2.4.1. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.2.4.2. Errors

###### 3.2.4.2.1 Async Errors

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.2.6. General Errors

These errors apply to all endpoints described here.

##### 3.2.6.1 Auth Errors

The auth layer can return the following errors if Meilisearch is secured (a master-key is defined).

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have the required permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

## 4. Technical Details
N/A

## 5. Future Possibilities

- Rework the `primaryKey` concept
