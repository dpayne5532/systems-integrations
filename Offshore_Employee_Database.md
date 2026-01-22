# Offshore Employee Database

## Executive Summary

The Offshore Employee Database is a **full-stack web application** designed to track and manage offshore employees across the organization. Built with a React/Vite frontend hosted on Azure Static Web Apps and a Node.js/Express backend on Azure App Service, it provides a centralized system for managing employee information, departmental assignments, skills tracking, and project allocations. The application integrates with Microsoft Entra ID for enterprise SSO authentication.

---

## Business Value

| Benefit | Description |
|---------|-------------|
| **Centralized Employee Data** | Single source of truth for all offshore employee information |
| **Enterprise Security** | Microsoft Entra ID SSO ensures only authorized users can access |
| **Audit Trail** | Login tracking via Application Insights for compliance |
| **Skills Management** | Track employee competencies for project staffing decisions |
| **Project Allocation** | Manage employee assignments across multiple projects |
| **Department Organization** | Organize employees by department with reporting structure |

---

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                     AZURE STATIC WEB APP                         │
│              zealous-coast-0755bb01e.6.azurestaticapps.net       │
│  ┌─────────────────────────┐  ┌─────────────────────────────┐   │
│  │   React/Vite Frontend    │  │   Azure Functions API       │   │
│  │   (Catalyst Branding)    │  │   /api/trackLogin           │   │
│  └───────────┬─────────────┘  └─────────────────────────────┘   │
└──────────────┼───────────────────────────────────────────────────┘
               │ HTTP + Bearer Token
               ▼
┌──────────────────────────────────────────────────────────────────┐
│                     AZURE APP SERVICE                            │
│              hr-offshore-database-webapp.azurewebsites.net       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │   Node.js/Express Backend                                │    │
│  │   - /api/employees    - /api/departments                 │    │
│  │   - /api/skills       - /api/projects                    │    │
│  └───────────┬─────────────────────────────────────────────┘    │
└──────────────┼───────────────────────────────────────────────────┘
               │ SQL Connection
               ▼
┌──────────────────────────────────────────────────────────────────┐
│                     AZURE SQL DATABASE                           │
│                      hr-offshore-db                              │
│  Tables: employees, departments, skills, projects,               │
│          employee_skills, employee_projects                      │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                    MICROSOFT ENTRA ID                            │
│                     (Authentication)                             │
│  App Registration: Offshore Employee Database                    │
│  Tenant: d39a588a-b5a6-4378-9f5f-f9a0e5484b06                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## Features

### Employee Management

| Feature | Description |
|---------|-------------|
| **Create/Edit/Delete** | Full CRUD operations for employee records |
| **Department Assignment** | Assign employees to departments |
| **Manager Hierarchy** | Set manager relationships |
| **Skills Tracking** | Assign multiple skills to employees |
| **Project Allocation** | Assign employees to projects with roles |

### Employee Data Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| First Name | Text | Yes | |
| Last Name | Text | Yes | |
| Email | Email | Yes | Unique constraint |
| Phone | Text | No | |
| Job Title | Text | Yes | |
| Department | Select | Yes | From departments table |
| Start Date | Date | Yes | |
| Salary | Decimal | No | |
| Contract Type | Select | No | full-time, part-time, contractor |
| Country | Text | Yes | |
| Timezone | Text | Yes | e.g., "Asia/Manila" |
| Office Location | Text | No | |
| Manager | Select | No | From employees table |

### Department Management

- Create and manage departments
- Track employee count per department
- Department descriptions

### Skills Management

- Define skill categories
- Assign multiple skills to employees
- Skill-based employee searching

### Project Management

- Create and manage projects
- Assign employees with roles
- Track project start/end dates

---

## User Interface Design

### Visual Design
- **Color Palette**: Catalyst brand green (#0AEF84), dark teal (#0E2F25)
- **Typography**: Inter font family
- **Layout**: Sidebar navigation with main content area
- **Branding**: Catalyst logo and favicon

### Dashboard Layout
```
┌────────────────────────────────────────────────────────────────┐
│  [Catalyst Logo]                                               │
│  Offshore Employee DB                                          │
├─────────────────┬──────────────────────────────────────────────┤
│                 │                                              │
│  Dashboard      │   ┌──────────┐ ┌──────────┐ ┌──────────┐    │
│  Employees      │   │ Total    │ │ Depts    │ │ Projects │    │
│  Departments    │   │ Employees│ │          │ │          │    │
│  Skills         │   └──────────┘ └──────────┘ └──────────┘    │
│  Projects       │                                              │
│                 │   Recent Employees                           │
│                 │   ┌────────────────────────────────────┐    │
│                 │   │ Name | Email | Dept | Country      │    │
│ ─────────────── │   │ ...                                │    │
│ [User Name]     │   └────────────────────────────────────┘    │
│ [Sign Out]      │                                              │
└─────────────────┴──────────────────────────────────────────────┘
```

---

## Authentication & Security

### Microsoft Entra ID Integration
- **App Registration**: Offshore Employee Database
- **Client ID**: `d843c456-b46e-4bed-b063-c0477ab69385`
- **Tenant ID**: `d39a588a-b5a6-4378-9f5f-f9a0e5484b06`
- **Auth Library**: MSAL React (@azure/msal-react)
- **Token Version**: v2.0

### API Security
- Bearer token authentication on all API endpoints
- JWT validation with JWKS (public key retrieval)
- Audience and issuer validation

### User Flow
1. User navigates to application URL
2. MSAL checks for existing authentication
3. If not authenticated, shows login page
4. User clicks "Sign in with Microsoft"
5. Redirects to Entra ID login
6. After successful login, redirected back with tokens
7. Access token sent with all API requests

---

## Backend API Endpoints

### Employees
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/employees` | List all employees |
| GET | `/api/employees/:id` | Get employee details |
| POST | `/api/employees` | Create employee |
| PUT | `/api/employees/:id` | Update employee |
| DELETE | `/api/employees/:id` | Delete employee |
| POST | `/api/employees/:id/skills` | Update employee skills |
| POST | `/api/employees/:id/projects` | Assign employee to project |

### Departments
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/departments` | List all departments |
| GET | `/api/departments/:id` | Get department details |
| POST | `/api/departments` | Create department |
| PUT | `/api/departments/:id` | Update department |
| DELETE | `/api/departments/:id` | Delete department |

### Skills
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/skills` | List all skills |
| POST | `/api/skills` | Create skill |
| PUT | `/api/skills/:id` | Update skill |
| DELETE | `/api/skills/:id` | Delete skill |

### Projects
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/projects` | List all projects |
| POST | `/api/projects` | Create project |
| PUT | `/api/projects/:id` | Update project |
| DELETE | `/api/projects/:id` | Delete project |

---

## Database Schema

```sql
┌─────────────────┐       ┌─────────────────┐
│   departments   │       │    employees    │
├─────────────────┤       ├─────────────────┤
│ id (PK)         │◄──────│ department_id   │
│ name            │       │ id (PK)         │
│ description     │       │ first_name      │
│ created_at      │       │ last_name       │
└─────────────────┘       │ email (UNIQUE)  │
                          │ phone           │
┌─────────────────┐       │ job_title       │
│     skills      │       │ start_date      │
├─────────────────┤       │ salary          │
│ id (PK)         │       │ contract_type   │
│ name            │       │ country         │
│ description     │       │ timezone        │
│ created_at      │       │ office_location │
└────────┬────────┘       │ manager_id (FK) │
         │                │ created_at      │
         │                │ updated_at      │
         │                └────────┬────────┘
         │                         │
         ▼                         ▼
┌─────────────────────┐   ┌─────────────────────┐
│  employee_skills    │   │  employee_projects  │
├─────────────────────┤   ├─────────────────────┤
│ employee_id (FK)    │   │ employee_id (FK)    │
│ skill_id (FK)       │   │ project_id (FK)     │
└─────────────────────┘   │ role                │
                          │ start_date          │
┌─────────────────┐       │ end_date            │
│    projects     │       └─────────────────────┘
├─────────────────┤                │
│ id (PK)         │◄───────────────┘
│ name            │
│ description     │
│ status          │
│ start_date      │
│ end_date        │
│ created_at      │
└─────────────────┘
```

---

## Technical Stack

### Frontend
| Component | Technology |
|-----------|------------|
| **Framework** | React 19 with Vite 7 |
| **Routing** | React Router DOM 7 |
| **Auth Library** | @azure/msal-react, @azure/msal-browser |
| **HTTP Client** | Axios |
| **Styling** | CSS with Custom Properties |
| **Hosting** | Azure Static Web Apps |

### Backend
| Component | Technology |
|-----------|------------|
| **Runtime** | Node.js 22.x |
| **Framework** | Express.js |
| **Database Driver** | mssql |
| **Validation** | express-validator |
| **Auth** | jsonwebtoken, jwks-rsa |
| **Security** | Helmet, CORS |
| **Hosting** | Azure App Service |

### Infrastructure
| Component | Technology |
|-----------|------------|
| **Database** | Azure SQL Database |
| **Authentication** | Microsoft Entra ID |
| **Monitoring** | Application Insights |
| **CI/CD** | GitHub Actions |

---

## Project Structure

### Frontend
```
odb-frontend/
├── public/
│   ├── favicon.ico              # Catalyst favicon
│   ├── logo-icon.png            # Chevron logo
│   └── logo-full.png            # Full Catalyst logo
├── src/
│   ├── components/
│   │   ├── Layout.jsx           # Main layout with sidebar
│   │   └── Layout.css
│   ├── pages/
│   │   ├── Dashboard.jsx        # Dashboard with stats
│   │   ├── Employees.jsx        # Employee list
│   │   ├── EmployeeDetail.jsx   # Employee details
│   │   ├── EmployeeForm.jsx     # Create/edit employee
│   │   ├── Departments.jsx      # Department management
│   │   ├── Skills.jsx           # Skills management
│   │   ├── Projects.jsx         # Project management
│   │   └── Login.jsx            # Login page
│   ├── services/
│   │   └── api.js               # API client with auth
│   ├── config/
│   │   └── authConfig.js        # MSAL configuration
│   ├── App.jsx                  # Main app component
│   └── main.jsx                 # Entry point with MSAL
├── api/
│   ├── host.json                # Functions config
│   ├── package.json             # Functions dependencies
│   └── trackLogin/
│       ├── function.json        # Function binding
│       └── index.js             # Login tracking handler
├── index.html
├── staticwebapp.config.json     # SPA routing config
└── .github/workflows/
    └── azure-static-web-apps-*.yml
```

### Backend
```
odb-backend/
├── src/
│   ├── config/
│   │   ├── db.js                # Azure SQL connection
│   │   └── auth.js              # MSAL config
│   ├── middleware/
│   │   └── auth.js              # JWT validation middleware
│   ├── routes/
│   │   ├── employees.js         # Employee CRUD
│   │   ├── departments.js       # Department CRUD
│   │   ├── skills.js            # Skills CRUD
│   │   └── projects.js          # Projects CRUD
│   └── server.js                # Express app entry
├── database/
│   └── schema.sql               # Database schema
├── package.json
└── .github/workflows/
    └── master_hr-offshore-database-webapp.yml
```

---

## Deployment Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   GitHub Repositories                            │
│  ┌──────────────────────┐  ┌──────────────────────┐            │
│  │ hr-offshore-db-      │  │ hr-offshore-db-      │            │
│  │ frontend             │  │ backend              │            │
│  └──────────┬───────────┘  └──────────┬───────────┘            │
└─────────────┼──────────────────────────┼────────────────────────┘
              │ Push                      │ Push
              ▼                           ▼
┌─────────────────────────┐  ┌─────────────────────────┐
│    GitHub Actions       │  │    GitHub Actions       │
│    (Frontend Build)     │  │    (Backend Build)      │
└──────────┬──────────────┘  └──────────┬──────────────┘
           │                             │
           ▼                             ▼
┌─────────────────────────┐  ┌─────────────────────────┐
│  Azure Static Web App   │  │  Azure App Service      │
│  zealous-coast-*        │  │  hr-offshore-database-  │
│  - React SPA            │  │  webapp                 │
│  - Azure Functions      │  │  - Node.js Express      │
└─────────────────────────┘  └──────────┬──────────────┘
                                        │
                                        ▼
                             ┌─────────────────────────┐
                             │   Azure SQL Database    │
                             │   hr-offshore-db        │
                             └─────────────────────────┘
```

---

## Login Tracking

### Azure Function: trackLogin
Logs user logins in a format compatible with Application Insights queries.

**Log Format**:
```
[HR Portal] User: {userName} ({userEmail}) | Action: Login | Time: {timestamp}
```

**KQL Query**:
```kql
traces
| where message contains "[HR Portal]"
| parse message with * "User: " user " | Action: " action " | Time: " logTime
| project logTime, user, action
| order by logTime desc
```

---

## Environment Variables

### Frontend (Build-time)
| Variable | Description |
|----------|-------------|
| `VITE_AZURE_CLIENT_ID` | Entra ID App Client ID |
| `VITE_AZURE_TENANT_ID` | Entra ID Tenant ID |
| `VITE_API_URL` | Backend API base URL |

### Backend
| Variable | Description |
|----------|-------------|
| `AZURE_CLIENT_ID` | Entra ID App Client ID |
| `AZURE_TENANT_ID` | Entra ID Tenant ID |
| `DB_SERVER` | Azure SQL server hostname |
| `DB_NAME` | Database name |
| `DB_USER` | Database username |
| `DB_PASSWORD` | Database password |
| `FRONTEND_URL` | Allowed CORS origin |

---

## Summary

The Offshore Employee Database provides a comprehensive solution for managing offshore workforce data. Its modern React frontend offers an intuitive user experience with Catalyst branding, while the Express backend provides a robust API with full authentication. The integration with Microsoft Entra ID ensures enterprise-grade security, and Application Insights provides visibility into user activity. The system's modular design allows for easy extension with additional features as business needs evolve.

---

*Document generated: January 2026*
