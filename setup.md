# Setup guide: AI × Web3 Savings Challenge

This guide gets the **backend** and the **frontend** running on your computer, and sets up **MetaMask** on Chrome so you can use the blockchain part.

Everything uses synthetic data and a test network. No real money moves and no real customer data is used.

## What you will clone

You only clone **two repositories**: the **frontend** and the **backend**. Everything else (the mock API, the smart contract and the documentation) is either hosted online or already deployed, so there is nothing else to download.

```bash
git clone <frontend-repository-url>
git clone <backend-repository-url>
```

Keep the two folders side by side, for example inside one `ai-web3-savings-challenge` folder.

The documentation lives in its own repository and is **not** part of what you clone. You can come back to it at any time for more information:

| Document | What it covers |
|---|---|
| [setup.md](https://github.com/ai-web3-savings-challenge/docs/blob/main/setup.md) | This guide |
| [api-reference.md](https://github.com/ai-web3-savings-challenge/docs/blob/main/api-reference.md) | Mock API, AI endpoint and smart contract reference |
| [mentor-guide.md](https://github.com/ai-web3-savings-challenge/docs/blob/main/mentor-guide.md) | Checklist, demo script and common problems for mentors |

Docs repository: https://github.com/ai-web3-savings-challenge/docs

---

## What each part does

| Part | What it does |
|---|---|
| **Frontend** (`frontend/`) | The dashboard you see in the browser. Shows simulated bank data, lets you connect a wallet and record a savings goal on-chain, and shows AI savings insights. |
| **Backend** (`backend/`) | Two jobs. (1) `POST /ai/analyse-savings` takes a household's savings data, calculates the key numbers in code, asks Mistral AI to explain them, checks the reply, and falls back to a pre-written response if anything goes wrong. (2) An event listener watches the SavingsGoal smart contract on Celo Sepolia and runs your code whenever your wallet sets a goal or records a contribution. |
| **Mock API** (Apidog) | Hosted online. Supplies the simulated customer, transactions, savings and goal data. Nothing to install. |
| **Smart contract** | Already deployed on Celo Sepolia (test network). Nothing to deploy. |

How they connect:

```
Browser (frontend) ──GET──> Apidog mock API   (customer, transactions, savings, goal)
Browser (frontend) ──POST─> Backend :5000     (AI insights)  ──> Mistral AI
Browser (frontend) ──wallet──> Celo Sepolia contract <── Backend listens for its events
```

---

## 1. Prerequisites

Install or confirm these before you start.

| You need | Check / install |
|---|---|
| **Google Chrome** | https://www.google.com/chrome/ |
| **Node.js 18 or higher** (20 recommended) | Run `node -v`. If it is missing or below 18, install the LTS version from https://nodejs.org |
| **npm** | Comes with Node.js. Check with `npm -v` |
| **MetaMask** browser extension | See section 2 |
| **Testnet CELO** in your wallet | See section 2, step 6 |
| **Git** | Run `git --version`. If it is missing, install it from https://git-scm.com. Needed to clone the frontend and backend |
| **The frontend and backend repositories** | Cloned side by side (see "What you will clone") |
| **A code editor** (optional) | For example VS Code, to edit `.env` files |
| **Postman or Apidog** (optional) | To test the backend directly |

The API keys for the backend (such as the Mistral key) would be provided separately by the team. Never post them in public chats or commit them to git.

---

## 2. Install and set up MetaMask on Chrome

MetaMask is the wallet that signs the blockchain transactions.

**Use a separate test wallet.** Create a new MetaMask wallet only for this challenge. Do not use a wallet that holds real money.

### Step 1: Install the extension

1. Open Chrome and go to **https://metamask.io/download**. Only install from this official site, because fake MetaMask copies exist.
2. Click **Install MetaMask for Chrome**. This opens the Chrome Web Store page.
3. Click **Add to Chrome**, then **Add extension**.
4. Click the puzzle-piece icon in the Chrome toolbar and pin MetaMask so it is easy to reach.

### Step 2: Create the wallet

1. Open MetaMask and click **Create a new wallet**.
2. Agree to the terms and set a strong password. This password only unlocks MetaMask on this computer.
3. MetaMask shows a **Secret Recovery Phrase** (12 words). Write it on paper and store it safely. Confirm it when asked.

> **Never share the Secret Recovery Phrase or your private key** with anyone: not in chat, not in an app, not in this repository, not with "support". Anyone who has it controls the wallet.

### Step 3: Copy your wallet address

1. Open MetaMask and click the **account name** at the top. This copies your address.
2. It looks like `0x` followed by 40 letters and numbers. Keep it, because the backend needs it as `WALLET_ADDRESS` (section 3).

### Step 4: Add the Celo Sepolia test network

MetaMask does not include Celo Sepolia by default. Add it with **Option A** (Chainlist, quickest) or **Option B** (manual). You can also skip this step: if MetaMask is on the wrong network, the dashboard shows a **Switch to Celo Sepolia** button that offers to add it for you.

#### Option A: Add it from Chainlist

1. Open **https://chainlist.org** in Chrome.
2. Turn on **Include Testnets**, then search for `Celo Sepolia` or the chain ID `11142220`.
3. Click **Connect Wallet** at the top and approve the connection in MetaMask.
4. On the Celo Sepolia result, check that the chain ID is `11142220` and the currency is `CELO`, then click **Add to MetaMask**.
5. In the MetaMask pop-up, approve **Add network**, then approve **Switch network**.
6. Confirm MetaMask now shows **Celo Sepolia Testnet** at the top.

If Chainlist does not show it or the button does not work, use Option B.

#### Option B: Add it manually

1. In MetaMask, click the network name at the top, then **Add a custom network**.
2. Enter these values and save:

| Field | Value |
|---|---|
| Network name | Celo Sepolia Testnet |
| New RPC URL | `https://forno.celo-sepolia.celo-testnet.org` |
| Chain ID | `11142220` |
| Currency symbol | `CELO` |
| Block explorer URL | `https://celo-sepolia.blockscout.com` |

3. Switch MetaMask to **Celo Sepolia Testnet**.

### Step 5: (Optional) Use only one wallet extension

If you have other wallet extensions in Chrome (for example Coinbase Wallet), they can conflict with MetaMask. Disable the others while testing: `chrome://extensions`.

### Step 6: Get free testnet CELO

Transactions need CELO testnet faucet to pay for gas. It has no real value.

1. Open one of these faucets:
   - Celo faucet: https://faucet.celo.org/celo-sepolia
   - Google Cloud faucet: https://cloud.google.com/application/web3/faucet/celo/sepolia
2. Paste your wallet address (copied in step 3) and request funds. If one faucet is busy or has a limit, try the other.
3. Check that the balance appears in MetaMask while on the Celo Sepolia network. It can take a minute.

Without testnet CELO you can still view the whole dashboard, but you cannot create goals or record contributions.

---

## 3. Backend setup

Start the backend **before** you use the AI insights in the frontend.

### Install and configure

1. Open a terminal and go into the backend:

   ```bash
   cd backend
   npm install
   ```

2. Create your settings file:

   ```bash
   # Mac / Linux
   cp .env.example .env

   # Windows
   copy .env.example .env
   ```

3. Open `backend/.env` and fill in these values:

   ```env
   AI_MODE=live
   MISTRAL_API_KEY=
   MISTRAL_MODEL=ministral-8b-latest
   MISTRAL_TIMEOUT_MS=15000
   PORT=5000
   FRONTEND_URL=http://localhost:5173
   CELO_RPC_URL=https://forno.celo-sepolia.celo-testnet.org
   CONTRACT_ADDRESS=0xf3bF2289BB778bE70829DC4E7C630db4B348f5DE
   WALLET_ADDRESS=paste-your-own-wallet-address-here
   EVENTS_FROM_BLOCK=latest
   ```

| Setting | What to put |
|---|---|
| `MISTRAL_API_KEY` | The key provided by the team |
| `WALLET_ADDRESS` | Your own MetaMask address (section 2, step 3). The full address, `0x` plus 40 characters |
| `FRONTEND_URL` | Where the frontend runs. Keep `http://localhost:5173` unless the frontend uses another port |
| The rest | Leave as shown. Each setting should appear only once |

The remaining API keys will be provided by the team. Never commit `.env`.

### Start the backend

```bash
npm run dev
```

You should see:

```
Backend running on port 5000
Event listener on. Wallet: 0x.... Reading from block ....
```

Open **http://localhost:5000/health** in your browser to check it is alive.

> After changing `backend/.env`, stop the backend (**Ctrl+C**) and run `npm run dev` again.

### What you can change

The only file you edit is `src/handlers/eventHandlers.js`. Replace the `console.log` in `onGoalCreated` and `onContributionRecorded` with your own logic. Leave every other file as it is: they run the AI endpoint and the event listener.

### Test the AI endpoint (optional)

In Postman or Apidog, send a **POST** to `http://localhost:5000/ai/analyse-savings`. Under **Body** choose **raw** and **JSON**, and paste the contents of `docs/sample-request.json`. The response contains `analysis`, `meta` and `facts`.

---

## 4. Frontend setup

### Install and configure

1. Open a **second** terminal (keep the backend running in the first) and go into the frontend:

   ```bash
   cd frontend
   npm install
   ```

2. Create your settings file:

   ```bash
   # Mac / Linux
   cp .env.example .env

   # Windows
   copy .env.example .env
   ```

3. Open `frontend/.env` and check these values:

   ```env
   VITE_API_BASE_URL=https://mock.apidog.com/m1/1403343-1412709-default/
   VITE_AI_API_URL=http://localhost:5000
   VITE_WALLETCONNECT_PROJECT_ID=
   VITE_CHAIN_ID=11142220
   VITE_RPC_URL=https://forno.celo-sepolia.celo-testnet.org
   VITE_EXPLORER_URL=https://celo-sepolia.blockscout.com
   VITE_CONTRACT_ADDRESS=0xf3bF2289BB778bE70829DC4E7C630db4B348f5DE
   ```

| Setting | What it does |
|---|---|
| `VITE_API_BASE_URL` | The Apidog mock API. Already filled in. Empty it to use built-in sample data |
| `VITE_AI_API_URL` | Your backend. Set to `http://localhost:5000` so the insights card uses it. If left empty, the card shows a fixed sample response instead |
| `VITE_WALLETCONNECT_PROJECT_ID` | Only needed for the **Connect with WalletConnect** button (phone wallets). Free ID from https://cloud.reown.com. Not needed for MetaMask in Chrome |
| The rest | Celo Sepolia network and contract settings. Leave as shown |

Settings starting with `VITE_` are visible to anyone using the page, so never put secret keys in this file. The Mistral key belongs only in the backend `.env`.

### Start the frontend

```bash
npm run dev
```

Open the address it prints, usually **http://localhost:5173**.

> After changing `frontend/.env`, stop the frontend (**Ctrl+C**) and run `npm run dev` again.

If the terminal says it is using a different port (for example 5174), set `FRONTEND_URL` in the backend `.env` to match and restart the backend. Otherwise the browser will block the AI requests.

---

## 5. Check that everything works

Work through this list in order.

1. **Backend health.** http://localhost:5000/health responds.
2. **Dashboard loads.** The page at http://localhost:5173 shows the bank data (goal, household, savings, transactions). No yellow warning about failed endpoints.
3. **Connect MetaMask.** Click **Connect browser wallet**, approve in MetaMask. Your shortened address and a green **Celo Sepolia Testnet** pill appear. If you see **Wrong network**, click **Switch to Celo Sepolia**.
4. **Create an on-chain goal.** In **On-chain demo record**, click **Create goal** and approve in MetaMask. When it says confirmed, click **View transaction** to see it on Blockscout.
5. **Record a contribution.** Enter a whole number and click **Record contribution**, then approve. The recorded total updates after confirmation.
6. **Backend sees the event.** In the backend terminal, your event handler runs (the `console.log` in `src/handlers/eventHandlers.js`) for the goal and the contribution. This only works if `WALLET_ADDRESS` is the same wallet you connected in step 3.
7. **AI insights.** Click **Get insights**. The card shows a summary, observations, recommendations and a short list of key figures. The small label under it should say the text is AI-generated.

Reminders: the blockchain figures are **demo records**, not real money, and creating a goal again replaces the old goal and resets its recorded total to 0.

---

## 6. Troubleshooting

| What you see | Likely cause and fix |
|---|---|
| `npm install` fails | Check `node -v` is 18 or higher and that you are online |
| Backend does not start, or says a setting is missing | `.env` is missing or incomplete. Copy `.env.example` to `.env` and fill it in |
| Changed `.env` but nothing changed | Stop with Ctrl+C and start again. `.env` is only read at start |
| Backend `/health` does not open | Backend is not running, or `PORT` is different from 5000 |
| Event listener does nothing | `WALLET_ADDRESS` is wrong or cut short. It must be the full address (`0x` + 40 characters) of the wallet you use in MetaMask. Also, `EVENTS_FROM_BLOCK=latest` only catches events that happen after the backend starts |
| Insights card says "mock API" or "placeholder" | `VITE_AI_API_URL` is empty or the backend is off. Set it to `http://localhost:5000`, start the backend, restart the frontend |
| Browser console shows a CORS error | `FRONTEND_URL` in the backend `.env` does not match the address of the frontend. Fix it and restart the backend |
| AI always returns the pre-written answer | Mistral key missing or wrong, `AI_MODE` is not `live`, or the request timed out. Check the backend terminal for the reason |
| "No browser wallet found" | MetaMask is not installed or not enabled in Chrome, or you are in a Chrome profile without it |
| **Wrong network** pill | MetaMask is on another network. Click **Switch to Celo Sepolia** |
| "Not enough testnet CELO for gas" | Get free testnet CELO from a faucet (section 2, step 6) |
| "You rejected the request" | You cancelled in MetaMask, so nothing was sent. Try again |
| Yellow notice "endpoints failed" | The mock API was unreachable or returned an unexpected shape. The notice names the endpoint. Open the browser console (F12) for details |
| Goal shows as empty after switching accounts | Goals belong to one wallet address. A new account has no goal yet |
| WalletConnect error about project ID | Only matters if you use WalletConnect. Add `VITE_WALLETCONNECT_PROJECT_ID` to `frontend/.env`, or use the MetaMask button instead |

---

## 7. Security checklist

- Never commit `.env` files. Check that `.env` is listed in `.gitignore`.
- Never share the Mistral key, your Secret Recovery Phrase or any private key.
- Use a dedicated test wallet with testnet tokens only.
- Use synthetic data only. Do not put real customer information anywhere.
- Treat AI output as suggestions, not verified facts or financial advice.

## 8. Starting everything each day

1. Terminal 1: `cd backend` then `npm run dev`
2. Terminal 2: `cd frontend` then `npm run dev`
3. Open http://localhost:5173 in Chrome, unlock MetaMask, and connect.

## 9. Need more information?

This guide only covers setup. For anything else, go back to the documentation repository at any time: https://github.com/ai-web3-savings-challenge/docs

- Data formats, the AI request and response, and the smart contract functions: `api-reference.md`
- Demo steps, common problems and known issues: `mentor-guide.md`
