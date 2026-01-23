# Paylocity Onboarding Backend (v3)

## Executive Summary

The Paylocity Onboarding Backend is an **Azure Functions-based webhook service** that automates the new hire provisioning process. When triggered by the HR frontend or Paylocity system, it orchestrates a complete onboarding workflow: creating user accounts in Microsoft Entra ID, assigning Microsoft 365 licenses, establishing manager relationships, and creating service desk tickets for IT follow-up tasks.

---

## Business Value

| Benefit | Description |
|---------|-------------|
| **Automated Provisioning** | New hires get accounts within seconds of onboarding submission |
| **License Management** | Automatic Microsoft 365 E5 license assignment |
| **Org Structure** | Manager relationships established automatically |
| **IT Notification** | Service desk tickets created for additional setup tasks |
| **Multi-Tenant** | Supports sandbox testing and production environments |

---

## Architecture Overview

```
┌──────────────────────┐
│   HR Frontend /      │
│   Paylocity System   │
└──────────┬───────────┘
           │ POST /api/webhookhandler
           ▼
┌──────────────────────────────────────────────────────────┐
│              AZURE FUNCTION: webhookHandler              │
│              (paylocity-onboarding-prod-westus3-func)    │
└──────────┬───────────────────────────────────────────────┘
           │
    ┌──────┼──────────────┬─────────────────┐
    │      │              │                 │
    ▼      ▼              ▼                 ▼
┌────────┐ ┌────────┐ ┌────────────┐ ┌─────────────┐
│ Create │ │ Assign │ │ Assign     │ │ Create      │
│ User   │ │License │ │ Manager    │ │ SWSD Ticket │
│(Entra) │ │(Entra) │ │ (Entra)    │ │(SolarWinds) │
└────────┘ └────────┘ └────────────┘ └─────────────┘
   FATAL    NON-FATAL    NON-FATAL      NON-FATAL
```

---

## Workflow Steps

### Step 1: Create Entra ID User (Critical)
- Creates new user account in Microsoft Entra ID
- Sets display name, email (UPN), job title, department
- Assigns temporary password: `Newhire1!` (force change on login)
- **FATAL**: If this fails, entire operation fails

### Step 2: Assign Microsoft 365 License (Non-Critical)
- Retrieves available license SKUs from tenant
- Assigns Microsoft 365 E5 license (configurable)
- Continues even if license assignment fails

### Step 3: Assign Manager (Non-Critical)
- Parses supervisor name from Paylocity data
- Searches for manager in Entra ID directory
- Establishes manager-subordinate relationship
- Continues even if manager not found

### Step 4: Create Service Desk Ticket (Non-Critical)
- Creates incident in SolarWinds Service Desk
- Documents new hire details for IT follow-up
- CC's People department for awareness
- Continues even if ticket creation fails

---

## API Contract

### Webhook Endpoint

**POST** `https://paylocity-onboarding-prod-westus3-func.azurewebsites.net/api/webhookhandler`

#### Request Body
```json
{
  "companyId": "163160",
  "employeeFirstName": "John",
  "employeeLastName": "Doe",
  "employeeWorkEMailAddress": "john.doe@catalystsolutions.com",
  "employeeJobTitle": "Software Developer",
  "employeeCostCenter1": "Engineering",
  "employeeSupervisor": "Manager Name",
  "employmentType": "RFT"
}
```

#### Response Codes

| Status | Description |
|--------|-------------|
| `200` | Success - user created (sub-operations may have warnings) |
| `400` | Bad Request - invalid payload or missing companyId |
| `409` | Conflict - user already exists (duplicate UPN) |
| `500` | Server Error - configuration or processing failure |

#### Success Response
```json
{
  "message": "User created. See details.",
  "environment": "prod",
  "userId": "uuid-here",
  "userPrincipalName": "john.doe@catalystsolutions.com",
  "licenseResult": { "success": true },
  "managerResult": { "success": true, "manager": "manager@catalystsolutions.com" },
  "ticketResult": { "success": true, "id": "ticket-id" }
}
```

---

## External Service Integrations

### Microsoft Entra ID (Azure AD)
- **Authentication**: OAuth 2.0 Client Credentials
- **API**: Microsoft Graph API
- **Operations**: User creation, license assignment, manager assignment

### SolarWinds Service Desk
- **Authentication**: Bearer token (`X-Samanage-Authorization`)
- **Base URL**: `https://catalystsolutions.samanage.com`
- **Operations**: Incident ticket creation

---

## Environment Configuration

| Environment | Company ID | Azure Tenant | Domain |
|-------------|------------|--------------|--------|
| **Sandbox** | `CATALYST` | `ab558d63-c75b-4100-ae33-9160bbbcfbaa` | `healthplansai.dog` |
| **Production** | `163160` | `d39a588a-b5a6-4378-9f5f-f9a0e5484b06` | `catalystsolutions.com` |

---

## Data Mapping

| Paylocity Field | Entra ID Field | Notes |
|-----------------|----------------|-------|
| `employeeFirstName` | `givenName` | First name |
| `employeeLastName` | `surname` | Last name |
| `employeeWorkEMailAddress` | `userPrincipalName` | Login email |
| `employeeJobTitle` | `jobTitle` | Job title |
| `employeeCostCenter1` | `department` | Department |
| `employeeSupervisor` | `manager` | Manager relationship |

---

## Technical Stack

| Component | Technology |
|-----------|------------|
| **Runtime** | Azure Functions (Node.js 22.x) |
| **Authentication** | @azure/msal-node |
| **HTTP Client** | Axios |
| **Query Parsing** | qs |
| **Deployment** | GitHub Actions |
| **Region** | Azure West US 3 |

---

## Project Structure

```
onboarding-v3/
├── src/
│   ├── functions/
│   │   └── webhookHandler.js       # Main webhook handler
│   ├── env.js                      # Environment configuration
│   ├── graph.js                    # Entra ID/Graph API client
│   ├── swsd.js                     # SolarWinds client
│   └── utils.js                    # Helper functions
├── index.js                        # Function registration
├── host.json                       # Azure Functions config
├── package.json                    # Dependencies
└── .github/workflows/
    └── master_paylocity-onboarding-*.yml  # CI/CD
```

---

## Deployment Pipeline

```
┌─────────────────────────────────────┐
│         GitHub Repository           │
│          (master branch)            │
└──────────────────┬──────────────────┘
                   │ Push to master
                   ▼
┌─────────────────────────────────────┐
│         GitHub Actions              │
│  1. Checkout code                   │
│  2. Setup Node.js 22.x              │
│  3. npm install                     │
│  4. npm build (if present)          │
│  5. npm test (if present)           │
│  6. Zip and upload artifact         │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│    Azure Function App               │
│    paylocity-onboarding-prod-       │
│    westus3-func                     │
└─────────────────────────────────────┘
```

---

## Error Handling Strategy

| Operation | Failure Behavior | Rationale |
|-----------|-----------------|-----------|
| User Creation | **FATAL** - Return 500/409 | Cannot proceed without user account |
| License Assignment | **NON-FATAL** - Continue with warning | Admin can assign manually |
| Manager Assignment | **NON-FATAL** - Continue with warning | Manager may not exist yet |
| SWSD Ticket | **NON-FATAL** - Continue with warning | Informational only |

---

## Manager Name Parsing

The system handles multiple supervisor name formats:

| Format | Example | Parsing |
|--------|---------|---------|
| Email | `manager@domain.com` | Used as-is |
| "Last, First" | `Payne, Dan` | Parsed to "Dan Payne" |
| Standard | `Dan Payne` | Searched directly |

---

## Service Desk Ticket Details

**Ticket Properties**:
- **Priority**: 3 (Medium)
- **Category**: Account Management
- **Subcategory**: Onboarding
- **CC**: People@Catalystsolutions.com
- **Description**: Contains employee details, job title, department, employment type

---

## Summary

The Onboarding Backend automates the critical first steps of employee provisioning, ensuring new hires have accounts and licenses ready on their start date. Its resilient design allows the core user creation to succeed even when secondary operations (licensing, manager assignment, ticketing) encounter issues, with detailed response data enabling administrators to address any gaps.

---
