---
title: Client Flow
---

## Overview

The practitioner uses JHE to create the organization and study, register the patient, and send a secure invitation link.

When the patient opens the link on their iOS device, it launches the JHE Client via a configured link. The client extracts authentication parameters and exchanges them with JHE for an access token.

Using that token, the JHE Client retrieves the required consent scopes and presents them to the patient. The patient selects their consent preferences, which are recorded in JHE.

After consent is captured, the JHE Client uploads the patient’s health data as FHIR Observations. JHE validates, associates, and stores the data for the study.

```mermaid
sequenceDiagram
    autonumber

    actor Prac as Practitioner
    participant JHE as JHE
    actor Pat as Patient
    participant iOS as iOS Device
    participant Client as JHE Client

    rect rgb(245, 245, 245)
        note over Prac,JHE: Study and patient setup
        Prac->>JHE: Create Organization
        Prac->>JHE: Create Study
        Prac->>JHE: Register Patient<br/>(name, DOB, email/SMS)
        Prac->>JHE: Add Patient to Study
        JHE-->>Pat: Send secure invitation link<br/>(email or SMS)
    end

    rect rgb(235, 245, 255)
        note over Pat,Client: App launch and authentication
        Pat->>iOS: Open invitation link
        iOS->>Client: Launch JHE Client via configured link
        Client->>Client: Parse hostname and auth parameters
        Client->>JHE: Exchange auth parameters for access token
        JHE-->>Client: Return access token
    end

    rect rgb(240, 255, 240)
        note over Client,JHE: Consent flow
        Client->>JHE: Request consent requirements<br/>using access token
        JHE-->>Client: Return requested scopes<br/>(BP, heart rate, etc.)
        Client-->>Pat: Display consent options
        Pat->>Client: Select consent choices
        Client->>JHE: Submit consent decisions
        JHE-->>Client: Confirm consent recorded
    end

    rect rgb(255, 250, 235)
        note over Client,JHE: Data upload
        Client->>JHE: Upload data as FHIR Observations
        JHE->>JHE: Validate and store patient data
        JHE-->>Client: Confirm upload accepted
    end
```
