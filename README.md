# 🎮 Pi Kingdoms

**A Mobile-First Casual Strategy Game for the Pi Ecosystem**

Pi Kingdoms is a browser-based, blockchain-integrated strategy game designed for Pi Network players. Build kingdoms, collect resources, complete missions, and earn Pi rewards while fully complying with 2026 Pi Network standards.

## 🌟 Features

### Core Gameplay
- **Kingdom Building**: Construct and upgrade buildings (farms, mines, barracks, markets)
- **Resource Management**: Collect wood, stone, food, and Pi energy
- **Daily Missions & Weekly Challenges**: Earn rewards through consistent engagement
- **PvE Adventure Mode**: Explore maps, defeat monsters, collect treasures
- **Leaderboard System**: Compete globally with real-time rankings
- **Achievement Badges**: 50+ unlockable achievements and progression levels
- **In-Game Marketplace**: Trade cosmetic items securely

### Pi Network Integration
- **Pi Wallet Integration**: Secure login and payment via official Pi SDK
- **KYC-Aware Rewards**: Mainnet Pi withdrawals only for verified users
- **Multi-Currency Support**: Pure Pi, PI USD, OPEN USD, PI EUR
- **Transparent Pricing**: All in-game purchases clearly display Pi value
- **Anti-Cheat Protection**: Server-side validation for all transactions

### User Experience
- **Responsive UI**: Optimized for Android, iOS, and web browsers
- **Dark/Light Mode**: Theme toggle with persistent user preference
- **Onboarding Tutorial**: Interactive guide for new players
- **Animations & Sound**: Polished UI with optional audio
- **Offline Support**: Core gameplay works without connection

## 🛠️ Tech Stack

| Component | Technology |
|-----------|------------|
| Frontend | React 18 + TypeScript |
| Backend | Node.js + Express |
| Database | PostgreSQL |
| Real-time | WebSocket (Socket.io) |
| Auth | Pi Wallet SDK + JWT |
| Payments | Pi SDK (App-to-User) |
| Hosting | Docker + Kubernetes ready |
| Admin | React Admin Dashboard |

## 📦 Project Structure

```
pi-kingdoms/
├── frontend/                 # React web app
│   ├── src/
│   │   ├── components/      # Reusable UI components
│   │   ├── pages/           # Game screens
│   │   ├── hooks/           # Custom React hooks
│   │   ├── services/        # API & Pi SDK integration
│   │   ├── store/           # Redux/Zustand state
│   │   ├── types/           # TypeScript interfaces
│   │   └── utils/           # Helpers & utilities
│   ├── public/              # Static assets
│   └── package.json
│
├── backend/                 # Node.js API
│   ├── src/
│   │   ├── controllers/     # Route handlers
│   │   ├── models/          # Database models
│   │   ├── services/        # Business logic
│   │   ├── middleware/      # Auth, validation
│   │   ├── routes/          # API endpoints
│   │   ├── config/          # Env & settings
│   │   └── utils/           # Helpers
│   ├── migrations/          # DB migrations
│   ├── tests/               # Unit & integration tests
│   └── package.json
│
├── admin/                   # Admin Dashboard
│   ├── src/
│   │   ├── pages/           # Admin screens
│   │   ├── components/      # Admin UI
│   │   └── services/        # Admin API calls
│   └── package.json
│
├── docs/                    # Documentation
│   ├── API.md              # REST API reference
│   ├── ARCHITECTURE.md     # System design
│   ├── PI_NETWORK.md       # Pi integration guide
│   └── DEPLOYMENT.md       # Deployment steps
│
├── docker-compose.yml       # Local dev environment
└── .env.example            # Environment template
```

## 🚀 Quick Start

### Prerequisites
- Node.js 18+
- PostgreSQL 14+
- Docker & Docker Compose
- Pi Browser (for testing Pi Wallet)

### Development Setup

```bash
# Clone repository
git clone https://github.com/Atbasha/pi-kingdoms.git
cd pi-kingdoms

# Install dependencies
cd frontend && npm install
cd ../backend && npm install
cd ../admin && npm install

# Start development environment
docker-compose up -d

# Run migrations
cd backend && npm run migrate

# Start servers
# Terminal 1: Frontend
cd frontend && npm start

# Terminal 2: Backend
cd backend && npm start

# Terminal 3: Admin
cd admin && npm start
```

### Environment Variables

Copy `.env.example` to `.env` and configure:

```bash
# Pi Network
PI_API_KEY=your_pi_api_key
PI_WALLET_SERVER_URL=https://api.mainnet.minepi.com
PI_DEVELOPMENT=false

# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=pi_kingdoms
DB_USER=postgres
DB_PASSWORD=dev_password

# JWT
JWT_SECRET=your_jwt_secret
JWT_EXPIRY=7d

# App
NODE_ENV=development
FRONTEND_URL=http://localhost:3000
BACKEND_URL=http://localhost:3001
```

## 🎯 Gameplay Loop

1. **Login**: Connect via Pi Wallet (non-custodial, user controls keys)
2. **Collect Resources**: Passive generation from buildings
3. **Upgrade Buildings**: Invest resources to unlock new features
4. **Complete Quests**: Daily missions reward 5-50 Pi energy
5. **PvE Combat**: Adventure maps drop treasure chests
6. **Earn Rewards**: Gain Pi through achievements and marketplace trades
7. **Progress**: Unlock new regions and building tiers
8. **Compete**: Climb leaderboards and earn badges

## 💰 Reward System (Pi Network Compliant)

### In-Game Rewards (Testnet/Sandbox)
- Daily login bonuses
- Mission completion rewards
- Achievement unlocks
- Leaderboard prizes

### Mainnet Rewards (KYC Required)
- Weekly top-100 leaderboard Pi payouts
- Special event tournaments
- Referral bonuses
- All withdrawals require completed KYC verification

### Security
- Server-side transaction validation
- Rate limiting on reward claims
- Cheat detection via pattern analysis
- Admin override capability for disputes

## 🔐 Security Features

- **Pi Wallet Integration**: Uses official SDK, never stores user keys
- **JWT Authentication**: Secure session management
- **HTTPS/TLS**: All connections encrypted
- **Rate Limiting**: DDoS protection on all endpoints
- **Input Validation**: Sanitize all user inputs
- **Anti-Cheat**: Server-side verification of all game actions
- **Database Encryption**: Sensitive data at rest
- **Admin Logging**: All admin actions audited

## 📊 Admin Panel Features

- **User Management**: View, ban, modify player accounts
- **Reward Management**: Create and distribute special rewards
- **Event Management**: Schedule events and challenges
- **Analytics Dashboard**: Player engagement metrics
- **Transaction Audit**: Review all Pi transactions
- **Support Tickets**: Manage player issues
- **System Health**: Monitor server status and logs

## 📱 Responsive Design

- **Mobile First**: Optimized for 320px+ screens
- **Tablet Support**: Enhanced UI for 768px+ screens
- **Desktop**: Full feature set on 1024px+ screens
- **Touch Gestures**: Swipe, pinch, long-press support
- **Performance**: <3s load time on 4G, offline fallback

## 🎨 Dark/Light Mode

- Automatic detection of system preference
- Manual toggle in settings
- Persistent storage in localStorage
- WCAG AA compliance for both themes
- Reduced motion support for animations

## 🧪 Testing

```bash
# Backend unit tests
cd backend && npm test

# Backend integration tests
npm run test:integration

# Frontend component tests
cd frontend && npm test

# E2E tests
npm run test:e2e

# Coverage report
npm run test:coverage
```

## 📚 Documentation

- [API Reference](docs/API.md) - All REST endpoints
- [Architecture Guide](docs/ARCHITECTURE.md) - System design
- [Pi Network Integration](docs/PI_NETWORK.md) - Wallet & payment setup
- [Deployment Guide](docs/DEPLOYMENT.md) - Production setup
- [Contributing Guide](CONTRIBUTING.md) - How to contribute

## 🌐 Pi Network Compliance (2026)

This project adheres to:

✅ **HTML5/Browser-Based**: No app downloads required  
✅ **Mobile-First**: Optimized for Pi's global mobile audience  
✅ **Official SDK**: Uses CiDi Games SDK for wallet integration  
✅ **KYC Requirements**: Mainnet rewards require verification  
✅ **Multi-Currency**: Supports Pi, PI USD, OPEN USD, PI EUR  
✅ **Branding Guidelines**: Compliant with Pi ecosystem standards  
✅ **Fair Play**: Anti-cheat and rate limiting  
✅ **Privacy**: No unnecessary data exposure  

## 🚀 Deployment

See [DEPLOYMENT.md](docs/DEPLOYMENT.md) for:
- Docker setup
- Kubernetes configuration
- AWS/GCP/Azure deployment
- CDN integration
- Database backup strategy

## 📈 Scalability

Architecture supports:
- 10,000+ concurrent users
- Real-time leaderboards via WebSocket
- Horizontal scaling with load balancing
- Database replication and backup
- Cache layer (Redis) for performance

## 🤝 Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## 📄 License

MIT License - See [LICENSE](LICENSE) file

## 📞 Support

- **Issues**: Report bugs on GitHub Issues
- **Discussions**: Ask questions in GitHub Discussions
- **Email**: support@pikingdoms.dev
- **Discord**: [Join our community](#)

## 🔗 Useful Links

- [Pi Developer Portal](https://developers.minepi.com)
- [Pi Network Official](https://minepi.com)
- [Pi SDK Documentation](https://github.com/pi-apps/pi-sdk)
- [Game Design Document](docs/GDD.md)

---

**Built with ❤️ for the Pi Network Community**

Last Updated: June 2026