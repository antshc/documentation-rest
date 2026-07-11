# Example REST API Documentation

This page demonstrates a complete Markdown-based REST API reference for `GET` and `POST` operations.

The fictional API manages projects.

---

# Conventions

## Base URL

```text
https://api.example.com/v1
```

## Authentication

All requests require a bearer access token. Requests without a valid token return `401 Unauthorized`.

## Common request headers

| Header | Required | Description |
|---|---:|---|
| `Authorization` | Yes | Bearer access token. |
| `Accept` | Yes | Should be `application/json`. |
| `Content-Type` | For requests with a body | Must be `application/json`. |
| `Idempotency-Key` | No | Client-provided idempotency value. See [Idempotency](#idempotency). |

## Common response headers

| Header | Description |
|---|---|
| `Content-Type` | Response media type, normally `application/json`. |
| `X-Request-ID` | Unique identifier for tracing the request. |
| `X-RateLimit-Limit` | Maximum number of requests allowed in the current window. |
| `X-RateLimit-Remaining` | Requests remaining in the current window. |

## Common formats

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

---

# Objects

## Project object

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

Example project:

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

---

# List projects

Returns a paginated list of projects visible to the authenticated user.

```http
GET /projects?[status=<status,...>]&[owner_id=<owner_id>]&[search=<search>]&[tag=<tag>]&[sort=<sort>]&[order=<order>]&[page=<page>]&[per_page=<per_page>]
```

## Query parameters

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

## Example request


```http
GET /v1/projects?status=active,draft&sort=updated_at&order=desc&page=1&per_page=20 HTTP/1.1
Host: api.example.com
Authorization: Bearer <access-token>
Accept: application/json
```

## Successful response

Returns `200 OK`.

```http
HTTP/1.1 200 OK
Content-Type: application/json
X-Request-ID: req_01J2Y2C31M9H5S8Q7F4P6R0T2V
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 997
```

```json
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
    },
    {
      "id": "prj_01J2XM4N6C8Q2T5V7W9Y1Z3A4B",
      "name": "Monitoring Service",
      "description": null,
      "status": "active",
      "owner_id": "usr_01J2XJZ6E7M2K9C4R8T1W3P5Q6",
      "tags": [
        "observability"
      ],
      "created_at": "2026-07-08T10:00:00Z",
      "updated_at": "2026-07-10T16:40:00Z"
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

## Response fields

| Field | Type | Description |
|---|---|---|
| `data` | array | Projects matching the request filters. |
| `pagination.page` | integer | Current page number. |
| `pagination.per_page` | integer | Maximum number of items requested per page. |
| `pagination.total_items` | integer | Total number of matching projects. |
| `pagination.total_pages` | integer | Total number of available pages. |
| `pagination.has_next_page` | boolean | Indicates whether another page is available. |
| `pagination.has_previous_page` | boolean | Indicates whether a previous page is available. |

## Empty response

An empty result still returns `200 OK`.

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total_items": 0,
    "total_pages": 0,
    "has_next_page": false,
    "has_previous_page": false
  }
}
```

## Invalid query parameter

Returns `400 Bad Request` when a query parameter is invalid.

Example request:

```http
GET /v1/projects?status=deleted HTTP/1.1
Host: api.example.com
Authorization: Bearer <access-token>
Accept: application/json
```

Response:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json
X-Request-ID: req_01J2Y3R7C1M8V5N0K4D9S2P6Q3
```

```json
{
  "error": {
    "code": "invalid_query_parameter",
    "message": "One or more query parameters are invalid.",
    "details": [
      {
        "field": "status",
        "message": "Must be one of: active, archived, draft.",
        "value": "deleted"
      }
    ],
    "request_id": "req_01J2Y3R7C1M8V5N0K4D9S2P6Q3"
  }
}
```

---

# Get a project

Returns one project by its identifier.

```http
GET /projects/{project_id}
```

## Path parameters

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `project_id` | string | Yes | Unique project identifier. |

## Example request


```http
GET /v1/projects/prj_01J2XK7W8D0Q3Y6N9A4B5C7E8F HTTP/1.1
Host: api.example.com
Authorization: Bearer <access-token>
Accept: application/json
```

## Successful response

Returns `200 OK`.

```http
HTTP/1.1 200 OK
Content-Type: application/json
X-Request-ID: req_01J2Y4D7K3V8C5N0Q2S9M6P1R4
```

```json
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

## Project not found

Returns `404 Not Found`.

```http
HTTP/1.1 404 Not Found
Content-Type: application/json
X-Request-ID: req_01J2Y5F8N4M1C7R9Q3V6W0K2T5
```

```json
{
  "error": {
    "code": "project_not_found",
    "message": "The requested project was not found.",
    "details": [],
    "request_id": "req_01J2Y5F8N4M1C7R9Q3V6W0K2T5"
  }
}
```

---

# Create a project

Creates a new project.

```http
POST /projects
```

## Request body

| Field | Type | Required | Constraints | Description |
|---|---|---:|---|---|
| `name` | string | Yes | 1–120 characters | Human-readable project name. |
| `description` | string or null | No | Maximum 2000 characters | Optional project description. |
| `status` | string | No | `active` or `draft` | Initial project status. Defaults to `draft`. |
| `owner_id` | string | Yes | Existing user identifier | User who will own the project. |
| `tags` | array of strings | No | Maximum 20 unique values | Optional project tags. Defaults to an empty array. |
| `client_request_id` | string | No | Maximum 100 characters | Client-provided idempotency value. |

Unknown fields are rejected.


## Example request


```http
POST /v1/projects HTTP/1.1
Host: api.example.com
Authorization: Bearer <access-token>
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

## Successful response

Returns `201 Created`.

The `Location` header contains the URL of the new resource.

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: https://api.example.com/v1/projects/prj_01J2XK7W8D0Q3Y6N9A4B5C7E8F
X-Request-ID: req_01J2Y6M3W8R4D7N1C9Q5V0K2P6
```

```json
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

## Validation error

Returns `422 Unprocessable Entity` when the JSON is valid but one or more fields fail validation.

Example request:

```json
{
  "name": "",
  "status": "completed",
  "owner_id": "unknown-user",
  "tags": [
    "cloud",
    "cloud"
  ]
}
```

Response:

```http
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json
X-Request-ID: req_01J2Y7P5N9V1C4D8K2Q6R0M3S7
```

```json
{
  "error": {
    "code": "validation_failed",
    "message": "The request body contains invalid values.",
    "details": [
      {
        "field": "name",
        "message": "Must contain at least 1 character.",
        "value": ""
      },
      {
        "field": "status",
        "message": "Must be one of: active, draft.",
        "value": "completed"
      },
      {
        "field": "owner_id",
        "message": "The specified user does not exist.",
        "value": "unknown-user"
      },
      {
        "field": "tags",
        "message": "Tags must be unique.",
        "value": [
          "cloud",
          "cloud"
        ]
      }
    ],
    "request_id": "req_01J2Y7P5N9V1C4D8K2Q6R0M3S7"
  }
}
```

## Malformed JSON

Returns `400 Bad Request` when the request body is not valid JSON.

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json
X-Request-ID: req_01J2Y8R4V7N0C3M6K9Q1D5P2S8
```

```json
{
  "error": {
    "code": "invalid_json",
    "message": "The request body contains malformed JSON.",
    "details": [],
    "request_id": "req_01J2Y8R4V7N0C3M6K9Q1D5P2S8"
  }
}
```

## Duplicate project

Returns `409 Conflict` when an active project with the same name already exists for the same owner.

```http
HTTP/1.1 409 Conflict
Content-Type: application/json
X-Request-ID: req_01J2Y9T6M3V8C1N5Q7D4R0K2P9
```

```json
{
  "error": {
    "code": "project_already_exists",
    "message": "An active project with this name already exists for the specified owner.",
    "details": [
      {
        "field": "name",
        "message": "Project names must be unique per owner among active projects.",
        "value": "Disaster Recovery Portal"
      }
    ],
    "request_id": "req_01J2Y9T6M3V8C1N5Q7D4R0K2P9"
  }
}
```

## Idempotency

Clients may include an `Idempotency-Key` header when creating a project.

```http
Idempotency-Key: 61cdf307-681e-4c37-bdf7-f21dd3f0c584
```

The server stores the result for 24 hours.

- Repeating the same request with the same key returns the original response.
- Reusing the key with a different request body returns `409 Conflict`.
- The key must be unique per authenticated client.

Example conflict:

```json
{
  "error": {
    "code": "idempotency_key_reused",
    "message": "The idempotency key was already used with a different request.",
    "details": [],
    "request_id": "req_01J2YAQ8D4N1C7M5V9R2K6P0S3"
  }
}
```

---

# Error format

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

## Error fields

| Field | Type | Description |
|---|---|---|
| `error.code` | string | Stable, machine-readable error code. |
| `error.message` | string | Human-readable summary. |
| `error.details` | array | Field-level or contextual error details. |
| `error.details[].field` | string | Related request field, when applicable. |
| `error.details[].message` | string | Explanation of the specific problem. |
| `error.details[].value` | any | Rejected value, when safe to return. |
| `error.request_id` | string | Identifier used for troubleshooting and support. |

## Common status codes

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

## Unauthorized response

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

## Forbidden response

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

## Rate-limit response

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

---

# Changelog

## 2026-07-11

- Added the `GET /projects` operation.
- Added the `GET /projects/{project_id}` operation.
- Added the `POST /projects` operation.
- Added pagination, validation, error, and idempotency examples.
