# Hooks

Custom React hooks in `hooks/`.

## Auth & Access

| Hook | Purpose |
|------|---------|
| useAuth | User, userDoc, organization |
| useProOrTeamsAccess | Pro/Teams feature access |
| useTeamsOnlyAccess | Teams-only feature access |
| useProCheckout | Pro checkout |
| usePlusCheckout | Plus checkout |
| useIsGlobalAdmin | Global admin check |

## Data – Trees & Species

| Hook | Purpose |
|------|---------|
| useTrees | Trees list |
| useTree | Single tree |
| useTreeUsage | Tree limits |
| useSpeciesCatalog | Species catalog |
| useSearchSpecies | Species search |
| useSpeciesDisplayName | Species display name |
| useQuickPicks | Quick picks |
| useSharedTreeIds | Shared tree IDs |
| useAccessRequestsForOwner | Tree access requests |

## Data – CRM

| Hook | Purpose |
|------|---------|
| useClients | Clients list |
| useClient | Single client |
| useProperties | Properties list |
| useProperty | Single property |
| useRequests | Requests list |
| useQuotes | Quotes list |
| useJobs | Jobs list |
| useInvoices | Invoices list |

## Data – Zones & Map

| Hook | Purpose |
|------|---------|
| useZones | Zones list |
| useZone | Single zone |
| useMapboxPlaceLabel | Reverse geocode label |
| useWeather | Weather data |
| useWeatherContext | Weather + recommendations |
| usePublicTreeSummaries | Public tree summaries |

## AI & Usage

| Hook | Purpose |
|------|---------|
| useAiChatContext | AI chat context |
| useAiUsage | AI usage limits |

## Other

| Hook | Purpose |
|------|---------|
| useContactPro | Contact pro (Hire Pro) |
| useConversations | Support/DM conversations |
| useGalleryCount | Gallery photo count |
| useNotifications | Notifications |
| useUnreadMessages | Unread message count |
| useUnreadNotifications | Unread notification count |
| useUserPreferences | Units, timezone, locale |
| usePublicProfile | Public profile |
| useTutorialState | Tutorial state |
