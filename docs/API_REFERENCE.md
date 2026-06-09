# API Reference

API reference for **PayFlow Ledger**, a multi-provider payment webhook and reconciliation API.

This document describes the planned API routes, request formats, response shapes, filtering options, and future reconciliation endpoints.

> Current status: **Planning / Repository Setup**
> Current version: **v0.1.0**

The backend has not been implemented yet. This file currently documents the planned API shape for the MVP.

---

## Base URL

Planned local development base URL:

```text
http://localhost:3000
```

Planned API version base path:

```text
/api/v1
```

Unless otherwise noted, API endpoint paths are shown relative to the base API version.

Example:

```text
/api/v1/events
```

---

## Response Format

API responses will return JSON.

Timestamps should use ISO 8601 date strings.

Example timestamp:

```json
"2026-06-08T00:00:00.000Z"
```

---

## Standard Error Response

Validation and HTTP errors should use a consistent JSON response shape.

Planned error response format:

```json
{
  "statusCode": 400,
  "timestamp": "2026-06-08T00:00:00.000Z",
  "path": "/api/v1/events",
  "method": "POST",
  "message": "Validation failed",
  "error": "Bad Request"
}
```

**Error fields:**

| Field        | Type               | Description                                    |
| ------------ | ------------------ | ---------------------------------------------- |
| `statusCode` | number             | HTTP status code returned by the API           |
| `timestamp`  | string             | ISO 8601 timestamp for when the error occurred |
| `path`       | string             | Request path that caused the error             |
| `method`     | string             | HTTP method used for the request               |
| `message`    | string or string[] | Error message or validation messages           |
| `error`      | string             | Short error name                               |

---

## Health Check

### `GET /`

Returns a basic application health message.

This route is planned as a simple root-level health check to confirm the backend is running.

**Example response:**

```json
{
  "message": "PayFlow Ledger API is running"
}
```

**Status:** Planned

---

## Payment Events

Payment events represent webhook-style records received from simulated payment providers.

Initial simulated providers:

* Stripe
* PayPal
* Square

Example event types:

* `payment.created`
* `payment.succeeded`
* `payment.failed`
* `payment.refunded`
* `refund.created`
* `refund.succeeded`
* `dispute.created`
* `dispute.resolved`
* `invoice.created`
* `invoice.paid`
* `subscription.created`
* `subscription.cancelled`

---

## Create Payment Event

### `POST /events`

Creates a new payment event record.

Full planned route:

```text
POST /api/v1/events
```

**Request body:**

```json
{
  "provider": "stripe",
  "eventType": "payment.succeeded",
  "externalEventId": "evt_001",
  "amount": 2500,
  "currency": "USD",
  "status": "succeeded",
  "payload": {
    "paymentId": "pay_001",
    "customerId": "cus_001",
    "description": "Simulated payment event"
  }
}
```

**Request fields:**

| Field             | Type   | Required | Description                                  |
| ----------------- | ------ | -------- | -------------------------------------------- |
| `provider`        | string | Yes      | Simulated payment provider name              |
| `eventType`       | string | Yes      | Provider event type                          |
| `externalEventId` | string | Yes      | External provider event ID                   |
| `amount`          | number | No       | Payment amount in the smallest currency unit |
| `currency`        | string | No       | Three-letter currency code                   |
| `status`          | string | No       | Payment or event status                      |
| `payload`         | object | Yes      | Original provider event payload              |

**Example response:**

```json
{
  "id": "event_001",
  "provider": "stripe",
  "eventType": "payment.succeeded",
  "externalEventId": "evt_001",
  "amount": 2500,
  "currency": "USD",
  "status": "succeeded",
  "payload": {
    "paymentId": "pay_001",
    "customerId": "cus_001",
    "description": "Simulated payment event"
  },
  "processed": false,
  "receivedAt": "2026-06-08T00:00:00.000Z",
  "processedAt": null,
  "createdAt": "2026-06-08T00:00:00.000Z",
  "updatedAt": "2026-06-08T00:00:00.000Z"
}
```

**Status:** Planned

---

## List Payment Events

### `GET /events`

Returns a list of payment events.

Full planned route:

```text
GET /api/v1/events
```

**Query parameters:**

| Parameter   | Type    | Required | Description                              |
| ----------- | ------- | -------- | ---------------------------------------- |
| `provider`  | string  | No       | Filter events by provider                |
| `eventType` | string  | No       | Filter events by event type              |
| `processed` | boolean | No       | Filter events by processed status        |
| `status`    | string  | No       | Filter events by payment or event status |

**Example request:**

```text
GET /api/v1/events?provider=stripe&processed=false
```

**Example response:**

```json
[
  {
    "id": "event_001",
    "provider": "stripe",
    "eventType": "payment.succeeded",
    "externalEventId": "evt_001",
    "amount": 2500,
    "currency": "USD",
    "status": "succeeded",
    "processed": false,
    "receivedAt": "2026-06-08T00:00:00.000Z",
    "processedAt": null,
    "createdAt": "2026-06-08T00:00:00.000Z",
    "updatedAt": "2026-06-08T00:00:00.000Z"
  }
]
```

**Status:** Planned

---

## Get Payment Event by ID

### `GET /events/:id`

Returns one payment event by ID.

Full planned route:

```text
GET /api/v1/events/:id
```

**Path parameters:**

| Parameter | Type   | Description      |
| --------- | ------ | ---------------- |
| `id`      | string | Payment event ID |

**Example request:**

```text
GET /api/v1/events/event_001
```

**Example response:**

```json
{
  "id": "event_001",
  "provider": "stripe",
  "eventType": "payment.succeeded",
  "externalEventId": "evt_001",
  "amount": 2500,
  "currency": "USD",
  "status": "succeeded",
  "payload": {
    "paymentId": "pay_001",
    "customerId": "cus_001",
    "description": "Simulated payment event"
  },
  "processed": false,
  "receivedAt": "2026-06-08T00:00:00.000Z",
  "processedAt": null,
  "createdAt": "2026-06-08T00:00:00.000Z",
  "updatedAt": "2026-06-08T00:00:00.000Z"
}
```

**Not found example:**

```json
{
  "statusCode": 404,
  "timestamp": "2026-06-08T00:00:00.000Z",
  "path": "/api/v1/events/event_missing",
  "method": "GET",
  "message": "Payment event not found",
  "error": "Not Found"
}
```

**Status:** Planned

---

## Mark Payment Event as Processed

### `PATCH /events/:id/processed`

Marks a payment event as processed.

Full planned route:

```text
PATCH /api/v1/events/:id/processed
```

**Path parameters:**

| Parameter | Type   | Description      |
| --------- | ------ | ---------------- |
| `id`      | string | Payment event ID |

**Example request:**

```text
PATCH /api/v1/events/event_001/processed
```

**Example response:**

```json
{
  "id": "event_001",
  "provider": "stripe",
  "eventType": "payment.succeeded",
  "externalEventId": "evt_001",
  "amount": 2500,
  "currency": "USD",
  "status": "succeeded",
  "processed": true,
  "receivedAt": "2026-06-08T00:00:00.000Z",
  "processedAt": "2026-06-08T00:05:00.000Z",
  "createdAt": "2026-06-08T00:00:00.000Z",
  "updatedAt": "2026-06-08T00:05:00.000Z"
}
```

**Status:** Planned

---

## Event Summary

### `GET /events/summary`

Returns summary totals for stored payment events.

Full planned route:

```text
GET /api/v1/events/summary
```

**Example response:**

```json
{
  "totalEvents": 5,
  "processedEvents": 3,
  "unprocessedEvents": 2,
  "eventsByProvider": {
    "stripe": 3,
    "paypal": 1,
    "square": 1
  },
  "eventsByType": {
    "payment.succeeded": 2,
    "payment.failed": 1,
    "refund.created": 1,
    "dispute.created": 1
  },
  "eventsByStatus": {
    "succeeded": 2,
    "failed": 1,
    "refunded": 1,
    "disputed": 1
  }
}
```

**Response fields:**

| Field               | Type   | Description                          |
| ------------------- | ------ | ------------------------------------ |
| `totalEvents`       | number | Total number of stored events        |
| `processedEvents`   | number | Number of events marked as processed |
| `unprocessedEvents` | number | Number of events not yet processed   |
| `eventsByProvider`  | object | Event totals grouped by provider     |
| `eventsByType`      | object | Event totals grouped by event type   |
| `eventsByStatus`    | object | Event totals grouped by status       |

**Status:** Planned

---

## Reconciliation

Reconciliation endpoints are planned for a later project phase.

The goal of reconciliation is to detect payment event issues such as duplicates, failed payments, unprocessed events, and mismatched records.

---

## View Reconciliation Results

### `GET /reconciliation`

Returns the latest reconciliation summary.

Full planned route:

```text
GET /api/v1/reconciliation
```

**Example response:**

```json
{
  "totalEventsChecked": 5,
  "duplicateExternalEventIds": 1,
  "unprocessedEvents": 2,
  "failedPayments": 1,
  "amountMismatches": 0,
  "currencyMismatches": 0,
  "issues": [
    {
      "type": "duplicate_external_event_id",
      "message": "Duplicate external event ID detected",
      "externalEventId": "evt_001",
      "count": 2
    },
    {
      "type": "unprocessed_event",
      "message": "Payment event has not been processed",
      "eventId": "event_002"
    }
  ]
}
```

**Status:** Future planned

---

## Run Reconciliation Check

### `POST /reconciliation/run`

Runs reconciliation checks against stored payment events.

Full planned route:

```text
POST /api/v1/reconciliation/run
```

**Example response:**

```json
{
  "message": "Reconciliation check completed",
  "result": {
    "totalEventsChecked": 5,
    "duplicateExternalEventIds": 1,
    "unprocessedEvents": 2,
    "failedPayments": 1,
    "amountMismatches": 0,
    "currencyMismatches": 0
  }
}
```

**Status:** Future planned

---

## Providers

Provider endpoints are planned for a later project phase.

---

## List Providers

### `GET /providers`

Returns supported simulated payment providers.

Full planned route:

```text
GET /api/v1/providers
```

**Example response:**

```json
[
  {
    "name": "stripe",
    "displayName": "Stripe",
    "enabled": true
  },
  {
    "name": "paypal",
    "displayName": "PayPal",
    "enabled": true
  },
  {
    "name": "square",
    "displayName": "Square",
    "enabled": true
  }
]
```

**Status:** Future planned

---

## Metrics

Metrics endpoints are planned for a later project phase.

---

## Payment Metrics

### `GET /metrics`

Returns payment event metrics.

Full planned route:

```text
GET /api/v1/metrics
```

**Example response:**

```json
{
  "totalVolume": 12500,
  "currency": "USD",
  "successfulPaymentCount": 4,
  "failedPaymentCount": 1,
  "refundCount": 1,
  "disputeCount": 1,
  "providerBreakdown": {
    "stripe": {
      "eventCount": 3,
      "totalVolume": 7500
    },
    "paypal": {
      "eventCount": 1,
      "totalVolume": 2500
    },
    "square": {
      "eventCount": 1,
      "totalVolume": 2500
    }
  }
}
```

**Status:** Future planned

---

## Planned Data Model

The first major database model will likely be a payment event.

**Payment event fields:**

| Field             | Type             | Description                              |
| ----------------- | ---------------- | ---------------------------------------- |
| `id`              | string           | Internal event ID                        |
| `provider`        | string           | Simulated payment provider               |
| `eventType`       | string           | Provider event type                      |
| `externalEventId` | string           | Event ID from simulated provider         |
| `amount`          | number           | Payment amount in smallest currency unit |
| `currency`        | string           | Three-letter currency code               |
| `status`          | string           | Payment or event status                  |
| `payload`         | object or string | Original provider event payload          |
| `processed`       | boolean          | Whether the event has been processed     |
| `receivedAt`      | string           | Timestamp when event was received        |
| `processedAt`     | string or null   | Timestamp when event was processed       |
| `createdAt`       | string           | Database creation timestamp              |
| `updatedAt`       | string           | Database update timestamp                |

---

## Planned Validation Rules

Initial validation may include:

* `provider` is required
* `eventType` is required
* `externalEventId` is required
* `payload` is required
* `amount` must be a number when provided
* `currency` should be a valid three-letter code when provided
* `processed` should default to `false`
* Duplicate external event IDs should eventually be detected

---

## Planned HTTP Status Codes

| Status Code | Meaning                             |
| ----------- | ----------------------------------- |
| `200`       | Request succeeded                   |
| `201`       | Resource created                    |
| `400`       | Invalid request or validation error |
| `404`       | Resource not found                  |
| `409`       | Duplicate event conflict            |
| `500`       | Unexpected server error             |

---

## Current Implementation Status

| Endpoint                             | Status         |
| ------------------------------------ | -------------- |
| `GET /`                              | Planned        |
| `POST /api/v1/events`                | Planned        |
| `GET /api/v1/events`                 | Planned        |
| `GET /api/v1/events/:id`             | Planned        |
| `PATCH /api/v1/events/:id/processed` | Planned        |
| `GET /api/v1/events/summary`         | Planned        |
| `GET /api/v1/reconciliation`         | Future planned |
| `POST /api/v1/reconciliation/run`    | Future planned |
| `GET /api/v1/providers`              | Future planned |
| `GET /api/v1/metrics`                | Future planned |

---

## Notes

This API reference will be updated as real backend routes are implemented.

During early development, this document describes the intended API design. Once implementation begins, examples should be updated to match the actual controller routes, DTOs, database model, and response shapes.
