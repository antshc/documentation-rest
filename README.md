# Example REST API Documentation

This page demonstrates a complete Markdown-based REST API reference for `GET` and `POST` operations.

The fictional API manages projects.

---

# Endpoints

## Projects

### List projects

Returns a paginated list of projects visible to the authenticated user.

<details>
<summary>Request</summary>

**Request schema**

```http
GET /projects?[status=<status,...>]&[owner_id=<owner_id>]&[search=<search>]&[tag=<tag>]&[sort=<sort>]&[order=<order>]&[page=<page>]&[per_page=<per_page>]
```

**Query parameters**

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `status` | string | No | — | Filters by status. Accepts a comma-separated list of values; multiple values use **OR** logic: `active`, `archived`, `draft`. Example: `status=active,draft`. |
| `owner_id` | string | No | — | Filters by project owner. |
| `search` | string | No | — | Case-insensitive search in project names and descriptions. |
| `tag` | string | No | — | Filters projects containing the specified tag. |
| `sort` | string | No | `created_at` | Sort field: `name`, `created_at`, or `updated_at`. |
| `order` | string | No | `desc` | Sort order: `asc` or `desc`. |
| `page` | integer | No | `1` | Page number. Must be at least `1`. |
| `per_page` | integer | No | `20` | Results per page. Minimum `1`, maximum `100`. |

</details>

<details>
<summary>Response</summary>

**Response schema**

```json
{
  "type": "object",
  "properties": {
    "data": {
      "type": "array",
      "items": { "$ref": "#/Objects/Project" }
    },
    "pagination": {
      "type": "object",
      "properties": {
        "page":                { "type": "integer" },
        "per_page":            { "type": "integer" },
        "total_items":         { "type": "integer" },
        "total_pages":         { "type": "integer" },
        "has_next_page":       { "type": "boolean" },
        "has_previous_page":   { "type": "boolean" }
      },
      "required": ["page", "per_page", "total_items", "total_pages", "has_next_page", "has_previous_page"]
    }
  },
  "required": ["data", "pagination"]
}
```

**Response headers**

| Header | Description |
|---|---|
| `X-Waiting-Approval-Count` | Total number of waiting approval items matching the applied filters, regardless of pagination. |

**Response body**

| Field | Type | Description |
|---|---|---|
| `data` | array | Projects matching the request filters. |
| `pagination.page` | integer | Current page number. |
| `pagination.per_page` | integer | Maximum number of items requested per page. |
| `pagination.total_items` | integer | Total number of matching projects. |
| `pagination.total_pages` | integer | Total number of available pages. |
| `pagination.has_next_page` | boolean | Indicates whether another page is available. |
| `pagination.has_previous_page` | boolean | Indicates whether a previous page is available. |

**Response codes**

| Status | Description |
|---:|---|
| `200 OK` | Request completed successfully. |
| `400 Bad Request` | One or more query parameters are invalid. |


</details>

<details>
<summary>Example</summary>

**Request**

```http
GET /v1/projects?status=active,draft&sort=updated_at&order=desc&page=1&per_page=20 HTTP/1.1
Host: api.example.com
Authorization: Bearer <access-token>
Accept: application/json
```

**Response**

```http
HTTP/1.1 200 OK
Content-Type: application/json
X-Request-ID: req_01J2Y2C31M9H5S8Q7F4P6R0T2V
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 997
X-Waiting-Approval-Count: 3

{
  "data": [
    {
      "id": "prj_01J2XK7W8D0Q3Y6N9A4B5C7E8F",
      "name": "Disaster Recovery Portal",
      "description": "Customer portal for managing protected workloads.",
      "status": "active",
      "owner_id": "usr_01J2XJZ6E7M2K9C4R8T1W3P5Q6",
      "tags": [
        "cloud",
        "recovery"
      ],
      "created_at": "2026-07-10T14:20:00Z",
      "updated_at": "2026-07-11T08:15:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total_items": 2,
    "total_pages": 1,
    "has_next_page": false,
    "has_previous_page": false
  }
}
```

</details>

### Get a project

Returns one project by its identifier.

<details>
<summary>Request</summary>

**Request schema**

```http
GET /projects/{project_id}
```

**Path parameters**

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `project_id` | string | Yes | Unique project identifier. |

</details>

<details>
<summary>Response</summary>

**Response schema**

```json
{
  "type": "object",
  "properties": {
    "data": { "$ref": "#/Objects/Project" }
  },
  "required": ["data"]
}
```

**Response body**

| Field | Type | Description |
|---|---|---|
| `data` | object | The requested project. |

**Response codes**

| Status | Description |
|---:|---|
| `200 OK` | Request completed successfully. |
| `404 Not Found` | The project does not exist or is not visible to the user. |

</details>

<details>
<summary>Example</summary>

**Request**

```http
GET /v1/projects/prj_01J2XK7W8D0Q3Y6N9A4B5C7E8F HTTP/1.1
Host: api.example.com
Authorization: ******
Accept: application/json
```

**Response**

```http
HTTP/1.1 200 OK
Content-Type: application/json
X-Request-ID: req_01J2Y4D7K3V8C5N0Q2S9M6P1R4

{
  "data": {
    "id": "prj_01J2XK7W8D0Q3Y6N9A4B5C7E8F",
    "name": "Disaster Recovery Portal",
    "description": "Customer portal for managing protected workloads.",
    "status": "active",
    "owner_id": "usr_01J2XJZ6E7M2K9C4R8T1W3P5Q6",
    "tags": [
      "cloud",
      "recovery"
    ],
    "created_at": "2026-07-10T14:20:00Z",
    "updated_at": "2026-07-11T08:15:00Z"
  }
}
```

</details>

### Create a project

Creates a new project.

<details>
<summary>Request</summary>

**Request schema**

```http
POST /projects
```

**Request body**

| Field | Type | Required | Constraints | Description |
|---|---|---:|---|---|
| `name` | string | Yes | 1–120 characters | Human-readable project name. |
| `description` | string or null | No | Maximum 2000 characters | Optional project description. |
| `status` | string | No | `active` or `draft` | Initial project status. Defaults to `draft`. |
| `owner_id` | string | Yes | Existing user identifier | User who will own the project. |
| `tags` | array of strings | No | Maximum 20 unique values | Optional project tags. Defaults to an empty array. |
| `client_request_id` | string | No | Maximum 100 characters | Client-provided idempotency value. |

Unknown fields are rejected.

</details>

<details>
<summary>Response</summary>

**Response schema**

```json
{
  "type": "object",
  "properties": {
    "data": { "$ref": "#/Objects/Project" }
  },
  "required": ["data"]
}
```

**Response headers**

| Header | Description |
|---|---|
| `Location` | URL of the newly created project. |

**Response body**

| Field | Type | Description |
|---|---|---|
| `data` | object | The created project. |

**Response codes**

| Status | Description |
|---:|---|
| `201 Created` | Project created successfully. |
| `400 Bad Request` | The request body is malformed JSON. |
| `409 Conflict` | An idempotency key conflict occurred. |
| `422 Unprocessable Entity` | One or more fields fail validation. |

</details>

<details>
<summary>Example</summary>

**Request**

```http
POST /v1/projects HTTP/1.1
Host: api.example.com
Authorization: ******
Accept: application/json
Content-Type: application/json
Idempotency-Key: 61cdf307-681e-4c37-bdf7-f21dd3f0c584
Content-Length: 367

{
  "name": "Disaster Recovery Portal",
  "description": "Customer portal for managing protected workloads.",
  "status": "active",
  "owner_id": "usr_01J2XJZ6E7M2K9C4R8T1W3P5Q6",
  "tags": [
    "cloud",
    "recovery"
  ],
  "client_request_id": "portal-project-2026-07-11"
}
```

**Response**

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: https://api.example.com/v1/projects/prj_01J2XK7W8D0Q3Y6N9A4B5C7E8F
X-Request-ID: req_01J2Y6M3W8R4D7N1C9Q5V0K2P6

{
  "data": {
    "id": "prj_01J2XK7W8D0Q3Y6N9A4B5C7E8F",
    "name": "Disaster Recovery Portal",
    "description": "Customer portal for managing protected workloads.",
    "status": "active",
    "owner_id": "usr_01J2XJZ6E7M2K9C4R8T1W3P5Q6",
    "tags": [
      "cloud",
      "recovery"
    ],
    "created_at": "2026-07-11T09:30:00Z",
    "updated_at": "2026-07-11T09:30:00Z"
  }
}
```

</details>

---

# Objects

## Projects

### Project object

<details>
<summary>Fields</summary>

| Field | Type | Nullable | Description |
|---|---|---:|---|
| `id` | string | No | Unique project identifier. |
| `name` | string | No | Human-readable project name. |
| `description` | string | Yes | Optional project description. |
| `status` | string | No | Project status: `active`, `archived`, or `draft`. |
| `owner_id` | string | No | Identifier of the user who owns the project. |
| `tags` | array of strings | No | Project tags. Returns an empty array when no tags are assigned. |
| `created_at` | string | No | Creation timestamp in ISO 8601 format. |
| `updated_at` | string | No | Last update timestamp in ISO 8601 format. |

</details>

<details>
<summary>Example</summary>

```json
{
  "id": "prj_01J2XK7W8D0Q3Y6N9A4B5C7E8F",
  "name": "Disaster Recovery Portal",
  "description": "Customer portal for managing protected workloads.",
  "status": "active",
  "owner_id": "usr_01J2XJZ6E7M2K9C4R8T1W3P5Q6",
  "tags": [
    "cloud",
    "recovery"
  ],
  "created_at": "2026-07-10T14:20:00Z",
  "updated_at": "2026-07-11T08:15:00Z"
}
```

</details>

## Common

### Pagination object

Returned by paginated list endpoints inside the `pagination` field.

<details>
<summary>Fields</summary>

| Field | Type | Nullable | Description |
|---|---|---:|---|
| `page` | integer | No | Current page number. |
| `per_page` | integer | No | Maximum number of items requested per page. |
| `total_items` | integer | No | Total number of matching items. |
| `total_pages` | integer | No | Total number of available pages. |
| `has_next_page` | boolean | No | Indicates whether another page is available. |
| `has_previous_page` | boolean | No | Indicates whether a previous page is available. |

</details>

<details>
<summary>Example</summary>

```json
{
  "page": 1,
  "per_page": 20,
  "total_items": 2,
  "total_pages": 1,
  "has_next_page": false,
  "has_previous_page": false
}
```

</details>

### Error object

Returned by all error responses.

<details>
<summary>Fields</summary>

| Field | Type | Nullable | Description |
|---|---|---:|---|
| `error.code` | string | No | Stable, machine-readable error code. |
| `error.message` | string | No | Human-readable summary. |
| `error.details` | array | No | Field-level or contextual error details. |
| `error.details[].field` | string | Yes | Related request field, when applicable. |
| `error.details[].message` | string | No | Explanation of the specific problem. |
| `error.details[].value` | any | Yes | Rejected value, when safe to return. |
| `error.request_id` | string | No | Identifier used for troubleshooting and support. |

</details>

<details>
<summary>Example</summary>

```json
{
  "error": {
    "code": "machine_readable_code",
    "message": "Human-readable error description.",
    "details": [
      {
        "field": "field_name",
        "message": "Field-specific explanation.",
        "value": "invalid value"
      }
    ],
    "request_id": "req_01J2YB3M7V1C9N4Q6D8R0K2P5S"
  }
}
```

</details>

---

# Conventions

## Base URL

<details>
<summary>Base URL</summary>

```text
https://api.example.com/v1
```

</details>

## Authentication

<details>
<summary>Authentication</summary>

All requests require a bearer access token. Requests without a valid token return `401 Unauthorized`.

</details>

## Common request headers

<details>
<summary>Common request headers</summary>

| Header | Required | Description |
|---|---:|---|
| `Authorization` | Yes | Bearer access token. |
| `Accept` | Yes | Should be `application/json`. |
| `Content-Type` | For requests with a body | Must be `application/json`. |
| `Idempotency-Key` | No | Client-provided idempotency value. See [Idempotency](#idempotency). |

</details>

## Common response headers

<details>
<summary>Common response headers</summary>

| Header | Description |
|---|---|
| `Content-Type` | Response media type, normally `application/json`. |
| `X-Request-ID` | Unique identifier for tracing the request. |
| `X-RateLimit-Limit` | Maximum number of requests allowed in the current window. |
| `X-RateLimit-Remaining` | Requests remaining in the current window. |

</details>

## Common formats

<details>
<summary>Common formats</summary>

### Timestamps

All timestamps use ISO 8601 in UTC.

```text
2026-07-11T09:30:00Z
```

### Identifiers

Resource identifiers are prefixed strings combining a resource-type prefix and a ULID.

```text
prj_01J2XK7W8D0Q3Y6N9A4B5C7E8F
usr_01J2XJZ6E7M2K9C4R8T1W3P5Q6
req_01J2Y2C31M9H5S8Q7F4P6R0T2V
```

### Enumerations

Enumeration values are lowercase strings. The API rejects values not in the documented set with `400 Bad Request` or `422 Unprocessable Entity`.

### Nullable fields

A field documented as nullable may be omitted from the response or present with a `null` value. Clients must handle both cases.

</details>

## Common status codes

<details>
<summary>Common status codes</summary>

| Status | Meaning |
|---:|---|
| `200 OK` | A `GET` request completed successfully. |
| `201 Created` | A resource was created successfully. |
| `400 Bad Request` | The request syntax, JSON, or query parameters are invalid. |
| `401 Unauthorized` | Authentication is missing or invalid. |
| `403 Forbidden` | The authenticated user cannot perform the operation. |
| `404 Not Found` | The requested resource does not exist or is not visible to the user. |
| `409 Conflict` | The request conflicts with the current resource state. |
| `415 Unsupported Media Type` | The request does not use `application/json`. |
| `422 Unprocessable Entity` | The request body is syntactically valid but fails validation. |
| `429 Too Many Requests` | The client exceeded the rate limit. |
| `500 Internal Server Error` | An unexpected server error occurred. |
| `503 Service Unavailable` | The service is temporarily unavailable. |

</details>

## Error format

<details>
<summary>Error format</summary>

All error responses use the same structure.

```json
{
  "error": {
    "code": "machine_readable_code",
    "message": "Human-readable error description.",
    "details": [
      {
        "field": "field_name",
        "message": "Field-specific explanation.",
        "value": "invalid value"
      }
    ],
    "request_id": "req_01J2YB3M7V1C9N4Q6D8R0K2P5S"
  }
}
```

### Error fields

| Field | Type | Description |
|---|---|---|
| `error.code` | string | Stable, machine-readable error code. |
| `error.message` | string | Human-readable summary. |
| `error.details` | array | Field-level or contextual error details. |
| `error.details[].field` | string | Related request field, when applicable. |
| `error.details[].message` | string | Explanation of the specific problem. |
| `error.details[].value` | any | Rejected value, when safe to return. |
| `error.request_id` | string | Identifier used for troubleshooting and support. |


### Unauthorized response

<details>
<summary>401 Unauthorized</summary>

```http
HTTP/1.1 401 Unauthorized
Content-Type: application/json
WWW-Authenticate: Bearer
```

```json
{
  "error": {
    "code": "unauthorized",
    "message": "A valid bearer token is required.",
    "details": [],
    "request_id": "req_01J2YC4N8V2D6M0Q3R7K9P1S5T"
  }
}
```

</details>

### Forbidden response

<details>
<summary>403 Forbidden</summary>

```http
HTTP/1.1 403 Forbidden
Content-Type: application/json
```

```json
{
  "error": {
    "code": "forbidden",
    "message": "You do not have permission to perform this operation.",
    "details": [],
    "request_id": "req_01J2YD5M9V3C7N1Q4R8K0P2S6T"
  }
}
```

</details>

### Rate-limit response

<details>
<summary>429 Too Many Requests</summary>

```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
Retry-After: 60
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
```

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "The request rate limit has been exceeded.",
    "details": [],
    "request_id": "req_01J2YE6N0V4C8M2Q5R9K1P3S7T"
  }
}
```

</details>

### Malformed JSON

<details>
<summary>400 Bad Request — malformed JSON</summary>

Returns `400 Bad Request` when the request body is not valid JSON.

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json
X-Request-ID: req_01J2Y8R4V7N0C3M6K9Q1D5P2S8

{
  "error": {
    "code": "invalid_json",
    "message": "The request body contains malformed JSON.",
    "details": [],
    "request_id": "req_01J2Y8R4V7N0C3M6K9Q1D5P2S8"
  }
}
```

</details>

</details>

## Idempotency

<details>
<summary>Idempotency</summary>

Clients may include an `Idempotency-Key` header when creating a project.

```http
Idempotency-Key: 61cdf307-681e-4c37-bdf7-f21dd3f0c584
```

The server stores the result for 24 hours.

- Repeating the same request with the same key returns the original response.
- Reusing the key with a different request body returns `409 Conflict`.
- The key must be unique per authenticated client.

</details>

---

