# Pi Kingdoms - System Architecture

## High-Level Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                     FRONTEND (React)                             │
│  - Kingdom Building UI                                           │
│  - Resource Management Dashboard                                 │
│  - PvE Adventure Mode                                            │
│  - Leaderboard Visualization                                     │
│  - Pi Wallet Integration                                         │
│  - Dark/Light Mode Toggle                                        │
└──────────────────────────────────────────────────────────────────┘
                            │
                    HTTPS / WebSocket
                            │
┌──────────────────────────────────────────────────────────────────┐
│                    BACKEND (Node.js)                             │
│                                                                  │
│  ┌─────────────────┬──────────────────┬──────────────────┐      │
│  │   Auth          │   Game           │   Payment        │      │
│  │   Service       │   Service        │   Service        │      │
│  └─────────────────┴──────────────────┴──────────────────┘      │
│                                                                  │
│  ┌─────────────────┬──────────────────┬──────────────────┐      │
│  │   Leaderboard   │   Quest          │   Admin          │      │
│  │   Service       │   Service        │   Service        │      │
│  └─────────────────┴──────────────────┴──────────────────┘      │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │           Pi SDK Integration Layer                     │     │
│  │  - Wallet Authentication                              │     │
│  │  - A2U Payment Processing                             │     │
│  │  - KYC Verification Checking                          │     │
│  │  - Transaction Validation                             │     │
│  └────────────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────────────┘
      │                    │                      │
      ▼                    ▼                      ▼
┌──────────────┐  ┌──────────────┐  ┌────────────────────────────┐
│  PostgreSQL  │  │   Redis      │  │  Pi Network Mainnet        │
│  Database    │  │   Cache      │  │  / Testnet                 │
└──────────────┘  └──────────────┘  └────────────────────────────┘
```

## Component Architecture

### Frontend Layer

**Technology**: React 18 + TypeScript + Vite

```
frontend/src/
├── components/
│   ├── Kingdom/          # Main kingdom screen
│   ├── Adventure/        # PvE combat mode
│   ├── Marketplace/      # Trading system
│   ├── Leaderboard/      # Rankings display
│   ├── Quests/           # Mission UI
│   ├── Profile/          # User profile
│   └── Settings/         # Theme, audio, etc.
├── pages/
│   ├── Login.tsx         # Pi Wallet login
│   ├── Home.tsx          # Main menu
│   ├── Game.tsx          # Main game screen
│   └── NotFound.tsx      # 404 page
├── hooks/
│   ├── useGame.ts        # Game state management
│   ├── usePiWallet.ts    # Pi Wallet integration
│   ├── useLeaderboard.ts # Leaderboard queries
│   └── useSocket.ts      # WebSocket connection
├── services/
│   ├── api.ts            # Backend API calls
│   ├── piWallet.ts       # Pi SDK wrapper
│   ├── storage.ts        # LocalStorage helpers
│   └── analytics.ts      # Event tracking
├── store/
│   ├── gameSlice.ts      # Game state (Redux/Zustand)
│   ├── userSlice.ts      # User state
│   ├── uiSlice.ts        # UI state (theme, etc.)
│   └── store.ts          # Store configuration
├── types/
│   ├── game.ts           # Game models (Kingdom, Resource, etc.)
│   ├── api.ts            # API response types
│   ├── user.ts           # User profile types
│   └── index.ts          # Barrel exports
├── utils/
│   ├── calculations.ts   # Game math utilities
│   ├── formatting.ts     # Number/date formatting
│   ├── constants.ts      # Game constants
│   └── validators.ts     # Input validation
├── styles/
│   ├── themes/
│   │   ├── light.css
│   │   └── dark.css
│   ├── responsive.css    # Mobile-first breakpoints
│   └── animations.css    # UI animations
└── App.tsx               # Root component
```

**Key Features**:
- **State Management**: Zustand for simplicity (Redux alternative)
- **API Client**: Axios with interceptors for JWT
- **Real-time**: Socket.io for leaderboard updates
- **Offline Support**: Service Worker for offline gameplay
- **Responsive**: Mobile-first CSS with breakpoints

### Backend Layer

**Technology**: Node.js 18 + Express + TypeScript

```
backend/src/
├── controllers/
│   ├── authController.ts         # Pi Wallet auth
│   ├── gameController.ts         # Kingdom operations
│   ├── questController.ts        # Quest completion
│   ├── adventureController.ts    # PvE combat
│   ├── marketplaceController.ts  # Trading
│   ├── leaderboardController.ts  # Rankings
│   ├── paymentController.ts      # Pi payments
│   └── adminController.ts        # Admin operations
├── models/
│   ├── User.ts                   # User profile
│   ├── Kingdom.ts                # Kingdom state
│   ├── Building.ts               # Building definitions
│   ├── Resource.ts               # Resource inventory
│   ├── Quest.ts                  # Quest definitions
│   ├── Achievement.ts            # Achievement system
│   ├── Transaction.ts            # Pi transactions
│   ├── LeaderboardEntry.ts       # Ranking data
│   └── AuditLog.ts              # Admin logs
├── services/
│   ├── authService.ts            # JWT, KYC checks
│   ├── gameService.ts            # Game logic
│   ├── questService.ts           # Quest completion
│   ├── adventureService.ts       # Combat engine
│   ├── marketplaceService.ts     # Trading logic
│   ├── leaderboardService.ts     # Ranking calculation
│   ├── paymentService.ts         # Pi SDK wrapper
│   ├── antiCheatService.ts       # Cheat detection
│   └── analyticsService.ts       # Event tracking
├── middleware/
│   ├── auth.ts                   # JWT verification
│   ├── errorHandler.ts           # Global error handling
│   ├── validator.ts              # Input validation
│   ├── rateLimiter.ts            # Rate limiting
│   ├── logger.ts                 # Request logging
│   └── cors.ts                   # CORS configuration
├── routes/
│   ├── auth.ts                   # Auth endpoints
│   ├── game.ts                   # Game endpoints
│   ├── quests.ts                 # Quest endpoints
│   ├── adventure.ts              # Adventure endpoints
│   ├── marketplace.ts            # Marketplace endpoints
│   ├── leaderboard.ts            # Leaderboard endpoints
│   ├── payments.ts               # Payment endpoints
│   └── admin.ts                  # Admin endpoints
├── config/
│   ├── database.ts               # PostgreSQL setup
│   ├── redis.ts                  # Redis setup
│   ├── pi-sdk.ts                 # Pi SDK config
│   └── constants.ts              # Game constants
├── utils/
│   ├── logger.ts                 # Logging utility
│   ├── piWalletUtils.ts          # Pi SDK helpers
│   ├── validators.ts             # Input validators
│   └── errors.ts                 # Custom errors
├── migrations/
│   ├── 001_init_schema.sql       # Initial schema
│   ├── 002_add_achievements.sql  # Achievement system
│   └── 003_add_audit_log.sql     # Audit logging
└── tests/
    ├── unit/                     # Unit tests
    ├── integration/              # Integration tests
    └── e2e/                      # End-to-end tests
```

## Scalability Considerations

### Horizontal Scaling

1. **Load Balancer**: Nginx/HAProxy distributes traffic
2. **Backend Instances**: Multiple Node.js servers
3. **Session Management**: Redis for session storage
4. **Database**: Read replicas for reporting
5. **CDN**: CloudFront for static assets

### Performance Optimization

1. **Database Indexing**: Strategic indexes on frequently queried fields
2. **Query Optimization**: Avoid N+1 queries
3. **Caching**: Redis for hot data
4. **Compression**: gzip for responses
5. **Code Splitting**: Lazy loading of components

## Deployment Architecture

### Docker Configuration

```dockerfile
# backend/Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY src ./src
EXPOSE 3001
CMD ["npm", "start"]
```

### Kubernetes Deployment

```yaml
# k8s/backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pi-kingdoms-backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: pi-kingdoms-backend
  template:
    metadata:
      labels:
        app: pi-kingdoms-backend
    spec:
      containers:
      - name: backend
        image: pi-kingdoms:backend-latest
        ports:
        - containerPort: 3001
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

## Performance Targets

- **Page Load**: <3s on 4G
- **API Response**: <200ms p99
- **Database Query**: <100ms p99
- **Uptime**: 99.9%
- **Error Rate**: <0.1%