# ⛓ Blockchain-Based Certificate Verification System

**N.B.K.R Institute of Science & Technology – CSE Department – Batch 22PR039**

> A full-stack decentralized application (DApp) to **issue**, **store**, and **verify** academic certificates using the Ethereum blockchain, preventing forgery through SHA-256 hashing and immutable smart contracts.

---

## 📁 Project Structure

```
blockchain-cert-verify/
├── smart-contract/               ← Solidity smart contract (Hardhat)
│   ├── contracts/
│   │   └── CertificateVerification.sol
│   ├── scripts/
│   │   └── deploy.js
│   ├── test/
│   │   └── CertificateVerification.test.js
│   ├── hardhat.config.js
│   └── package.json
│
├── backend/                      ← Node.js + Express REST API
│   ├── models/
│   │   ├── Certificate.js        ← MongoDB schema
│   │   └── User.js
│   ├── routes/
│   │   ├── auth.js               ← Login / Register
│   │   └── certificates.js       ← Issue / Verify APIs
│   ├── middleware/
│   │   └── auth.js               ← JWT middleware
│   ├── utils/
│   │   └── blockchain.js         ← Ethers.js integration
│   ├── server.js
│   └── package.json
│
└── frontend/                     ← React.js UI
    ├── public/
    │   └── index.html
    └── src/
        ├── components/
        │   ├── Navbar.js
        │   └── PrivateRoute.js
        ├── pages/
        │   ├── HomePage.js
        │   ├── LoginPage.js
        │   ├── VerifyPage.js     ← Public: verify by file or hash
        │   ├── IssuePage.js      ← Admin: issue certificates
        │   └── DashboardPage.js  ← Admin: view all certificates
        ├── utils/
        │   ├── api.js            ← Axios API calls
        │   └── AuthContext.js    ← Auth state management
        ├── App.js
        ├── index.js
        └── index.css
```

---

## 🛠️ Prerequisites

Install the following before starting:

| Tool | Version | Download |
|------|---------|----------|
| Node.js | v18+ | https://nodejs.org |
| MongoDB | v6+ | https://mongodb.com/try/download/community |
| Git | any | https://git-scm.com |

---

## 🚀 Step-by-Step Setup

### Step 1 — Clone / Open the Project

```bash
cd blockchain-cert-verify
```

---

### Step 2 — Setup & Deploy the Smart Contract

```bash
# Navigate to smart contract directory
cd smart-contract

# Install Hardhat and dependencies
npm install

# Compile the Solidity contract
npx hardhat compile
# ✅ Should show: "Compiled 1 Solidity file successfully"

# Run tests
npx hardhat test
# ✅ Should show all tests passing

# Open a NEW terminal window and start local Ethereum node
npx hardhat node
# ✅ Shows 20 test accounts with 10000 ETH each
# Keep this terminal running!

# In original terminal — deploy the contract to local node
npx hardhat run scripts/deploy.js --network localhost
# ✅ Outputs: CertificateVerification deployed to: 0x5FbDB2...
# ✅ Creates deployment.json with contract address and ABI
```

---

### Step 3 — Setup the Backend

```bash
# Navigate to backend directory
cd ../backend

# Install dependencies
npm install

# Create your .env file
cp .env.example .env
# Edit .env if needed (defaults work for local development)

# Make sure MongoDB is running:
# On Windows: net start MongoDB
# On Mac/Linux: sudo systemctl start mongod
# Or simply: mongod

# Start the backend server
npm run dev
# ✅ Shows: MongoDB connected
# ✅ Shows: Server running on http://localhost:5000
```

---

### Step 4 — Setup the Frontend

```bash
# Navigate to frontend directory
cd ../frontend

# Install dependencies
npm install

# Start the React development server
npm start
# ✅ Opens http://localhost:3000 in your browser
```

---

### Step 5 — Create an Admin Account

1. Open http://localhost:3000/login in your browser
2. Click **Register** tab
3. Fill in your name, email, password
4. Set Role to **Admin**
5. Set Institution to **N.B.K.R Institute of Science & Technology**
6. Click **Create Account**
7. You'll be redirected to the Dashboard

---

### Step 6 — Issue Your First Certificate

1. Go to **Issue Certificate** page (top navbar)
2. Fill in student details (Name, Roll No., Course, Date)
3. Upload a certificate PDF or image
4. Click **Issue & Store on Blockchain**
5. ✅ A SHA-256 hash is computed and stored on Ethereum
6. Note the **certificate hash** shown in the success message

---

### Step 7 — Verify a Certificate

1. Go to **Verify Certificate** page (public, no login needed)
2. Choose **Upload File** tab
3. Upload the same certificate file
4. Click **Verify on Blockchain**
5. ✅ Shows **VERIFIED** with student details

Or use **Enter Hash** tab and paste the hash from Step 6.

---

## 🔌 API Endpoints Reference

### Auth
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Register admin/verifier |
| POST | `/api/auth/login` | Login, returns JWT |
| GET | `/api/auth/me` | Get current user |

### Certificates
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/certificates/issue` | Admin | Upload & issue certificate |
| POST | `/api/certificates/verify/file` | Public | Verify by file upload |
| GET | `/api/certificates/verify/:hash` | Public | Verify by hash string |
| GET | `/api/certificates` | Admin | List all certificates |
| POST | `/api/certificates/hash` | Public | Get hash of a file |

---

## 🔗 Smart Contract Functions

```solidity
// Store a certificate (admin only)
storeCertificate(bytes32 hash, string studentName, string course, string date, string institution)

// Verify and emit event (gas cost)
verifyCertificate(bytes32 hash) returns (bool isValid, ...)

// Read-only verify (no gas / free)
getCertificate(bytes32 hash) view returns (bool isValid, ...)

// Admin: authorize another address as issuer
setIssuer(address issuer, bool status)

// Total count of certificates stored
getTotalCertificates() view returns (uint256)
```

---

## ⚙️ How It Works (Technical Flow)

```
ISSUE FLOW:
Admin uploads PDF → Backend computes SHA-256 → Calls storeCertificate() on Ethereum
→ Transaction mined → Hash saved immutably on blockchain → Metadata saved to MongoDB

VERIFY FLOW:
User uploads PDF → Backend computes SHA-256 → Calls getCertificate() on Ethereum
→ If hash found → VALID ✅ | If not found → INVALID ❌
```

---

## 🧪 Testing the Smart Contract

```bash
cd smart-contract
npx hardhat test
```

Tests cover:
- Deployment and ownership
- Authorized issuer management
- Certificate storage and duplicate prevention
- Verification (valid and invalid cases)
- Certificate count tracking

---

## 🌐 Deploying to Sepolia Testnet (Optional)

1. Get free Sepolia ETH from https://sepoliafaucet.com
2. Create an Infura account at https://infura.io
3. Edit `smart-contract/.env`:
   ```
   INFURA_URL=https://sepolia.infura.io/v3/YOUR_PROJECT_ID
   PRIVATE_KEY=your_wallet_private_key
   ```
4. Deploy:
   ```bash
   npx hardhat run scripts/deploy.js --network sepolia
   ```
5. Update `backend/.env`:
   ```
   RPC_URL=https://sepolia.infura.io/v3/YOUR_PROJECT_ID
   ```

---

## 👥 Team

| Name | Roll No. |
|------|----------|
| T. Mounika | 22KB1A05H4 |
| K. Sai Pallavi | 22KB1A0565 |
| N. Nikitha | 22KB1A05B2 |
| M. Chamudeswari | 22KB1A0591 |

**Guide:** O. Kiran Kishore, M.Tech. — Assistant Professor, Dept. of CSE

---

## 📦 Tech Stack Summary

| Layer | Technology |
|-------|-----------|
| Frontend | React.js, React Router, Axios |
| Backend | Node.js, Express.js |
| Blockchain | Solidity, Hardhat, Ethers.js |
| Database | MongoDB, Mongoose |
| Auth | JWT (JSON Web Tokens) |
| Hashing | SHA-256 (Node.js crypto) |
| File Upload | Multer |

---

*Built for N.B.K.R Institute of Science & Technology – CSE Department – 2024*
