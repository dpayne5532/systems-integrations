# HR Frontend Suite - P&C Management Dashboard

## Executive Summary

The HR Frontend Suite is a **single-page web application** that provides a modern, user-friendly interface for HR/People & Culture teams to process employee **onboarding** and **termination** workflows. Built as an Azure Static Web App with Microsoft Entra ID authentication, it serves as the primary user interface for initiating employee lifecycle events that trigger backend automation.

---

## Business Value

| Benefit | Description |
|---------|-------------|
| **Simplified HR Operations** | Clean, intuitive interface for common HR tasks |
| **Enterprise Security** | Microsoft Entra ID SSO ensures only authorized users can access |
| **Audit Trail** | All user actions logged to Azure Functions for compliance |
| **Real-Time Processing** | Immediate feedback on onboarding/termination submissions |
| **Mobile Ready** | Responsive design works on desktop and mobile devices |

---

## Architecture Overview

```
┌──────────────────────┐         ┌──────────────────────┐
│                      │         │                      │
│   AZURE STATIC       │────────▶│  BACKEND AZURE       │
│   WEB APP            │  HTTP   │  FUNCTIONS           │
│   (Frontend UI)      │         │                      │
│                      │         │  - Onboarding API    │
└──────────┬───────────┘         │  - Termination API   │
           │                     │  - Access Logging    │
           │ SSO                 └──────────────────────┘
           ▼
┌──────────────────────┐
│  MICROSOFT ENTRA ID  │
│  (Authentication)    │
└──────────────────────┘
```

---

## Features

### Employee Onboarding Card

**Purpose**: Add new team members to the system

| Field | Type | Notes |
|-------|------|-------|
| Company ID | Text (required) | Identifies the tenant |
| Client/Cost Center 1 | Text (required) | Department assignment |
| Employment Type | Dropdown | RFT, TFT, H1/C2C, Unknown |
| First Name | Text (required) | Employee first name |
| Last Name | Text (required) | Employee last name |
| Work Email | Auto-generated | `firstname.lastname@catalystsolutions.com` |

**Features**:
- Auto-generates email address as user types
- Loading spinner during submission
- Success/error feedback cards
- "Add Another Employee" reset functionality

### Employee Termination Card

**Purpose**: Process employee departures

| Field | Type | Notes |
|-------|------|-------|
| Company ID | Text (required) | Identifies the tenant |
| First Name | Text (required) | Employee first name |
| Last Name | Text (required) | Employee last name |
| Work Email | Text (required) | Employee email address |

**Features**:
- Automatic termination date (current timestamp)
- Error handling with user feedback
- "Process Another" reset functionality

---

## User Interface Design

### Visual Design
- **Color Palette**: Catalyst brand green (#0AEF84), dark teal backgrounds
- **Typography**: Montserrat font family
- **Layout**: 2-column grid (desktop), single column (mobile)
- **Animations**: Smooth card expand/collapse, hover effects, loading spinners

### Dashboard Layout
```
┌─────────────────────────────────────────────────────────┐
│              [Catalyst Logo]                            │
│           P&C Management Dashboard                      │
├────────────────────────┬────────────────────────────────┤
│                        │                                │
│  ┌──────────────────┐  │  ┌──────────────────────────┐  │
│  │ Employee         │  │  │ Employee                 │  │
│  │ Onboarding       │  │  │ Termination              │  │
│  │ (Green)          │  │  │ (Red)                    │  │
│  │                  │  │  │                          │  │
│  │ [Form Fields]    │  │  │ [Form Fields]            │  │
│  │                  │  │  │                          │  │
│  │ [Submit Button]  │  │  │ [Submit Button]          │  │
│  └──────────────────┘  │  └──────────────────────────┘  │
│                        │                                │
└────────────────────────┴────────────────────────────────┘
```

---

## Backend API Integrations

### Onboarding Service
- **Endpoint**: `https://paylocity-onboarding-prod-westus3-func.azurewebsites.net/api/webhookhandler`
- **Method**: POST
- **Payload**: Company ID, cost center, employment type, name, email

### Termination Service
- **Endpoint**: `https://paylocity-term-prod-westus3-func.azurewebsites.net/api/terminationhandler`
- **Method**: POST
- **Payload**: Company ID, name, email, termination date

### Access Logging Service
- **Endpoint**: `/api/logAccess` (internal Azure Function)
- **Events Tracked**:
  - `page_view` - Dashboard loaded
  - `onboarding_submit` - Onboarding form submitted
  - `termination_submit` - Termination form submitted

---

## Authentication & Security

### Microsoft Entra ID Integration
- **Provider**: Azure Active Directory (OpenID Connect)
- **Tenant**: `d39a588a-b5a6-4378-9f5f-f9a0e5484b06`
- **Protected Routes**: All routes require authentication
- **Unauthenticated Redirect**: `/.auth/login/aad`

### User Flow
1. User navigates to dashboard URL
2. Azure Static Web Apps checks authentication
3. If not authenticated, redirects to Entra ID login
4. User completes SSO login
5. Redirected back to dashboard with session established
6. User identity available via `x-ms-client-principal-name` header

---

## Technical Stack

| Component | Technology |
|-----------|------------|
| **Frontend** | Vanilla HTML5, CSS3, JavaScript (no frameworks) |
| **Styling** | CSS Custom Properties, Flexbox, Grid |
| **Fonts** | Google Fonts (Montserrat) |
| **Hosting** | Azure Static Web Apps |
| **Backend** | Azure Functions (Node.js) |
| **Authentication** | Microsoft Entra ID |
| **CI/CD** | GitHub Actions |

---

## Deployment Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   GitHub Repository                      │
│                    (master branch)                       │
└──────────────────────────┬──────────────────────────────┘
                           │ Push to master
                           ▼
┌─────────────────────────────────────────────────────────┐
│                   GitHub Actions                         │
│  • Build static files                                    │
│  • Package Azure Functions                               │
│  • Deploy to Azure Static Web Apps                       │
└──────────────────────────┬──────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────┐
│         Azure Static Web App: witty-stone-067993f0f     │
│  ┌─────────────────────┐  ┌─────────────────────────┐   │
│  │   Static Content    │  │   Azure Functions       │   │
│  │   (HTML/CSS/JS)     │  │   (/api/logAccess)      │   │
│  └─────────────────────┘  └─────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## Project Structure

```
hr-frontendsuite-combined/
├── index.html                          # Single-page application
├── favicon.ico                         # Catalyst logo favicon
├── Catalyst_logo_bright_green.png      # Header logo
├── staticwebapp.config.json            # Azure SWA + auth config
├── api/
│   ├── host.json                       # Functions runtime config
│   ├── package.json                    # Node.js dependencies
│   └── logAccess/
│       ├── function.json               # Function binding
│       └── index.js                    # Access logging handler
└── .github/workflows/
    └── azure-static-web-apps-*.yml     # CI/CD pipeline
```

---

## User Workflows

### Onboarding an Employee
1. Log in via Microsoft Entra ID SSO
2. Click "Employee Onboarding" card to expand
3. Enter company ID and cost center
4. Select employment type from dropdown
5. Enter employee first and last name
6. Email auto-generates (editable if needed)
7. Click "Submit Onboarding"
8. View success confirmation
9. Click "Add Another Employee" to repeat

### Terminating an Employee
1. Click "Employee Termination" card to expand
2. Enter company ID
3. Enter employee first name, last name, and email
4. Click "Submit Termination"
5. View success confirmation
6. Click "Process Another" to repeat

---

## Summary

The HR Frontend Suite provides a streamlined, secure interface for HR teams to manage employee lifecycle events. Its simple architecture (vanilla HTML/CSS/JS) ensures fast loading and easy maintenance, while Azure Static Web Apps provides enterprise-grade hosting with built-in authentication. The dashboard connects seamlessly to backend Azure Functions that handle the actual provisioning and deprovisioning in Paylocity and Microsoft Entra ID.

---

*Document generated: January 2026*
