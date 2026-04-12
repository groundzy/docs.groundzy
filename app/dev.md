# Development & Dev Tools

Development-only features.

## API Routes

| Route | Purpose |
|-------|---------|
| `POST /api/dev/create-user` | Create dev user (development only) |
| `POST /api/dev/simulate-payment` | Simulate payment (development only) |

## Components

| Component | Purpose |
|-----------|---------|
| `components/dev/dev-payment-button.tsx` | Dev payment simulation button (Profile drawer) |

## Guards

- Both dev routes check `process.env.NODE_ENV !== 'development'` and return 403 in production.
