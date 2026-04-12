# Hire a Pro Feature

Connects users with tree care professionals (arborists, landscapers) near their location.

## Overview

- **Drawer ID**: `hire-groundzy-pro`
- **Location**: `app/drawers/hire-groundzy-pro.tsx`
- **Tier**: Home, Plus only (hidden for Pro/Teams)

## Flow

1. **Intro steps**: 4-step intro (Info, UserCheck, UserPlus, MessageCircle) – explains how it works
2. **Location**: Map center or user GPS; reverse geocode for address
3. **Nearby pros**: 
   - **Groundzy Certified Pros**: `GET /api/groundzy-pros/nearby` – from `groundzy_pros` collection (admin-managed)
   - **Places API**: `GET /api/places/nearby-tree-services` – Google Places (tree service businesses)
4. **Pro cards**: Name, rating, address, phone, website, distance
5. **Contact**: `useContactPro()` – creates `pro_contact_requests` doc; triggers email (Cloud Function)
6. **Default pro**: User can set a default pro (`setDefaultPro`, `clearDefaultPro`) – stored in user doc. For a certified pro with a linked Groundzy business, a **dialog** confirms sharing trees and contact info with their team and (if the user has multiple properties) which properties/trees to include. Optional drawer params: `propertyId`, `treeId` (resolves property for “only this property”).

## Data

- **Groundzy Pros**: Firestore `groundzy_pros` – admin write only; API reads via Admin SDK
- **Pro contact requests**: Firestore `pro_contact_requests` – user creates; fields: `userId`, pro info
- **Places**: Google Places API – `GOOGLE_PLACES_API_KEY` or `GOOGLE_MAPS_API_KEY`

## APIs

| Route | Purpose |
|-------|---------|
| `GET /api/groundzy-pros/nearby` | Nearby Groundzy Certified Pros |
| `GET /api/places/nearby-tree-services` | Nearby tree services (Places) |

## Environment Variables

- `GOOGLE_PLACES_API_KEY` or `GOOGLE_MAPS_API_KEY` – for Places API
