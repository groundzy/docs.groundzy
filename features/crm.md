# CRM Feature (Clients, Properties, Workflow)

CRM covers clients, properties, and the workflow: requests → quotes → jobs → invoices. Available on **Pro** and **Teams** tiers.

## Overview

| Entity | Drawer IDs | Collection |
|--------|------------|------------|
| Clients | `clients`, `view-client`, `add-client`, `edit-client` | `clients` |
| Properties | `properties`, `view-property`, `add-property`, `edit-property` | `properties` |
| Requests | `requests`, `view-request`, `add-request`, `edit-request` | `requests` |
| Quotes | `quotes`, `view-quote`, `add-quote`, `edit-quote` | `quotes` |
| Jobs | `jobs`, `view-job`, `add-job`, `edit-job` | `jobs` |
| Invoices | `invoices`, `view-invoice`, `add-invoice` | `invoices` |

**Requests, Quotes, Jobs, Invoices**: Teams tiers only (Small Team, Mid Team, Large Team, Enterprise).

## Clients

- **Location**: `app/drawers/clients.tsx`, `view-client`, `client-form.tsx`
- **Fields**: firstName, lastName, companyName, emails (primary), phones (mobile, etc.)
- **Search**: By name, company, email, phone
- **Hooks**: `useClients`, `useClient`
- **Access**: `useProOrTeamsAccess()` – upgrade prompt for Home/Plus

## Properties

- **Location**: `app/drawers/properties.tsx`, `view-property`, `property-form.tsx`
- **Fields**: address, geometry (GeoJSON), clientId, zoneId
- **Map**: Property polygons on map (`property-layer.tsx`)
- **Hooks**: `useProperties`, `useProperty`
- **Plus-only**: Some property features gated; `useIsPlusOnlyProperties()`

## Requests

- **Location**: `app/drawers/requests.tsx`, `view-request`, `request-form.tsx`
- **Fields**: clientId, status, description, etc.
- **Workflow**: Request → Quote → Job → Invoice
- **Hooks**: `useRequests`, `useRequest`

## Quotes

- **Location**: `app/drawers/quotes.tsx`, `view-quote`, `quote-form.tsx`
- **Fields**: clientId, requestId, status, amount, line items, etc.
- **Hooks**: `useQuotes`, `useQuote`

## Jobs

- **Location**: `app/drawers/jobs.tsx`, `view-job`, `job-form.tsx`
- **Fields**: clientId, status, schedule (startDate), propertyId, etc.
- **Dashboard**: Upcoming jobs in `getDashboardUpcoming()`
- **Weather**: `job_reschedule` recommendation for bad weather
- **Hooks**: `useJobs`, `useJob`

## Invoices

- **Location**: `app/drawers/invoices.tsx`, `view-invoice`, `invoice-form.tsx`
- **Fields**: clientId, jobId, status, line items, due date, etc.
- **Hooks**: `useInvoices`, `useInvoice`

## Tier Access

| Tier | Clients | Properties | Requests | Quotes | Jobs | Invoices |
|------|---------|------------|----------|--------|------|----------|
| Home | — | — | — | — | — | — |
| Plus | — | Limited | — | — | — | — |
| Pro | ✓ | ✓ | — | — | — | — |
| Teams | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

- **ProOrTeamsAccess**: `useProOrTeamsAccess()` – for Clients, Properties
- **TeamsOnlyAccess**: `useTeamsOnlyAccess()` – for Requests, Quotes, Jobs, Invoices
- **Upgrade**: `useProCheckout()` for Pro, team invite for Teams

## Data Scoping

- **organizationId**: Team users – data scoped by team
- **databaseCode**: Pro individual – scoped by user's databaseCode
- **organizationId == uid**: Legacy/home-tier fallback

## Firestore Indexes

Composite indexes for `organizationId` + `status`, `clientId`, `createdAt`, etc. See `firebase/firestore.indexes.json`.
