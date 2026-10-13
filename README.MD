
# Web3 Staking Dashboard Architecture

**Network Deployment:** Polygon (PoS) Mainnet  

---

## 1. Executive Summary

The Web3 Staking Dashboard is a performance-optimized decentralized application (dApp) designed to abstract the high-latency and multi-step complexities of smart contract interactions into a highly responsive, real-time user interface. 

The system architecture balances asynchronous blockchain mutations with performant UI updates by separating concerns into four core modules. The implementation solves two persistent Web3 frontend challenges: **eliminating opaque transaction states** during multi-step contract execution, and **mitigating excessive RPC node overhead** through deterministic data caching and high-efficiency rendering.

---

## 2. System Architecture & Tech Stack

The platform is designed around a headless architecture where the visual presentation layers are entirely insulated from underlying network updates.

# Web3 Staking Dashboard Architecture

**Network Deployment:** Polygon (PoS) Mainnet  

---

## 1. Executive Summary

The Web3 Staking Dashboard is a performance-optimized decentralized application (dApp) designed to abstract the high-latency and multi-step complexities of smart contract interactions into a highly responsive, real-time user interface. 

The system architecture balances asynchronous blockchain mutations with performant UI updates by separating concerns into four core modules. The implementation solves two persistent Web3 frontend challenges: **eliminating opaque transaction states** during multi-step contract execution, and **mitigating excessive RPC node overhead** through deterministic data caching and high-efficiency rendering.

---

## 2. System Architecture & Tech Stack

The platform is designed around a headless architecture where the visual presentation layers are entirely insulated from underlying network updates.



┌────────────────────────────────────────────────────────┐
│                   Presentation Layer                   │
│         StakingPanel.tsx  │  StakingStats.tsx          │
└───────────────────────────┬────────────────────────────┘
(Triggers Actions) │  │ (Consumes Cached Data)
▼  ▼
┌────────────────────────────────────────────────────────┐
│                Asynchronous Hook Bridge                │
│     useStakingActions.ts  │  useStakingViews.ts        │
└───────────────────────────┬────────────────────────────┘
(Executes Mutations)│  │ (Orchestrates Queries)
▼  ▼
┌────────────────────────────────────────────────────────┐
│                    Data & Web3 Layer                    │
│   @tanstack/react-query   │   Thirdweb SDK (v4+)       │
└───────────────────────────┬────────────────────────────┘
│ (JSON-RPC Provider)
▼
Polygon Blockchain Network


* **Frontend Environment:** React 18, TypeScript, and Tailwind CSS.
* **Web3 Integration Layer:** **Thirdweb SDK (v4+)** for managed wallet connections (`useActiveAccount`), contract abstraction, and explicit block confirmations.
* **Server-State Architecture:** **@tanstack/react-query** for memory caching, data invalidation paths, and transaction mutation tracking.
* **Data Formatting:** **Ethers.js (v6)** utilities for cryptographic big-integer unit conversions.

---

## 3. Module Deep-Dive & Source Review

### Module A: Asynchronous Write Engine (`useStakingActions.ts`)
This custom hook serves as the core transaction engine, modeling sequential blockchain interactions into an explicit multi-state finite state machine (`TxState`).

* **Atomic Multi-Step Sequencing:** Depositing non-native ERC-20 tokens requires a two-step transactional chain: modifying a user's spending limit via `approve` on the token contract, and then executing `stakeBITS` on the staking contract. The mutation handler wraps this imperatively, updating the state step-by-step (`approving` → `awaiting_approval` → `executing` → `awaiting_execution`).
* **Cross-Hook Cache Invalidation:** Upon receiving a verified block receipt from the Polygon network via `waitForReceipt`, the mutation engine triggers a targeted invalidation of the `["stakingData"]` cache key. This forces a background re-fetch in `useStakingViews`, causing the visual metrics across the entire application to update automatically without requiring manual page refreshes.
* **Memory Safety & Cleanups:** Uses decoupled side-effect lifecycles (`useEffect`) to reset transient transaction flags (`success`, `error`) back to `idle` after a designated cooldown window. This protects the UI from displaying stale status states.

### Module B: Read-Optimized Data Pipeline (`useStakingViews.ts`)
This custom hook serves as the telemetry pipeline, gathering global contract records and contextual user wallet configurations.

* **Identity Preservation:** Instantiates the Thirdweb contract context using React's `useMemo`. This preserves structural reference identity across render frames, shielding the application from duplicate contract setup overhead.
* **Conditional RPC Execution:** Batches parallel calls to retrieve global contract data (`totalStaked`) and user metrics (`totalStakedOf`, `totalRewardOf`). If no active user wallet is detected, it skips user-specific contract reads, cutting empty network requests in half.
* **BigInt Safety & Serialization:** Handles high-precision blockchain integers (`uint256` / `bigint`) securely by enforcing safe math boundaries. It passes fallback big integers (`0n`) to formatting utility methods to protect against application runtime crashes during periods of node latency.
* **Deterministic Client-Side Tiers:** Runs local client-side business logic (`calculateBitPower`) to dynamically map an account's staking weight into Gamified Governance Multipliers instantly upon data arrival, avoiding additional server or gas overhead.

### Module C: UI Presentation Controller (`StakingPanel.tsx`)
This component acts as the input controller, managing local view toggles and parsing human interaction limits.

* **Defensive Boundary Validation:** Prior to invoking expensive Web3 RPC calls, inputs are checked against clear client-side criteria (null values, non-numeric strings, and numbers ≤ 0). This blocks invalid transactions before they reach the wallet, saving user gas fees and preventing redundant network errors.
* **Granular Lifecycle Mapping:** Translates complex async execution states from the write engine into clear text markers on the primary button (e.g., "Confirming approval...", "Claiming rewards..."), keeping the user informed throughout the transaction lifespan.

### Module D: High-Efficiency Data Display (`StakingStats.tsx`)
This pure structural component organizes data streams into a readable metrics dashboard.

* **Render Memoization:** Wrapped completely in React’s `memo` higher-order component. It bypasses expensive virtual DOM re-computations unless its descriptive statistics change, insulating layout frames from ongoing user inputs in neighboring panels.
* **Graceful Degradation Design:** Combines safe-fallback symbols (`|| "—"`) with nullish coalescing constraints (`?? 0`) to handle data gracefully if network latency occurs or user wallets disconnect.

---

## 4. Key Engineering Strengths Demonstrated

* **Advanced Clean Architecture:** Code separation is strictly maintained. The codebase splits asynchronous Web3 read models (`useStakingViews`), mutations (`useStakingActions`), user forms (`StakingPanel`), and presentation metrics (`StakingStats`) into highly modular, reusable engineering units.
* **Granular Blockchain Telemetry:** Replaces generic loading screens with descriptive transaction updates that walk users through approval, execution, and confirmation phases.
* **Strict Runtime Type Safety:** Extensively relies on rigid TypeScript typings to prevent runtime bugs. This enforces correct formatting across both contract ABI interfaces and local application UI configurations'
