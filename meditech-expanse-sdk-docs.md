# MEDITECH Expanse SDK
## Developer Documentation & API Reference

---

## Table of Contents

1. [Overview](#overview)
2. [Authentication](#authentication)
3. [Installation](#installation)
4. [SDK Usage](#sdk-usage)
5. [API Reference](#api-reference)
6. [MEDITECH Integration](#meditech-integration)
7. [Onboarding Process](#onboarding-process)
8. [Testing](#testing)

---

## Overview

The MEDITECH Expanse SDK enables seamless integration of AI-powered medical transcription and note generation capabilities into healthcare applications. This SDK provides real-time audio recording, secure authentication, and automated clinical documentation workflow.

### Key Features

- **Real-time Audio Recording**: Capture and process medical conversations
- **Secure Authentication**: JWT-based authentication with provider validation
- **Automated Note Generation**: AI-powered SOAP note creation
- **EHR Integration**: Direct writeback to MEDITECH Expanse systems
- **React Native Support**: Native mobile application integration

---

## Authentication

### Authentication Flow

The SDK uses a multi-layered authentication approach combining application-level and provider-level security.

#### Basic Authentication (Current Implementation)

```mermaid
sequenceDiagram
    participant App as Client App
    participant SDK as Scribe SDK
    participant API as Scribe API
    participant EHR as MEDITECH EHR
    
    App->>SDK: Initialize with credentials
    SDK->>API: isLicensed API call
    API->>API: Validate provider & organization
    API->>SDK: Return JWT token
    SDK->>App: Authentication complete
    App->>SDK: Start recording session
    SDK->>API: Authenticated API calls
```

#### Authentication Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `secret` | string | Yes | Client application secret |
| `provider_email` | string | Yes | Provider's email address |
| `provider_npi` | string | Yes | Provider's National Provider Identifier |
| `organization_id` | string | Yes | Organization identifier |

#### SSO Support

While Basic Auth is the primary method for initial implementation, the system supports:

- **SAML Integration**: Generic SSO with SAML support
- **Active Directory**: Integration with hospital AD systems
- **Federated Authentication**: Delegation to hospital Identity Providers

### User Identity Management

#### Provider Onboarding

Providers are registered during the onboarding process with the following information:

- Provider Name
- Provider Email
- Provider NPI
- Provider ID
- Organization ID

#### License Validation

The `isLicensed` API validates provider access and returns appropriate responses:

- **200 OK**: Valid license with JWT token
- **401 Unauthorized**: Invalid credentials or unlicensed user

---

## Installation

### Prerequisites

- Node.js 16+ 
- React Native development environment
- npm or yarn package manager

### NPM Registry Configuration

Create a `.npmrc` file in your project root:

```bash
@getathelas:registry=https://npm.pkg.github.com/
//npm.pkg.github.com/:_authToken=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
```

### Package Installation

```bash
npm install @getathelas/scribe-sdk-react
```

---

## SDK Usage

### Basic Setup

#### 1. Provider Wrapper

Wrap your application with the `ScribeProvider`:

```jsx
import { ScribeProvider } from '@getathelas/scribe-sdk-react/native';

export default function App() {
  return (
    <ScribeProvider>
      {/* Your application components */}
    </ScribeProvider>
  );
}
```

#### 2. Recording Component

Implement the recording functionality:

```jsx
import { ScribeRecorder } from "@getathelas/scribe-sdk-react/native/components";
import { SafeAreaView } from 'react-native';

const RecordingScreen = ({ appointment, userCredentials }) => {
  const handleStatusChange = (status) => {
    console.log('Recording status:', status);
    // Handle status changes: 'recording', 'paused', 'stopped', 'processing'
  };

  return (
    <SafeAreaView>
      <ScribeRecorder
        auth={{
          secret: "<client-secret>",
          userData: {
            provider_npi: "<provider_npi>",
            provider_email: "<provider_email>",
            organization_id: "<organization_id>",
          }
        }}
        appointment={appointmentData}
        onStatusChanged={handleStatusChange}
      />
    </SafeAreaView>
  );
};
```

### Type Definitions

#### Authentication Interface

```typescript
interface OrganizationUserAuthOpts {
  secret: string;
  userData: {
    provider_email: string;
    provider_npi: string;
    organization_id: string;
  };
}
```

#### Appointment Interface

```typescript
interface SessionAppointment {
  patient_first_name: string;
  patient_last_name: string;
  patient_date_of_birth: string; // YYYY-MM-DD format
  patient_gender: string;
  encounter_id: string;
  patient_id: string;
  scribe_id?: string;
  ai_workflow_id?: string;
  metadata_from_ehr: {
    patient_ref_id: string;
    encounter_ref_id: string;
    practitioner_ref_id: string;
    document_ref_id: string;
  };
}
```

### Authentication Methods

#### Independent Authentication

```jsx
// Authenticate after user logs into MEDITECH
const authenticateUser = async () => {
  // Authentication handled by SDK with provided credentials
};
```

#### Integrated Authentication

```jsx
// Authentication triggered automatically when recording screen loads
<ScribeRecorder
  auth={authConfig}
  appointment={appointmentData}
  // Authentication occurs on component mount
/>
```

---

## API Reference

### Base URL

```
{base_api_url}/v1/scribe/
```

### Authentication Endpoints

#### `POST /sdk/isLicensed`

Authenticates the SDK and returns a JWT token.

**Request:**
```json
{
  "secret": "client_secret",
  "user_data": {
    "provider_npi": "1234567890",
    "provider_email": "provider@example.com",
    "organization_id": "TEST_AUGMEDIX_1"
  }
}
```

**Response:**
```json
{
  "auth_token": "eyJhbGciOiJIUzI1NiIs...",
  "expires_at": "2025-05-17T02:35:58.392069",
  "token_type": "Bearer"
}
```

**Error Responses:**
- `401 Unauthorized`: Invalid credentials or unlicensed user
- `400 Bad Request`: Missing or invalid parameters

### Recording Endpoints

#### `POST /generate_signed_urls`

Generates signed URLs for audio file uploads.

**Request:**
```json
{
  "upload_session_id": "1fef2112-a854-4313-b28d-d1953b724806",
  "filenames": [
    "1fef2112-a854-4313-b28d-d1953b724806-chunk-0.wav",
    "1fef2112-a854-4313-b28d-d1953b724806-chunk-1.wav",
    "1fef2112-a854-4313-b28d-d1953b724806-manifest.json"
  ]
}
```

**Response:**
```json
{
  "signed_urls_map": {
    "1fef2112-a854-4313-b28d-d1953b724806-chunk-0.wav": "https://storage.googleapis.com/...",
    "1fef2112-a854-4313-b28d-d1953b724806-chunk-1.wav": "https://storage.googleapis.com/...",
    "1fef2112-a854-4313-b28d-d1953b724806-manifest.json": "https://storage.googleapis.com/..."
  }
}
```

#### `POST /workflows/trigger`

Initiates the note generation process.

**Request (Form Data):**
```json
{
  "manifest_file_signed_url": "https://storage.googleapis.com/...",
  "template_ids": "1362",
  "patient_name": "John Doe"
}
```

**Response:**
```json
{
  "scribe_id": 1002775,
  "workflow": 987718,
  "success": true
}
```

### Appointment Management

#### `POST /sdk/create_appointment`

Creates appointment metadata for note writeback.

**Request:**
```json
{
  "patient_first_name": "Patient_1",
  "patient_last_name": "Test",
  "patient_date_of_birth": "1990-03-01",
  "patient_gender": "M",
  "provider_npi": "1234567890",
  "scribe_id": "1002775",
  "encounter_id": "encounter4567",
  "patient_id": "patient1234",
  "metadata_from_ehr": {
    "patient_ref_id": "patient1234",
    "encounter_ref_id": "encounter4567",
    "practitioner_ref_id": "practitioner789",
    "document_ref_id": "document321"
  }
}
```

**Response:**
```json
{
  "appointment_id": "123456"
}
```

### Monitoring & Logging

#### `POST /sdk/log`

Logs SDK workflow events and errors.

**Request:**
```json
{
  "level": "ERROR",
  "message": "User login failed",
  "timestamp": "2023-11-01T12:34:56Z",
  "context": {
    "ip": "192.168.1.1"
  },
  "sdk_version": "1.4.1",
  "session_id": "123456",
  "user": "provider@example.com"
}
```

**Response:**
```json
{
  "status": "success",
  "message": "Log entry recorded"
}
```

### Workflow Data Retrieval

#### `GET /workflows/workflow_data/{workflow_id}`

Retrieves transcription and generated notes.

**Response:**
```json
{
  "scribe_id": 1012637,
  "workflow_id": 987718,
  "appointment_id": 5448422,
  "patient_name": "John Doe",
  "date_of_service": "2025-05-19T23:53:16.881393",
  "scribe_status": "completed",
  "audio_length_in_seconds": 69,
  "transcriptions": [
    {
      "transcription_id": 990403,
      "transcription_status": "STT_COMPLETED",
      "raw_text": "Complete transcription text...",
      "confidence_score": 0.8435459787733014
    }
  ],
  "workflow_tasks": [
    {
      "task_id": 1763738,
      "task_template_name": "SOAP",
      "task_status": "completed",
      "output_text": "Subjective:\nPatient presents with..."
    }
  ]
}
```

---

## MEDITECH Integration

### EHR Writeback Workflow

The SDK integrates with MEDITECH Expanse V2 APIs for note writeback through a multi-step process:

#### Step 1: Patient Search

```bash
curl --location 'https://greenfield-prod-apis.meditech.com/v2/uscore/R4/Patient?name=LastName%5C%2CFirstName&birthdate=1976-10-15&identifier=MRN123' \
--header 'Authorization: Bearer <token>'
```

#### Step 2: Encounter Search

```bash
curl --location 'https://greenfield-prod-apis.meditech.com/v4/patient-visit/?patient=<patient_ref_id>' \
--header 'Authorization: Bearer <token>'
```

#### Step 3: Encounter Details

```bash
curl --location 'https://greenfield-prod-apis.meditech.com/v2/uscore/STU7/Encounter/<encounter_ref_id>' \
--header 'Authorization: Bearer <token>'
```

#### Step 4: Note Writeback

```bash
curl --location 'https://greenfield-prod-apis.meditech.com/v2/uscore/STU7/Observation/' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <token>' \
--data '{
  "resourceType": "Observation",
  "subject": {
    "reference": "Patient/<patient_ref_id>",
    "type": "Patient"
  },
  "encounter": {
    "reference": "Encounter/<encounter_ref_id>",
    "type": "Encounter"
  },
  "category": [{
    "coding": [{
      "code": "document",
      "system": "https://greenfield-prod-apis.meditech.com:443/identifier/observation-category"
    }]
  }],
  "code": {
    "coding": [{
      "system": "http://loinc.org",
      "code": "10164-2"
    }]
  },
  "performer": [{
    "reference": "Practitioner/<practitioner_ref_id>",
    "type": "Practitioner"
  }],
  "status": "preliminary",
  "valueString": "Generated SOAP note content..."
}'
```

### Required Data Elements

For successful EHR integration, ensure the following data points are available:

| Field | Type | Required | Source |
|-------|------|----------|---------|
| Patient First Name | string | Yes | EHR |
| Patient Last Name | string | Yes | EHR |
| Patient Date of Birth | date | Yes | EHR |
| Patient MRN | string | Yes | EHR |
| Encounter ID | string | Yes | EHR |
| Appointment Start Time | datetime | Optional | EHR/Current |
| Document Reference ID | string | Yes | EHR |
| Provider NPI | string | Yes | Authentication |

---

## Onboarding Process

### Overview

The onboarding process establishes the necessary infrastructure for healthcare organizations to integrate with the Scribe system.

### Responsibilities

#### Commure/Athelas Responsibilities
1. Site creation in Scribe system
2. Facility setup per site
3. Provider account creation
4. Scribe account assignment
5. Technical configuration

#### Customer (Hospital) Responsibilities
1. Provide provider data
2. Notify of provider changes
3. Supply EHR integration credentials
4. Coordinate testing activities

### Onboarding Steps

#### 1. Site Creation
- Create unique site entry for health system
- Configure organizational settings
- Set up billing and administrative details

#### 2. Facility Setup
- Create facility entries per site
- Configure facility-specific settings
- Map to organizational structure

#### 3. Provider Onboarding

**Required Provider Data:**
```json
{
  "provider_id": "PROV001",
  "provider_email": "doctor@hospital.com",
  "provider_npi": "1234567890",
  "provider_name": "Dr. John Smith",
  "organization_id": "ORG001"
}
```

#### 4. Scribe Account Creation
- Generate Scribe accounts for note generation
- Link provider accounts to Scribe accounts
- Configure workflow templates

#### 5. Appointment Management
- Create appointment placeholders
- Configure scheduling integration
- Set up encounter mapping

### Post-Onboarding Maintenance

#### Provider Updates
- **Frequency**: As needed
- **Process**: Customer notifies Scribe of changes
- **Examples**: New providers, role changes, departures

#### Data Synchronization
- Maintain accurate provider information
- Update NPI numbers when changed
- Ensure organization mapping accuracy

---

## Testing

### Test Environment

#### Test Users

The following test accounts are available for integration testing:

**User 1:**
```json
{
  "user_email": "sdk_user_1@augmedix.com",
  "user_npi": "1234567890",
  "organization_id": "TEST_SDK_1"
}
```

**User 2:**
```json
{
  "user_email": "sdk_user_2@augmedix.com",
  "user_npi": "1234567891",
  "organization_id": "TEST_SDK_1"
}
```

### Testing Workflow

#### 1. Authentication Testing
```javascript
const testAuth = async () => {
  const authConfig = {
    secret: "test_client_secret",
    userData: {
      provider_email: "sdk_user_1@augmedix.com",
      provider_npi: "1234567890",
      organization_id: "TEST_SDK_1"
    }
  };
  
  // Test authentication flow
  const result = await authenticateUser(authConfig);
  console.log('Auth result:', result);
};
```

#### 2. Recording Testing
```javascript
const testRecording = () => {
  const appointmentData = {
    patient_first_name: "Test",
    patient_last_name: "Patient",
    patient_date_of_birth: "1990-01-01",
    patient_gender: "M",
    encounter_id: "test_encounter_123",
    patient_id: "test_patient_456",
    metadata_from_ehr: {
      patient_ref_id: "test_patient_456",
      encounter_ref_id: "test_encounter_123",
      practitioner_ref_id: "test_practitioner_789",
      document_ref_id: "test_document_321"
    }
  };
  
  // Test recording workflow
};
```

### Error Handling

#### Common Error Scenarios

| Error | Cause | Resolution |
|-------|-------|------------|
| 401 Unauthorized | Invalid credentials | Verify client secret and provider data |
| 403 Forbidden | Unlicensed user | Contact administrator for license assignment |
| 400 Bad Request | Invalid parameters | Check required fields and data format |
| 500 Internal Server Error | System error | Contact technical support |

#### Error Response Format

```json
{
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Provider not found or unlicensed",
    "details": {
      "provider_email": "provider@example.com",
      "organization_id": "ORG001"
    }
  }
}
```

---

## Support & Resources

### Technical Support
- **Email**: 
- **Documentation**: 
- **Status Page**: 

### SDK Versioning
- **Current Version**: 1.4.1
- **Release Notes**: Available in GitHub repository
- **Migration Guides**: Provided for major version updates

### Rate Limits
- **Authentication**: 10 requests/minute per organization
- **Recording APIs**: 100 requests/minute per provider
- **Logging**: 1000 requests/minute per organization

---

*Last Updated: June 2025*  
*SDK Version: 1.4.1*  
*API Version: v1*
