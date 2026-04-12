# Types and Stores Reference

## TypeScript Types (`types/`)

### tree.ts

- `HealthStatus`: "excellent" | "good" | "fair" | "poor" | "critical" | "dead"
- `RiskLevel`: "low" | "moderate" | "high" | "extreme" | "imminent"
- `PlantType`: "tree" | "shrub" | "bush" | "vine" | "perennial"
- `ServiceType`: enum (pruning, health, installation, emergency, assessment, etc.)
- `ServiceCategory`: enum (preventive, corrective, emergency, consulting, maintenance)
- `HistoryMetadata`, `Tree`, `TreeEntry`, etc.

### zone.ts

- `Zone`, zone geometry types

### client.ts

- `Client` and related types

### property.ts

- `Property` and related types

### request.ts, quote.ts, job.ts, invoice.ts

- Workflow entity types

### species.ts

- Species catalog types

### ai-chat.ts

- AI chat message, context types

### conversation.ts

- Support/DM conversation types

### quick-picks.ts

- Quick pick region, set, pick types

### team.ts

- Team, member roles

### gallery.ts

- User gallery types

### map.ts

- Map-related types

### mapbox-gl-draw.d.ts

- Mapbox Draw type augmentations

### certified-pro.ts, tutorial.ts, signup-flow.ts, session.ts

- Domain-specific types

## Zustand Stores (`stores/`)

### navigation-store.ts

- **State**: `sidebarOpen`, `workAreaState` (desktop/mobile), `drawerHistory`, `currentDrawerId`, `currentParams`
- **Persisted**: Yes (localStorage)
- **Purpose**: Sidebar, work area, drawer stack, URL sync

### map-store.ts

- **State**: `center`, `zoom`, `bounds`, etc.
- **Purpose**: Map viewport state

### selection-store.ts

- **State**: `selectedTreeId`, `selectedZoneId`, `selectedPropertyId`
- **Purpose**: Map selection (tree, zone, property)

### draw-store.ts

- **State**: Draw mode, polygon state
- **Purpose**: Mapbox Draw tool state

### measure-store.ts

- **State**: Measure tool state
- **Purpose**: Measurement tool

### tree-add-store.ts

- **State**: Add-tree form state (location, species, etc.)
- **Purpose**: Add tree flow

### auth-store.ts

- **State**: Auth UI state
- **Purpose**: Login/signup UI

### fast-mapping-store.ts

- **State**: Fast mapping mode
- **Purpose**: Quick add flow

### identifying-wand-store.ts

- **State**: AI identify wizard state
- **Purpose**: Species identification flow

### view-tree-header-store.ts

- **State**: View-tree header state
- **Purpose**: View tree drawer header

### breakout-placement-store.ts

- **State**: Breakout placement state
- **Purpose**: Breakout UI

## Subscription Tier Type

From `lib/drawer-registry.ts`:

```typescript
export type SubscriptionTier =
  | 'Home'
  | 'Plus'
  | 'Pro'
  | 'Small Team'
  | 'Mid Team'
  | 'Large Team'
  | 'Enterprise';
```

## Tier Utilities (`lib/utils/tier-utils.ts`)

- `getUserSubscriptionTier(userDoc)`: Raw tier from user doc
- `getEffectiveSubscriptionTier(userDoc)`: Tier for access control (Pro/Teams only if status active)
- `hasActivePaidSubscription(userDoc)`: True if paid tier with active subscription
- `hasPaidTierIncomplete(userDoc)`: True if paid tier but subscription not active
- `isTierInGroup(tier, group)`: Check tier in 'home' | 'plus' | 'pro' | 'teams'
- `normalizeTier(tier)`: Normalize string to SubscriptionTier
- `getWeatherCardTier(tier)`: Map to 'home' | 'pro' | 'team' for WeatherCard
