# System Integration Portfolio

This repository contains technical documentation for enterprise system integrations I've designed and built. Each project demonstrates cloud-native architecture patterns using Azure services, API integrations, and automated workflows.

---

## Projects

### [HR Dashboard](HR_Dashboard.md)
A single-page web application for HR teams to process employee onboarding and termination workflows. Built as an Azure Static Web App with Microsoft Entra ID authentication.

**Tech Stack:** HTML/CSS/JS, Azure Static Web Apps, Azure Functions, Microsoft Entra ID

---

### [Onboarding Automation](Onboarding_Function.md)
Azure Functions webhook service that automates new hire provisioning—creating user accounts in Microsoft Entra ID, assigning Microsoft 365 licenses, establishing manager relationships, and creating IT service desk tickets.

**Tech Stack:** Node.js, Azure Functions, Microsoft Graph API, SolarWinds Service Desk API

---

### [Employee Data Sync](Paylocity-employee-change.md)
Real-time synchronization of employee data changes from Paylocity HR platform to Microsoft Entra ID. Automatically propagates job title, department, and manager changes with full audit logging.

**Tech Stack:** Node.js, Azure Functions, Paylocity API, Microsoft Graph API, Application Insights

---

### [Salesforce Data Platform](Sales_Cat.md)
Full-stack platform that syncs Salesforce CRM data to Azure SQL and provides a searchable web interface. Features incremental sync, JWT authentication to Salesforce, and a React frontend.

**Tech Stack:** React, Express.js, Azure Functions, Azure SQL, Salesforce API, Microsoft Entra ID

---

### [SFTP File Monitor](SFTP_monitor_alert.md)
Azure Functions-based monitoring system that polls multiple SFTP directories and sends email notifications when new files arrive. Tracks state in Azure Blob Storage to prevent duplicate alerts.

**Tech Stack:** Node.js, Azure Functions, SSH2/SFTP, Azure Blob Storage, Microsoft Graph API

---

### [Termination Automation](Termination_Function.md)
Webhook service that automates employee offboarding by disabling user accounts in Microsoft Entra ID. Includes deny-list protection for critical accounts and creates audit trail tickets.

**Tech Stack:** Node.js, Azure Functions, Microsoft Graph API, SolarWinds Service Desk API

---

### [Offshore Employee Database](Offshore_Employee_Database.md)
Full-stack web application for tracking and managing offshore employees across the organization. Features employee CRUD operations, skills tracking, project allocations, and departmental assignments with a React/Vite frontend and Express.js backend.

**Tech Stack:** React, Vite, Express.js, Azure Static Web Apps, Azure App Service, Azure SQL, Microsoft Entra ID

---

## Common Patterns

These projects share several architectural patterns:

- **Azure Functions** for serverless compute and webhook handling
- **Microsoft Entra ID** for authentication and identity management
- **Microsoft Graph API** for user and license management
- **GitHub Actions** for CI/CD deployment pipelines
- **Multi-environment support** (sandbox/production) for safe testing

---

*Each document includes architecture diagrams, API contracts, data flows, and deployment details.*
