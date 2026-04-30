---
title: Admin API
---

### Admin REST API

The Admin API is used by the Web UI SPA for Practitioner/Patient/Organization/Study management and Patient data provider apps/clients to manage Patient consents.

#### Profile

The `profile` endpoint returns the current user details.

```json
// GET /api/v1/users/profile
{
  "id": 10001,
  "email": "peter@example.com",
  "firstName": "Peter",
  "lastName": "ThePatient",
  "patient": {
    "id": 40001,
    ...
  }
}
```

#### Patient Consents

The `consents` endpoint returns the studies that are pending and consented for the specified Patient. In this example, the Patient has been invited to *Demo Study 2* and has already consented to sharing blood glucose data with *Demo Study 1*.

```json
// GET /api/v1/patients/40001/consents
{
  "patient": {
    "id": 40001,
    //...
  },
  "consolidatedConsentedScopes": [
    {
      "id": 50002,
      "codingSystem": "https://w3id.org/openmhealth",
      "codingCode": "omh:blood-pressure:4.0",
      "text": "Blood pressure"
    }
  ],
  "studiesPendingConsent": [
    {
      "id": 30002,
      "name": "Demo Study 2",
      "organization": { ... },
      "dataSources": [ ... ],
      "pendingScopeConsents": [
        {
          "code": {
            "id": 50002,
            "codingSystem": "https://w3id.org/openmhealth",
            "codingCode": "omh:blood-pressure:4.0",
            "text": "Blood pressure"
          },
          "consented": null
        }
      ]
    }
  ],
  "studies": [
    {
      "id": 30001,
      "name": "Demo Study 1",
      "organization": { ... },
      "dataSources": [ ... ],
      "scopeConsents": [
        {
          "code": {
            "id": 50001,
            "codingSystem": "https://w3id.org/openmhealth",
            "codingCode": "omh:blood-glucose:4.0",
            "text": "Blood glucose"
          },
          "consented": true
        }
      ]
    }
  ]
}
```

To respond to requested consents, a POST is sent to the same `consents` endpoint with the scope and the `consented` boolean.

```json
// POST /api/v1/patients/40001/consents
{
  "studyScopeConsents": [
    {
      "studyId": 30002,
      "scopeConsents": [
        {
          "codingSystem": "https://w3id.org/openmhealth",
          "codingCode": "omh:blood-pressure:4.0",
          "consented": true
        }
      ]
    }
  ]
}

```

- A `PATCH` request can be sent with the same payload to update an existing Consent.
- A `DELETE` request can be sent with the same payload excluding `scopeConsents.consented` to delete the Consent.
