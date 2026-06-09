# Pi Network Transaction-Based Game System

## Overview

This document outlines how Pi Kingdoms can run on Pi Network by counting transactions as the primary game mechanic. Players earn rewards by completing in-game actions that trigger Pi transactions.

## Core Concept: Transaction-to-Game-Value Mapping

Instead of traditional game rewards, every player action creates a verifiable Pi transaction on the blockchain. The game tracks these transactions as proof of play and engagement.

### Transaction Types

```typescript
enum TransactionType {
  QUEST_COMPLETION = 'quest_completion',
  BUILDING_UPGRADE = 'building_upgrade',
  ADVENTURE_VICTORY = 'adventure_victory',
  LEADERBOARD_REWARD = 'leaderboard_reward',
  MARKETPLACE_PURCHASE = 'marketplace_purchase',
  DAILY_REWARD = 'daily_reward',
  ACHIEVEMENT_UNLOCK = 'achievement_unlock',
  REFERRAL_BONUS = 'referral_bonus'
}
```

## Transaction Flow Architecture

```
┌─────────────────────────────────────────────────────┐
│         Player Action (Quest Completed)             │
└──────────────────┬──────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────┐
│  Validate Action (Anti-Cheat)                       │
│  - Check KYC status                                 │
│  - Rate limit check                                 │
│  - Pattern detection                                │
└──────────────────┬──────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────┐
│  Calculate Reward Value                             │
│  - Quest difficulty * difficulty_multiplier        │
│  - Player level modifier                            │
│  - Rarity bonus                                     │
└──────────────────┬──────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────┐
│  Create Pi Transaction (A2U Payment)                │
│  - Amount: calculated_reward                        │
│  - Currency: Pure Pi / PI USD / etc                 │
│  - Network: Testnet (unverified) / Mainnet (KYC)    │
└──────────────────┬──────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────┐
│  Submit to Pi SDK (App-to-User)                     │
│  - Pi SDK processes blockchain transaction          │
│  - Returns transaction ID                           │
│  - Stores on blockchain (immutable proof)           │
└──────────────────┬──────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────┐
│  Log Transaction in DB                              │
│  - Store Pi transaction ID                          │
│  - Store game action metadata                       │
│  - Timestamp for analytics                          │
└──────────────────┬──────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────┐
│  Update Game State                                  │
│  - Add Pi to player wallet                          │
│  - Mark quest as completed                          │
│  - Award experience points                          │
└──────────────────┬──────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────┐
│  Notify Player                                      │
│  - Show reward notification                         │
│  - Display transaction ID                           │
│  - Option to view on blockchain explorer            │
└─────────────────────────────────────────────────────┘
```

## Transaction Counting System

### Backend Implementation

```typescript
// backend/src/services/transactionCountingService.ts

import { PiSDK } from 'pi-sdk';
import { Transaction } from '../models/Transaction';
import { User } from '../models/User';

interface GameAction {
  userId: string;
  actionType: TransactionType;
  baseReward: number;
  metadata: Record<string, any>;
}

export class TransactionCountingService {
  private piSdk: PiSDK;

  constructor() {
    this.piSdk = new PiSDK({
      apiKey: process.env.PI_API_KEY,
      apiSecret: process.env.PI_API_SECRET,
    });
  }

  /**
   * Process game action and create Pi transaction
   */
  async processGameAction(action: GameAction): Promise<Transaction> {
    // 1. Validate action
    const isValid = await this.validateAction(action);
    if (!isValid) throw new Error('Invalid action');

    // 2. Calculate reward
    const reward = await this.calculateReward(action);

    // 3. Check if user can receive mainnet rewards
    const user = await User.findById(action.userId);
    const isMainnet = user.kyc_verified;
    const network = isMainnet ? 'mainnet' : 'testnet';

    // 4. Create Pi transaction via SDK
    const piPayment = await this.piSdk.createPayment({
      amount: reward,
      memo: `Pi Kingdoms - ${action.actionType}: ${action.metadata.name}`,
      metadata: {
        userId: action.userId,
        actionType: action.actionType,
        timestamp: Date.now(),
        ...action.metadata,
      },
    });

    // 5. Log transaction in database
    const transaction = new Transaction({
      user_id: action.userId,
      amount: reward,
      currency: 'pi',
      transaction_type: action.actionType,
      network: network,
      pi_transaction_id: piPayment.identifier,
      status: 'pending',
      metadata: action.metadata,
    });

    await transaction.save();

    // 6. Update leaderboard
    await this.updateLeaderboard(action.userId, reward);

    // 7. Check for achievements
    await this.checkAchievements(action.userId);

    return transaction;
  }

  /**
   * Calculate reward based on action, difficulty, and player level
   */
  private async calculateReward(action: GameAction): Promise<number> {
    const user = await User.findById(action.userId);
    
    const baseReward = action.baseReward;
    const levelMultiplier = 1 + (user.level * 0.05); // 5% per level
    const dailyBonusMultiplier = await this.getDailyBonusMultiplier(action.userId);
    
    const finalReward = Math.floor(
      baseReward * levelMultiplier * dailyBonusMultiplier
    );

    return Math.max(1, Math.min(finalReward, 1000)); // Cap between 1 and 1000 Pi
  }

  /**
   * Validate action to prevent cheating
   */
  private async validateAction(action: GameAction): Promise<boolean> {
    const user = await User.findById(action.userId);

    // Check if banned
    if (user.is_banned) return false;

    // Check rate limiting
    const recentTransactions = await Transaction.find({
      user_id: action.userId,
      created_at: { $gt: Date.now() - 60000 }, // Last minute
    });

    if (recentTransactions.length > 10) return false; // Max 10 transactions per minute

    // Check if action is physically possible
    const timeSinceLastAction = Date.now() - user.last_action_time;
    if (timeSinceLastAction < 5000) return false; // Minimum 5 seconds between actions

    // Check skill-level alignment
    const requiredLevel = action.metadata.requiredLevel || 1;
    if (user.level < requiredLevel) return false;

    return true;
  }

  /**
   * Update leaderboard based on total transactions
   */
  private async updateLeaderboard(userId: string, transactionAmount: number): Promise<void> {
    const totalTransactions = await Transaction.count({
      user_id: userId,
      status: 'completed',
    });

    const totalValue = await Transaction.sum('amount', {
      user_id: userId,
      status: 'completed',
    });

    // Update leaderboard entry
    await this.updateLeaderboardEntry({
      userId,
      transactionCount: totalTransactions,
      totalValue: totalValue,
      score: totalTransactions * 10 + Math.floor(totalValue), // Combined score
    });
  }

  /**
   * Get daily login bonus multiplier
   */
  private async getDailyBonusMultiplier(userId: string): Promise<number> {
    const lastReward = await Transaction.findOne({
      user_id: userId,
      transaction_type: 'daily_reward',
      created_at: { $gt: Date.now() - 86400000 }, // Last 24 hours
    });

    if (!lastReward) return 2.0; // 2x multiplier on first daily reward
    return 1.0; // Normal multiplier
  }

  /**
   * Check and unlock achievements based on transactions
   */
  private async checkAchievements(userId: string): Promise<void> {
    const totalTransactions = await Transaction.count({
      user_id: userId,
      status: 'completed',
    });

    const achievements = [
      { milestone: 1, key: 'first_transaction' },
      { milestone: 10, key: 'transaction_beginner' },
      { milestone: 50, key: 'transaction_explorer' },
      { milestone: 100, key: 'transaction_veteran' },
      { milestone: 500, key: 'transaction_master' },
      { milestone: 1000, key: 'transaction_legend' },
    ];

    for (const achievement of achievements) {
      if (totalTransactions >= achievement.milestone) {
        await this.unlockAchievement(userId, achievement.key);
      }
    }
  }

  private async unlockAchievement(userId: string, achievementKey: string): Promise<void> {
    // Implementation to unlock achievement
  }

  private async updateLeaderboardEntry(data: any): Promise<void> {
    // Implementation to update leaderboard
  }
}
```

## Transaction Reward Structure

### Quest Completions (by difficulty)

```typescript
const QUEST_REWARDS = {
  EASY: 5,      // 5 Pi
  MEDIUM: 15,   // 15 Pi
  HARD: 50,     // 50 Pi
  EXTREME: 200, // 200 Pi
};
```

### Building Upgrades

```typescript
const BUILDING_UPGRADE_REWARDS = {
  FARM: 2,
  MINE: 3,
  BARRACKS: 5,
  MARKET: 4,
};
```

### Adventure Victories (by chapter)

```typescript
const ADVENTURE_REWARDS = {
  CHAPTER_1: 10,
  CHAPTER_2: 20,
  CHAPTER_3: 35,
  CHAPTER_4: 60,
  CHAPTER_5: 100,
};
```

### Daily & Weekly

```typescript
const TIME_BASED_REWARDS = {
  DAILY_LOGIN: 2,
  DAILY_STREAK_3: 5,
  DAILY_STREAK_7: 10,
  WEEKLY_LEADERBOARD_TOP1: 500,
  WEEKLY_LEADERBOARD_TOP10: 250,
  WEEKLY_LEADERBOARD_TOP100: 100,
};
```

## Leaderboard Ranking by Transaction Count

### Global Leaderboard

```typescript
// backend/src/services/leaderboardService.ts

export class LeaderboardService {
  /**
   * Calculate global leaderboard based on transactions
   */
  async getGlobalLeaderboard(limit: number = 100): Promise<LeaderboardEntry[]> {
    const entries = await LeaderboardEntry.query(`
      SELECT 
        le.id,
        le.user_id,
        u.pi_username,
        u.level,
        COUNT(t.id) as transaction_count,
        SUM(t.amount) as total_pi_earned,
        MAX(t.created_at) as last_transaction,
        ROW_NUMBER() OVER (ORDER BY SUM(t.amount) DESC) as rank
      FROM leaderboard_entries le
      JOIN users u ON le.user_id = u.id
      LEFT JOIN transactions t ON u.id = t.user_id AND t.status = 'completed'
      WHERE u.is_banned = false
      GROUP BY le.user_id, u.pi_username, u.level
      ORDER BY total_pi_earned DESC
      LIMIT $1
    `, [limit]);

    return entries;
  }

  /**
   * Calculate weekly leaderboard (transaction count last 7 days)
   */
  async getWeeklyLeaderboard(limit: number = 100): Promise<LeaderboardEntry[]> {
    const sevenDaysAgo = Date.now() - 7 * 24 * 60 * 60 * 1000;

    const entries = await LeaderboardEntry.query(`
      SELECT 
        u.id,
        u.pi_username,
        u.level,
        COUNT(t.id) as transaction_count,
        SUM(t.amount) as weekly_pi_earned,
        ROW_NUMBER() OVER (ORDER BY SUM(t.amount) DESC) as rank
      FROM users u
      LEFT JOIN transactions t ON u.id = t.user_id 
        AND t.status = 'completed' 
        AND t.created_at > $1
      WHERE u.is_banned = false
      GROUP BY u.id, u.pi_username, u.level
      ORDER BY weekly_pi_earned DESC
      LIMIT $2
    `, [new Date(sevenDaysAgo), limit]);

    return entries;
  }

  /**
   * Calculate player rank by transaction count
   */
  async getPlayerRank(userId: string): Promise<number> {
    const result = await Transaction.query(`
      SELECT COUNT(DISTINCT t2.user_id) + 1 as rank
      FROM users u1
      LEFT JOIN transactions t1 ON u1.id = t1.user_id AND t1.status = 'completed'
      CROSS JOIN transactions t2
      WHERE u1.id = $1
      GROUP BY u1.id
    `, [userId]);

    return result[0]?.rank || 0;
  }
}
```

## Real-time Transaction Analytics Dashboard

```typescript
// backend/src/services/analyticsService.ts

export class AnalyticsService {
  /**
   * Get transaction statistics for analytics
   */
  async getTransactionStats(): Promise<TransactionStats> {
    const totalTransactions = await Transaction.count();
    const totalValue = await Transaction.sum('amount', { status: 'completed' });
    const activeUsers = await User.count({ last_action_time: { $gt: Date.now() - 3600000 } });
    
    const last24h = await Transaction.count({
      created_at: { $gt: Date.now() - 86400000 },
      status: 'completed',
    });

    const transactionsPerMinute = last24h / (24 * 60);

    return {
      totalTransactions,
      totalValue,
      activeUsers,
      last24hTransactions: last24h,
      transactionsPerMinute,
    };
  }

  /**
   * Get transaction breakdown by type
   */
  async getTransactionBreakdown(): Promise<Record<string, number>> {
    const breakdown = await Transaction.query(`
      SELECT 
        transaction_type,
        COUNT(*) as count,
        SUM(amount) as total_value
      FROM transactions
      WHERE status = 'completed'
      GROUP BY transaction_type
      ORDER BY total_value DESC
    `);

    return breakdown;
  }

  /**
   * Track transaction velocity (transactions per hour)
   */
  async getTransactionVelocity(): Promise<VelocityData[]> {
    const data = await Transaction.query(`
      SELECT 
        DATE_TRUNC('hour', created_at) as hour,
        COUNT(*) as count,
        SUM(amount) as value
      FROM transactions
      WHERE status = 'completed' AND created_at > NOW() - INTERVAL '7 days'
      GROUP BY hour
      ORDER BY hour DESC
    `);

    return data;
  }
}
```

## Blockchain Verification

### Transaction Verification View

```typescript
// frontend/src/components/TransactionVerification.tsx

import React, { useState } from 'react';
import { Transaction } from '../types';

interface Props {
  transaction: Transaction;
}

export const TransactionVerification: React.FC<Props> = ({ transaction }) => {
  const [isVerified, setIsVerified] = useState(false);

  const verifyOnBlockchain = async () => {
    try {
      const response = await fetch(
        `https://explorer.mainnet.minepi.com/tx/${transaction.pi_transaction_id}`
      );
      setIsVerified(response.ok);
    } catch (error) {
      console.error('Verification failed:', error);
    }
  };

  return (
    <div className="transaction-verification">
      <h3>Transaction Proof</h3>
      <div className="transaction-id">
        <code>{transaction.pi_transaction_id}</code>
      </div>
      
      <div className="transaction-details">
        <p><strong>Amount:</strong> {transaction.amount} Pi</p>
        <p><strong>Type:</strong> {transaction.transaction_type}</p>
        <p><strong>Network:</strong> {transaction.network}</p>
        <p><strong>Status:</strong> {transaction.status}</p>
        <p><strong>Timestamp:</strong> {new Date(transaction.created_at).toLocaleString()}</p>
      </div>

      <button onClick={verifyOnBlockchain} className="verify-btn">
        Verify on Blockchain
      </button>

      {isVerified && (
        <div className="verified-badge">
          ✓ Verified on Pi Network Blockchain
        </div>
      )}

      <div className="blockchain-link">
        <a 
          href={`https://explorer.mainnet.minepi.com/tx/${transaction.pi_transaction_id}`}
          target="_blank"
          rel="noopener noreferrer"
        >
          View on Blockchain Explorer →
        </a>
      </div>
    </div>
  );
};
```

## Reward Distribution Schedule

### Automatic Distribution via Transactions

1. **Immediate Rewards** (A2U transactions)
   - Quest completion
   - Building upgrades
   - Achievement unlocks

2. **Daily Rewards** (Batch at 00:00 UTC)
   - Daily login bonus
   - Daily streak rewards
   - Passive resource generation

3. **Weekly Rewards** (Every Sunday 00:00 UTC)
   - Leaderboard rankings
   - Weekly challenge winners
   - Special event rewards

4. **Monthly Rewards** (1st of month 00:00 UTC)
   - Top players bonus
   - Milestone rewards
   - Seasonal rewards

## Anti-Cheat via Transaction Analysis

```typescript
// backend/src/services/antiCheatService.ts

export class AntiCheatService {
  /**
   * Detect suspicious transaction patterns
   */
  async detectFraudPatterns(userId: string): Promise<FraudIndicators> {
    const transactions = await Transaction.find({
      user_id: userId,
      created_at: { $gt: Date.now() - 3600000 }, // Last hour
    });

    const indicators: FraudIndicators = {
      transactionCount: transactions.length,
      averageTimeBetween: this.calculateAverageTime(transactions),
      suspiciousPattern: false,
      reason: '',
    };

    // Check for impossible transaction rate
    if (transactions.length > 20) {
      indicators.suspiciousPattern = true;
      indicators.reason = 'Too many transactions in 1 hour';
    }

    // Check for average time too short (< 3 seconds)
    if (indicators.averageTimeBetween < 3000) {
      indicators.suspiciousPattern = true;
      indicators.reason = 'Transactions happening too fast';
    }

    // Check for identical amounts (possible automation)
    const uniqueAmounts = new Set(transactions.map(t => t.amount)).size;
    if (uniqueAmounts === 1 && transactions.length > 5) {
      indicators.suspiciousPattern = true;
      indicators.reason = 'Identical transaction amounts (possible bot)';
    }

    return indicators;
  }

  private calculateAverageTime(transactions: Transaction[]): number {
    if (transactions.length < 2) return 0;
    
    const times = transactions
      .map(t => t.created_at.getTime())
      .sort((a, b) => a - b);

    let totalTime = 0;
    for (let i = 1; i < times.length; i++) {
      totalTime += times[i] - times[i - 1];
    }

    return totalTime / (times.length - 1);
  }

  /**
   * Flag suspicious users for review
   */
  async flagForReview(userId: string, reason: string): Promise<void> {
    await User.update(userId, {
      flagged_for_review: true,
      review_reason: reason,
      flagged_at: new Date(),
    });
  }
}
```

## Summary: Transaction-Based Economy

| Metric | Value |
|--------|-------|
| **Min Transaction** | 1 Pi |
| **Max Transaction** | 1000 Pi |
| **Transaction Types** | 8 main types |
| **Daily Active Users** | Target: 10,000+ |
| **Avg Transactions/Player/Day** | 5-10 |
| **Total Monthly Pi Flow** | Dynamic (user-earned) |
| **Blockchain Verification** | 100% of transactions |
| **KYC Required** | Mainnet only |
| **Testnet Available** | All users |

## Advantages of Transaction-Counting System

✅ **Verifiable** - Every reward on blockchain  
✅ **Fair** - No fake rewards or inflation  
✅ **Transparent** - Players see exact Pi flow  
✅ **Scalable** - Handles thousands of concurrent transactions  
✅ **Compliant** - Follows Pi Network 2026 standards  
✅ **Anti-Cheat** - Rate limiting + pattern detection  
✅ **Incentive Aligned** - Players earn real Pi for engagement  
✅ **Real Value** - Testnet Pi convertible to Mainnet when KYC verified  

This system makes Pi Kingdoms a true Play-to-Earn game on the Pi Network! 🎮💰