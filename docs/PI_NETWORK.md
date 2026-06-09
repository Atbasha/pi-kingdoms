# Pi Network Integration Guide

## 2026 Compliance Overview

Pi Kingdoms follows the latest Pi Network requirements for game developers as outlined by CiDi Games and the Pi Network ecosystem.

## Key Requirements

### 1. Browser-Based Delivery
✅ **REQUIREMENT**: Games must be HTML5 browser-based with no app downloads

**Implementation**:
- Frontend built with React (transpiles to HTML5)
- Hosted on web servers with HTTPS
- Responsive design works on mobile browsers
- No native app required (can add PWA later for app-like experience)

**Files**:
- `frontend/public/index.html` - Entry point
- `frontend/src/App.tsx` - React root component

### 2. Pi Wallet Integration
✅ **REQUIREMENT**: Use official Pi SDK for authentication and payments

**Implementation**:

```typescript
// frontend/src/services/piWalletService.ts
import Pi from 'pi';

const piConfig = {
  version: '2.0',
  appId: process.env.REACT_APP_PI_API_KEY,
};

export const initializePiWallet = async () => {
  try {
    await Pi.init(piConfig);
    console.log('Pi SDK initialized');
  } catch (error) {
    console.error('Failed to initialize Pi SDK:', error);
  }
};

// Authenticate user via Pi Wallet
export const authenticateWithPi = async () => {
  const scopes = ['wallet', 'username'];
  const auth = await Pi.authenticate(scopes, onIncompletePaymentFound);
  return auth;
};

// Create A2U Payment (App-to-User)
export const createPayment = async (
  userId: string,
  amount: number,
  currency: 'pi' | 'pi_usd' | 'open_usd' | 'pi_eur'
) => {
  const payment = await Pi.createPayment({
    amount: amount,
    memo: `Pi Kingdoms - Reward for user ${userId}`,
    metadata: { userId, currency, timestamp: Date.now() },
  });
  return payment;
};
```

**Key Features**:
- Non-custodial: Users control their own keys
- Official SDK only: No third-party wallets
- Transaction validation server-side
- Payment flow follows App-to-User (A2U) protocol

### 3. KYC Requirements for Mainnet
✅ **REQUIREMENT**: Mainnet Pi rewards require completed KYC verification

**Implementation**:

```typescript
// backend/src/services/kycService.ts
export interface UserKYCStatus {
  userId: string;
  kycVerified: boolean;
  kycLevel: 'unverified' | 'level1' | 'level2' | 'level3';
  verifiedAt?: Date;
  allowMainnetTransactions: boolean;
}

// Check KYC before allowing withdrawals
export const canWithdrawPi = async (userId: string): Promise<boolean> => {
  const user = await User.findById(userId);
  return user.kycVerified && user.allowMainnetTransactions;
};

// Testnet rewards (no KYC required)
export const rewardTestnetPi = async (userId: string, amount: number) => {
  // Testnet transaction - no KYC check needed
  const transaction = new Transaction({
    userId,
    amount,
    type: 'testnet_reward',
    status: 'completed',
  });
  await transaction.save();
};

// Mainnet rewards (KYC required)
export const rewardMainnetPi = async (userId: string, amount: number) => {
  const kyc = await KYCVerification.findOne({ userId });
  
  if (!kyc || !kyc.verified) {
    throw new Error('KYC verification required for mainnet transactions');
  }
  
  // Process mainnet transaction via Pi SDK
  const payment = await createPayment(userId, amount, 'pi');
  return payment;
};
```

**User Flow**:
1. User completes KYC in Pi App (not your game)
2. Check KYC status via Pi API
3. Only then allow Mainnet Pi withdrawals
4. Testnet rewards available immediately

### 4. Multi-Currency Support
✅ **REQUIREMENT**: Accept Pure Pi and stablecoins

**Supported Currencies**:
- **Pure Pi** - The main cryptocurrency
- **PI USD** - Pi-backed stablecoin
- **OPEN USD** - OpenDollar stablecoin
- **PI EUR** - Euro-pegged stablecoin

**Implementation**:

```typescript
// backend/src/types/currency.ts
export enum SupportedCurrency {
  PURE_PI = 'pi',
  PI_USD = 'pi_usd',
  OPEN_USD = 'open_usd',
  PI_EUR = 'pi_eur',
}

// backend/src/services/marketplaceService.ts
export const convertPriceForCurrency = (
  priceInPi: number,
  targetCurrency: SupportedCurrency,
  exchangeRates: Record<SupportedCurrency, number>
): number => {
  if (targetCurrency === SupportedCurrency.PURE_PI) {
    return priceInPi;
  }
  return priceInPi * (exchangeRates[targetCurrency] || 1);
};
```

**Display Requirements**:
- Always show Pi equivalent value
- Clearly label stablecoin prices
- Real-time exchange rates
- Transparent pricing for all transactions

### 5. SDK and Protocol Upgrades
✅ **REQUIREMENT**: Use CiDi Games SDK and keep Protocol 23+ updated

**Installation**:

```bash
# In frontend/package.json
npm install @cidi-games/pi-sdk@latest

# In backend/package.json
npm install pi-sdk-node@latest
```

**Version Checking**:

```typescript
// backend/src/middleware/protocolCheck.ts
export const checkProtocolVersion = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const nodeVersion = await getProtocolVersion();
  
  if (nodeVersion < 23) {
    return res.status(503).json({
      error: 'Protocol upgrade required',
      currentVersion: nodeVersion,
      requiredVersion: 23,
    });
  }
  
  next();
};
```

### 6. Compliance Checklist

Before deployment, verify:

- [ ] HTML5/React frontend (browser-based only)
- [ ] Pi SDK v2.0+ integrated for payments
- [ ] Authentication via Pi Wallet (non-custodial)
- [ ] KYC check before mainnet withdrawals
- [ ] Multi-currency pricing implemented
- [ ] Rate limiting on reward claims
- [ ] Server-side transaction validation
- [ ] Privacy policy compliant with GDPR
- [ ] Admin audit log for all transactions
- [ ] No fake/inflated reward claims
- [ ] Branding follows Pi Network guidelines
- [ ] Documentation updated for Pi compliance

## Payment Flow (Detailed)

### A2U Payment Process

1. **User initiates action** (completes quest, wins leaderboard)
2. **Backend calculates reward** with KYC check
3. **Backend creates payment** via Pi SDK
4. **Pi Network processes** the blockchain transaction
5. **Frontend confirms** receipt to user
6. **Database logs** the transaction

**Code Example**:

```typescript
// backend/src/controllers/questController.ts
export const completeQuest = async (req: Request, res: Response) => {
  const { questId } = req.body;
  const userId = req.user.id;

  // Verify quest
  const quest = await Quest.findById(questId);
  if (!quest) return res.status(404).json({ error: 'Quest not found' });

  // Check if already completed
  const completion = await QuestCompletion.findOne({ userId, questId });
  if (completion) {
    return res.status(400).json({ error: 'Quest already completed' });
  }

  // Calculate reward
  const reward = quest.rewardAmount; // e.g., 50 Pi
  
  // Check user KYC for mainnet
  const canRewardMainnet = await canWithdrawPi(userId);
  
  // Create payment (testnet if KYC not verified)
  const network = canRewardMainnet ? 'mainnet' : 'testnet';
  const payment = await createPayment(userId, reward, 'pi');

  // Log transaction
  const txn = new Transaction({
    userId,
    questId,
    amount: reward,
    currency: 'pi',
    type: 'quest_reward',
    network,
    piTransactionId: payment.identifier,
    status: 'pending',
  });
  await txn.save();

  // Mark quest as completed
  const questComp = new QuestCompletion({
    userId,
    questId,
    completedAt: new Date(),
  });
  await questComp.save();

  res.json({
    success: true,
    reward,
    payment,
    message: `Received ${reward} Pi for completing ${quest.name}!`,
  });
};
```

## Testing with Pi Network

### 1. Testnet Testing

```bash
# backend/.env
PI_DEVELOPMENT=true
PI_WALLET_SERVER_URL=https://api.testnet.minepi.com
```

- Request testnet Pi from faucet
- Test payment flows without real money
- Verify anti-cheat systems

### 2. Mainnet Deployment

```bash
# backend/.env (production)
PI_DEVELOPMENT=false
PI_WALLET_SERVER_URL=https://api.mainnet.minepi.com
```

- Enable KYC checks
- Real Pi transactions
- Full audit logging

## Anti-Cheat and Rate Limiting

**Rate Limiting Example**:

```typescript
// backend/src/middleware/rateLimiter.ts
import rateLimit from 'express-rate-limit';

// Limit quest completions to 10 per hour per user
export const questLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 10, // 10 requests per hour
  keyGenerator: (req) => req.user.id,
  message: 'Too many quest completions, try again later',
});

// Use in routes
router.post('/quests/:id/complete', questLimiter, completeQuest);
```

**Anti-Cheat Validation**:

```typescript
// backend/src/services/antiCheatService.ts
export const validateAction = async (
  userId: string,
  action: string,
  metadata: any
): Promise<boolean> => {
  const user = await User.findById(userId);
  
  // Check if action is physically possible
  if (action === 'complete_adventure' && metadata.timeToComplete < 30) {
    return false; // Impossible to complete in <30s
  }
  
  // Check if user level allows this
  if (action === 'enter_chapter' && user.level < metadata.requiredLevel) {
    return false;
  }
  
  // Suspicious pattern detection
  const recentActions = await getRecentActions(userId, 1);
  if (recentActions.length > 100) {
    return false; // More than 100 actions in 1 minute
  }
  
  return true;
};
```

## Admin Audit Logging

```typescript
// backend/src/models/AuditLog.ts
interface AuditLog {
  adminId: string;
  action: string; // 'create_reward', 'ban_user', 'modify_event', etc.
  targetId?: string; // User or event ID
  changes: Record<string, any>;
  timestamp: Date;
  ipAddress: string;
  userAgent: string;
}

// Log all admin actions
export const logAdminAction = async (
  adminId: string,
  action: string,
  targetId: string,
  changes: any,
  req: Request
) => {
  const log = new AuditLog({
    adminId,
    action,
    targetId,
    changes,
    timestamp: new Date(),
    ipAddress: req.ip,
    userAgent: req.get('user-agent'),
  });
  await log.save();
};
```

## Resources

- [Pi Developer Portal](https://developers.minepi.com)
- [Pi SDK Documentation](https://github.com/pi-apps/pi-sdk)
- [CiDi Games Roadmap](https://cidi.games)
- [Pi Network Official](https://minepi.com)

## Questions?

For integration questions:
1. Check [Pi Developer Guide](https://pi-apps.github.io/community-developer-guide/)
2. Review [API.md](./API.md) for endpoints
3. Ask in Pi Developer Discord
4. File an issue on GitHub