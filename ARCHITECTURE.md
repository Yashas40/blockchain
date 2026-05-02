# 🔗 ClearLedger — Project Architecture & Technical Documentation

> **ClearLedger** is a blockchain-powered government fund tracking system that creates an immutable, transparent audit trail for every rupee of public money disbursed across Indian government schemes.

---

## 📌 Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [Solution Overview](#2-solution-overview)
3. [Tech Stack](#3-tech-stack)
4. [System Architecture](#4-system-architecture)
5. [User Roles & Access Control](#5-user-roles--access-control)
6. [Full Application Workflow](#6-full-application-workflow)
7. [Smart Contract Design](#7-smart-contract-design)
8. [IPFS Proof System](#8-ipfs-proof-system)
9. [AI Anomaly Detection](#9-ai-anomaly-detection)
10. [Geographic Fund Map](#10-geographic-fund-map)
11. [Data Sources](#11-data-sources)
12. [File & Folder Structure](#12-file--folder-structure)
13. [Key Design Decisions](#13-key-design-decisions)
14. [Security Model](#14-security-model)

---

## 1. Problem Statement

India loses an estimated **₹1.87 lakh crore annually** to fund leakage in government welfare schemes (CAG Report 2023-24). The root causes are:

| Problem | Traditional System (PFMS) |
|---|---|
| **Data Tampering** | Single mutable database — any official can alter records |
| **No Audit Trail** | Paper receipts can be fabricated or lost |
| **Zero Transparency** | Citizens have no real-time visibility into fund status |
| **Garbage In, Garbage Out** | Funds released without documentary proof of delivery |
| **No Anomaly Detection** | Night transfers, duplicate payments go unnoticed |

---

## 2. Solution Overview

ClearLedger replaces the paper-and-PFMS system with a **three-layer stack**:

```
┌─────────────────────────────────────────────────────────────┐
│  LAYER 3 — AI LAYER                                         │
│  Pre-submission risk scoring (Groq LLaMA 3)                 │
│  Pattern detection: night transfers, round amounts, dupes   │
├─────────────────────────────────────────────────────────────┤
│  LAYER 2 — IPFS EVIDENCE LAYER                              │
│  Receipts & geo-tagged images hashed before funds move      │
│  Immutable content-addressed storage (IPFS / Filecoin)      │
├─────────────────────────────────────────────────────────────┤
│  LAYER 1 — BLOCKCHAIN LAYER                                 │
│  Ethereum Sepolia — smart contract stores every transaction │
│  Multi-sig auditor approval — cryptographic proof           │
└─────────────────────────────────────────────────────────────┘
```

**Core Guarantee:** A fund transfer cannot be initiated without:
1. Documentary proof (receipt / geo-tagged image) → IPFS hash generated
2. AI risk scan passing below threshold
3. Admin digital signature (MetaMask wallet)
4. Auditor cryptographic approval on-chain

---

## 3. Tech Stack

### Frontend
| Technology | Version | Purpose |
|---|---|---|
| **React** | 19.x | Component-based UI framework |
| **Vite** | 8.x | Lightning-fast dev server & bundler |
| **TailwindCSS** | 3.x | Utility-first styling |
| **Framer Motion** | 12.x | Smooth UI animations |
| **React Router DOM** | 7.x | Client-side routing |

### Blockchain & Web3
| Technology | Version | Purpose |
|---|---|---|
| **Ethers.js** | 6.x | Ethereum wallet interaction & contract calls |
| **MetaMask** | Browser Extension | Wallet auth + transaction signing |
| **Solidity** | ^0.8.0 | Smart contract language |
| **Remix IDE** | Web | Contract compilation & deployment |
| **Ethereum Sepolia** | Testnet | Blockchain network (free test ETH) |

### Data & Visualization
| Technology | Version | Purpose |
|---|---|---|
| **Recharts** | 3.x | Fund flow charts & analytics |
| **React-Leaflet** | 5.x | Interactive India fund map |
| **Leaflet** | 1.9.x | Underlying map engine |
| **jsPDF** | 4.x | PDF audit report generation |

### AI / LLM
| Technology | Purpose |
|---|---|
| **Groq API** | Ultra-fast LLM inference (LLaMA 3 8B) |
| **LLaMA 3 8B** | Transaction suspicion scoring & audit summaries |

### Datasets
| Dataset | Source | Size | Purpose |
|---|---|---|---|
| India Cities Geocodes | GeoNames via lutangar/cities.json | 4,425 entries | Village map pin placement |
| Indian Cities List | Custom curated | ~6,100 entries | Fund release autocomplete |
| Government Scheme Data | CAG Report 2023-24, DBT Mission | Static | Dashboard real data |

---

## 4. System Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                        BROWSER (React SPA)                        │
│                                                                    │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────────────────┐  │
│  │  AuthContext │  │ TxContext    │  │  Pages / Components    │  │
│  │  userRole   │  │ transactions │  │  Dashboard, FundRelease│  │
│  │  wallet addr│  │ addLocal()   │  │  AuditorReview, Verify │  │
│  └──────┬──────┘  └──────┬───────┘  │  VillageMap, FlagReport│  │
│         │                │           └────────────────────────┘  │
│         └────────────────┼─────────────────────┐                 │
│                          │                      │                 │
│              ┌───────────▼───────────┐  ┌───────▼──────────┐    │
│              │   src/utils/          │  │  src/data/        │    │
│              │   contract.js         │  │  realData.js      │    │
│              │   aiAnalysis.js       │  │  india_geocodes   │    │
│              │   dataGenerator.js    │  │  indian_cities    │    │
│              └───────────┬───────────┘  └──────────────────┘    │
└──────────────────────────┼───────────────────────────────────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
   ┌──────▼──────┐  ┌──────▼──────┐  ┌─────▼──────────┐
   │  MetaMask   │  │  Groq API   │  │  IPFS Network  │
   │  Wallet     │  │  LLaMA 3    │  │  (Simulated)   │
   └──────┬──────┘  └─────────────┘  └────────────────┘
          │
   ┌──────▼──────────────────────────────┐
   │  Ethereum Sepolia Testnet            │
   │  Smart Contract: ClearLedger.sol    │
   │  ┌────────────────────────────────┐ │
   │  │  addTransaction()              │ │
   │  │  flagTransaction()             │ │
   │  │  addSignature()                │ │
   │  │  getTransaction()              │ │
   │  │  transactionCount()            │ │
   │  └────────────────────────────────┘ │
   └──────────────────────────────────────┘
```

---

## 5. User Roles & Access Control

ClearLedger has **three distinct roles**, each with restricted page access:

```
┌─────────────────────────────────────────────────────────────┐
│  ADMIN (Government Official)                                 │
│  ✓ Fund Release    ✓ Dashboard    ✓ Fund Ledger             │
│  ✗ Auditor Review  ✗ Flag Report                            │
│  Signs transactions with MetaMask wallet                     │
│  Must attach documentary proof (receipt/image) first         │
├─────────────────────────────────────────────────────────────┤
│  AUDITOR (Independent Oversight)                             │
│  ✓ Auditor Review  ✓ Dashboard    ✓ Fund Ledger             │
│  ✓ Flag Report     ✓ Audit Export ✓ Village Map             │
│  Approves or freezes transactions via blockchain signature   │
├─────────────────────────────────────────────────────────────┤
│  CITIZEN (Public Verification)                               │
│  ✓ Verify Portal   ✓ Village Map  (read-only)               │
│  Can search their village and view blockchain proof          │
│  No write access to any transaction                          │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. Full Application Workflow

### Step 1 — Admin Logs In
```
Login page → Select Role (Admin) → Connect MetaMask wallet
→ Auto-switches MetaMask to Sepolia testnet
→ Wallet address stored in AuthContext
```

### Step 2 — Admin Initiates Fund Release
```
Fund Release page:
  1. Fill form: Domain, State, Village, Amount, Purpose, Time
  2. Upload documentary proof (receipt PDF / geo-tagged image)
     → Deterministic IPFS hash generated (Qm...) per file
     → Glowing green IPFS pill displayed on screen
  3. AI Pre-Scan runs automatically:
     → Night transfer check (after 10 PM / before 6 AM)
     → Round number structuring detection
     → Exceeds ₹50L threshold check
     → Duplicate village payment detection
     → Risk score displayed (0–100)
  4. Click "Release Funds & Sign on Blockchain"
     → MetaMask popup appears for signature
     → submitTransaction() called → Ethereum Sepolia tx
     → Transaction stored in context (pending status)
     → IPFS hash embedded in on-chain record
```

### Step 3 — Auditor Reviews
```
Auditor Review page (Live Queue):
  → All pending transactions visible in real-time feed
  → Status strip: Yellow (pending) / Green (approved) / Red (frozen)
  → Select transaction → Detail panel shows:
     - Amount, village, purpose, timestamp
     - IPFS proof document link with glowing green hash
     - AI suspicion score with breakdown
  → Two actions:
     [APPROVE] → addSignature() on-chain → status = approved
     [FLAG]    → flagTransaction() on-chain → status = frozen
```

### Step 4 — Map Updates in Real-Time
```
Village Map page:
  → Merges static baseline data + live transactions from context
  → Pin colours update immediately when auditor acts:
     🟡 Yellow = Admin submitted, auditor pending
     🟢 Green  = Auditor approved
     🔴 Red    = Flagged / frozen
  → 4,425 city geocodes for accurate pin placement
  → Popup shows: amount, status, purpose, IPFS proof hash
```

### Step 5 — Citizen Verifies
```
Verify Portal (public access):
  → Search village name
  → Blockchain read → shows real-time status
  → Displays IPFS proof link ("View IPFS Proof Document")
  → Shows Etherscan link for immutable on-chain proof
  → Three states: Blockchain Confirmed / Awaiting Auditor / Frozen
```

### Step 6 — Audit Export
```
Audit Export page:
  → Select scheme and time range
  → AI generates executive summary paragraph (Groq LLaMA 3)
  → Downloads full PDF report with:
     - All transactions, amounts, statuses
     - IPFS hashes for each document
     - Blockchain TX hashes
     - Risk scores and anomaly flags
```

---

## 7. Smart Contract Design

**File:** `ClearLedger.sol` (deployed on Ethereum Sepolia)

```solidity
contract ClearLedger {
    struct Transaction {
        string  fromEntity;          // State / releasing authority
        string  toEntity;            // Village / Gram Panchayat
        uint256 amount;              // In rupees (raw integer, no decimals)
        string  scheme;              // MGNREGA / Healthcare / Agriculture / Education
        string  ipfsHash;            // IPFS hash of proof document
        uint8   signaturesReceived;  // Auditor signature count
        bool    flagged;             // Frozen by auditor
        uint256 timestamp;           // block.timestamp at submission
    }

    // Key functions:
    addTransaction(from, to, amount, scheme, ipfsHash)  // Admin only
    flagTransaction(id)                                  // Auditor freeze
    addSignature(id)                                     // Auditor approve
    getTransaction(id)                                   // Public read
    transactionCount()                                   // Total count
    flaggedCount()                                       // Flagged count
}
```

**Why `BigInt` not `parseUnits`:**
`ethers.parseUnits(x, 0)` has floating-point precision bugs with large Indian rupee values (e.g., `1200000` → `99808702`). We use `BigInt(Math.round(Number(amount)))` to pass the exact integer.

**Network:** Ethereum Sepolia (Chain ID: `11155111`)

---

## 8. IPFS Proof System

### The "Garbage In, Garbage Out" Problem
Traditional blockchain systems only prove *that* a transaction happened — not *whether the funds actually reached the ground*. An official can enter any data.

### ClearLedger's Fix
Before funds are released, the admin must upload **physical documentary evidence**:
- ✅ Scanned government receipts
- ✅ Geo-tagged photographs from the village site
- ✅ Beneficiary acknowledgement forms
- ✅ Bank disbursement slips

### How It Works
```
File selected by admin
      ↓
Deterministic hash generated:
  seed = fileName + fileSize → Qm[44 chars]
      ↓
ipfsHash stored in:
  - React state (immediate display)
  - addTransaction() call (on-chain)
  - Transaction context (map + auditor access)
      ↓
Displayed as glowing green pill:
  ipfs://QmXyZaBcDeFg...
      ↓
Auditor sees "View IPFS Proof" button
Citizen sees "View IPFS Proof Document" button
```

> **Note:** In production, files would be pinned to a real IPFS node (e.g., Pinata, Web3.Storage, or Filecoin). The hash generation is deterministic for demo purposes.

---

## 9. AI Anomaly Detection

### Pre-Submission Scan (Rule-Based, Instant)
Runs in `FundRelease.jsx` via `calculateAIScore()`:

| Rule | Points Added | Trigger |
|---|---|---|
| Night transfer | +30 | Transfer time between 10 PM – 6 AM |
| Exceeds threshold | +15 | Amount ≥ ₹50,00,000 |
| Round number | +20 | Amount ends in `00000` (structuring pattern) |
| Duplicate village | +30 | Same village received funds within 12 days |
| **Total possible** | **95** | Hard-capped to prevent 100/100 false alarm |

Risk levels: `0–40` = Safe | `41–70` = Medium Risk | `71+` = High Risk

### Post-Submission AI (LLM-Powered, Groq API)
Used in `AuditExport.jsx` via `generateSummaryForPDF()`:
- Model: `llama3-8b-8192`
- Generates a professional 3-4 sentence audit summary
- Uses real transaction data (village, amount, scheme, TX hash)
- Fallback: static template if API unavailable

**API Key:** Set `VITE_GROQ_API_KEY` in `.env` (never hardcode — was removed from codebase)

---

## 10. Geographic Fund Map

### Data Flow
```
Static MAP_VILLAGES (realData.js)        Live Transactions (context)
         +                                         +
         └──────────────────┬────────────────────┘
                            ↓
                   allVillages = [...static, ...live]
                            ↓
              Geocode lookup (3-tier):
              1. Exact city name → india_geocodes.json (4,425 cities)
              2. Prefix match   → "Kolar" → "Kolar Gold Fields"
              3. State centroid → fallback if city not found
                            ↓
                     Leaflet map pins
```

### Pin Colour Logic
```
tx.flagged || tx.status === 'frozen'          → 🔴 Red   (#ef4444)
tx.status === 'approved' || signatures > 0    → 🟢 Green (#22c55e)
else                                          → 🟡 Yellow (#f59e0b)
```

### Dataset
- **Source:** GeoNames database via `github.com/lutangar/cities.json`
- **Filter:** country = `IN`
- **Size:** 4,425 Indian cities/towns with precise lat/lng
- **File:** `src/data/india_geocodes.json` (430 KB)

---

## 11. Data Sources

| Data | Source | Usage |
|---|---|---|
| MGNREGA allocation (₹86,000 Cr) | CAG Report 2023-24 | Dashboard stats |
| Healthcare leakage (₹892 Cr flagged) | CAG Report 2022-23 | Scheme data |
| Agriculture utilization (91.3%) | DBT Mission Report 2023 | Scheme data |
| Education shortfall (61.5%) | CAG Report 2022-23 | Scheme data |
| Village coordinates | GeoNames (4,425 cities) | Map pins |

---

## 12. File & Folder Structure

```
stitch_clearledger_blockchain_fund_tracker/
│
├── public/
│   ├── favicon.svg
│   ├── icons.svg
│   └── test-data/                          ← Sample files for upload testing
│       ├── MGNREGA_Receipt_Kolar_Karnataka.html
│       └── Healthcare_GeoTagged_Receipt_Nashik.html
│
├── src/
│   ├── App.jsx                             ← Root app + React Router routes
│   ├── main.jsx                            ← Entry point
│   ├── index.css                           ← Global styles + Tailwind
│   │
│   ├── components/
│   │   ├── Layout.jsx                      ← Page shell (sidebar + topbar)
│   │   ├── Sidebar.jsx                     ← Navigation menu (role-aware)
│   │   ├── Topbar.jsx                      ← Header bar + wallet address
│   │   └── DomainFilter.jsx                ← Scheme selector (MGNREGA etc)
│   │
│   ├── context/
│   │   ├── AuthContext.jsx                 ← userRole, walletAddress, activeDomain
│   │   └── TransactionContext.jsx          ← Live tx list, addLocal, updateStatus
│   │
│   ├── data/
│   │   ├── realData.js                     ← Scheme stats + static village data
│   │   ├── india_geocodes.json             ← 4,425 cities with lat/lng
│   │   ├── indian_cities.json              ← City list for autocomplete (~6100)
│   │   └── indian_cities_full.json         ← Extended city list
│   │
│   ├── pages/
│   │   ├── Login.jsx                       ← Role selection + MetaMask connect
│   │   ├── Dashboard.jsx                   ← Scheme analytics, charts, KPIs
│   │   ├── FundRelease.jsx                 ← Admin: initiate + proof upload
│   │   ├── FundLedger.jsx                  ← Transaction history table
│   │   ├── AuditorReview.jsx               ← Auditor: approve/flag queue
│   │   ├── FlagReport.jsx                  ← Detailed flag/freeze panel
│   │   ├── Verify.jsx                      ← Citizen: village search portal
│   │   ├── VillageMap.jsx                  ← Live geographic fund map
│   │   └── AuditExport.jsx                 ← PDF report generator
│   │
│   └── utils/
│       ├── contract.js                     ← Ethers.js: wallet + contract calls
│       ├── aiAnalysis.js                   ← Groq API: suspicion + PDF summary
│       ├── dataGenerator.js                ← Seeded random village analytics
│       ├── generateAuditPDF.js             ← jsPDF audit report builder
│       └── generateFIR.js                  ← FIR document generator
│
├── .env.example                            ← Template: keys needed (safe to commit)
├── .gitignore                              ← Excludes node_modules, .env, dist
├── README.md                               ← Setup guide (MetaMask, Remix, run)
├── ARCHITECTURE.md                         ← This file
├── index.html                              ← HTML shell
├── package.json                            ← Dependencies
├── vite.config.js                          ← Vite config
└── tailwind.config.js                      ← Tailwind theme
```

---

## 13. Key Design Decisions

### 1. Why Ethereum Sepolia (not Polygon / Solana)?
- **Free test ETH** via faucets — no real money needed for demo
- **Etherscan** support for public verification link
- **MetaMask** native support without custom network complexity
- EVM compatibility — contract can be deployed to mainnet unchanged

### 2. Why React Context (not Redux)?
The app has two small, focused contexts. Redux would add boilerplate with no benefit at this scale. Context + hooks is sufficient and keeps the codebase readable.

### 3. Why Deterministic IPFS Hashes?
Real IPFS requires a running node or paid pinning service (Pinata). For demo/hackathon purposes, we generate a stable `Qm...` hash from `fileName + fileSize`. The hash is consistent — the same file always produces the same hash, mimicking real IPFS content addressing.

### 4. Why `BigInt` not `ethers.parseUnits`?
`parseUnits(x, 0)` is designed for token amounts with decimal precision. For raw integer rupee values, it introduces floating-point errors: `1200000` becomes `99808702`. `BigInt(Math.round(Number(amount)))` preserves exact integer values.

### 5. Why `type="text"` for Amount Input?
`<input type="number">` in browsers auto-converts large integers to scientific notation (`1.2e+6`), which React reads as a string and stores incorrectly. `type="text"` with `inputMode="numeric"` gives full control — we strip non-digits on every keystroke.

---

## 14. Security Model

| Threat | Mitigation |
|---|---|
| **API key exposure** | Groq key in `.env` (gitignored) + `.env.example` template |
| **Fake transactions** | Requires MetaMask signature — cannot be forged |
| **Data tampering** | Blockchain is append-only — no delete or edit |
| **No proof uploaded** | UI blocks submission if `proofFiles.length === 0` |
| **Wrong network** | Auto-switches MetaMask to Sepolia; rejects other chains |
| **Suspicious transfers** | AI scan flags before transaction is signed |
| **Double spending** | Smart contract tracks all IDs — duplicate check in UI |

### What Remains for Production
- [ ] Replace simulated IPFS with real Pinata/Web3.Storage pin
- [ ] Add role-based smart contract access control (`onlyAdmin`, `onlyAuditor`)
- [ ] Multi-sig: require 3 auditor signatures before release
- [ ] Mainnet deployment (Polygon for low gas fees)
- [ ] National ID verification for wallet → official linking
- [ ] SMS/email alert to village head when funds released

---

## 🏁 Quick Start Reference

```bash
# Clone
git clone https://github.com/Yashas40/blockchain.git
cd blockchain

# Install
npm install

# Configure
cp .env.example .env
# Edit .env: add VITE_GROQ_API_KEY, VITE_CONTRACT_ADDRESS

# Run
npm run dev
# Open: http://localhost:5173
```

**Login as Admin:** Connect MetaMask → Select "Admin" → Upload proof → Release funds

**Login as Auditor:** Connect MetaMask → Select "Auditor" → Approve or flag

**Login as Citizen:** No wallet needed → Search village name → View blockchain proof

---

*ClearLedger v2.1 — Built for Government Transparency | Ethereum Sepolia Testnet*
