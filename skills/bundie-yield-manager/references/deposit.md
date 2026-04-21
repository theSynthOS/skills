# Deposit & Withdraw Workflows

## Depositing into Bundie

### Step 1: Deposit to Vault

Use `vault_deposit` to move funds from the user's wallet into their Bundie vault on Scroll.

```
deposit(walletAddress, asset="USDC", amount="500")
```

This deposits funds into the vault but doesn't allocate to any yield strategy yet.

### Step 2: Allocate to a Strategy

After depositing, allocate to a specific yield strategy using `strategy_deposit`.

```
strategy_deposit(walletAddress, protocolId="<uuid>", amount="500")
```

The `protocolId` comes from `yields_check` results.

### Full Deposit Flow (Recommended)

For the best experience, combine with AI recommendations:

1. `wallet_recommend(walletAddress)` — get AI-optimized allocation
2. Confirm the bundle with the user
3. `deposit(walletAddress, asset="USDC", amount="500")` — vault deposit
4. For each allocation in the recommendation:
   - `strategy_deposit(walletAddress, protocolId, amount)` — deploy to strategy

### Important Notes

- Always confirm the amount and asset with the user before executing
- Cross-chain strategy deposits (e.g., to Base or Arbitrum) take 5-15 minutes via LayerZero
- The transaction response includes explorer and LayerZero tracking links
- If deposit fails, check that the user has sufficient balance and has approved the token

## Withdrawing from Bundie

### Withdraw from Strategy Position

Use `strategy_withdraw` to pull funds from an active strategy.

```
strategy_withdraw(walletAddress, positionIndex=0, amount="250", destinationChain=534352)
```

- `positionIndex` comes from `portfolio_view` results
- `destinationChain` is where to receive the funds (Scroll=534352, Base=8453, etc.)

### Withdraw from Vault

After strategy withdrawal completes, use `vault_withdraw` to move funds back to the user's wallet.

```
withdraw(walletAddress, asset="USDC", amount="250")
```

### Withdraw to Different Address

```
withdraw(walletAddress, asset="USDC", amount="250", recipientAddress="0x...")
```

### Full Withdrawal Flow

1. `portfolio(walletAddress)` — see current positions
2. `strategy_withdraw(walletAddress, positionIndex, amount, destinationChain)` — pull from strategy
3. Wait for cross-chain settlement if applicable (5-15 min)
4. `withdraw(walletAddress, asset, amount)` — move to wallet

## Edge Cases

- **Partial withdrawals:** User can withdraw part of a position
- **Cross-chain delays:** Always mention the 5-15 min settlement time for cross-chain ops
- **Insufficient balance:** The backend will return an error — surface it clearly
- **Gas fees:** Cross-chain deposits require ETH for LayerZero bridge fees (shown in response)
