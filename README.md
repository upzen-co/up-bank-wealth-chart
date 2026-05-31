# 📈 Up Wealth

A single-file web app that charts your [Up Bank](https://up.com.au) balance over time — reconstructed from your full transaction history.

> **Unofficial tool — not affiliated with or endorsed by Up Banking.**

![Up Wealth screenshot placeholder](https://placehold.co/1200x600/f5f4f1/FF6B35?text=Up+Wealth)

-----

## ✨ Features

- **Balance history chart** — see your total Up balance plotted over time, going back as far as your transaction history allows
- **Per-account breakdown** — all your personal Up accounts shown individually with current balances
- **Smart caching** — data is cached locally so the app loads instantly on return visits; incremental refresh only fetches new transactions
- **100% client-side** — a single `.html` file with no server, no backend, no accounts to create
- **Dark mode** — follows your system preference automatically
- **Mobile-friendly** — responsive layout that works on any screen size

-----

## 🔒 Privacy — Your Token Never Leaves Your Device

This is the most important thing to understand about how Up Wealth works:

- **Your API token is never transmitted anywhere except directly to `api.up.com.au`** — there is no intermediary server
- **All data is stored in your browser’s `localStorage`** — it never touches any third-party server
- **The app makes requests directly from your browser to the Up API** — you can verify this in your browser’s DevTools Network tab
- **Disconnecting clears everything** — the “Disconnect” button wipes the token and all cached data from your device

You can [read the source code](./up-wealth-interactive-chart.html) in its entirety — it’s a single HTML file with no minification or obfuscation.

-----

## 🚀 Getting Started

### 1. Get your Up Personal Access Token

Go to **[api.up.com.au](https://api.up.com.au)** and create a Personal Access Token, or find it in the Up app under **Settings → Data Sharing**.

Your token looks like: `up:yeah:xxxxxxxxxxxxxxxxxxxx`

> **Keep your token secret.** It grants read access to your Up account data. Treat it like a password. You can revoke it any time from the Up app.

### 2. Open the app

**Option A — Visit the web app (easiest)**

Open **[jamesbedwell.github.io/up-wealth](https://jamesbedwell.github.io/up-wealth)** in your browser — no download or installation required.

**Option B — Download and run locally**

Download [`up-wealth-interactive-chart.html`](./up-wealth-interactive-chart.html) and open it directly in your browser. Useful if you’d prefer to run it entirely from your own machine.

### 3. Connect

Paste your token into the input field and tap **Connect to Up**. The app will fetch your accounts and transaction history (this may take a moment if you have a long history), then display your balance chart.

-----

## 📊 How the Chart Works

Up’s API doesn’t directly expose historical balance snapshots, so Up Wealth reconstructs your balance history by:

1. Fetching your **current account balance** from the API
1. Fetching your **full transaction history**
1. **Walking backwards** through time — subtracting each day’s net transactions to derive the end-of-day balance for every previous date

This means the rightmost point on the chart always matches your live Up balance exactly, with each prior point calculated from settled transactions.

> Only **personal (individual) accounts** are included. Joint accounts are excluded.

-----

## ⚡ Caching & Refresh

After the first load, everything is cached in `localStorage`:

- **Return visits** load instantly from the cache, with a timestamp showing when data was last fetched
- **Refresh** only fetches transactions newer than the most recent cached one — much faster than a full reload
- **Disconnect** wipes all cached data including your token

-----

## 🛠 Technical Details

|Detail              |Value                                               |
|--------------------|----------------------------------------------------|
|**Dependencies**    |[Chart.js 4.4.1](https://www.chartjs.org/) (via CDN)|
|**Framework**       |None — vanilla HTML/CSS/JS                          |
|**Storage**         |Browser `localStorage` only                         |
|**API**             |[Up Banking API v1](https://developer.up.com.au)    |
|**Permissions used**|`accounts:read`, `transactions:read`                |
|**File size**       |~20 KB                                              |

-----

## ⚠️ Disclaimer

This is an **unofficial, community-made tool** and is not affiliated with, endorsed by, or supported by Up Banking or Bendigo and Adelaide Bank.

Use of the Up API is subject to [Up’s API terms of service](https://api.up.com.au). Your Personal Access Token grants read access to your financial data — review Up’s documentation and revoke access at any time from within the Up app.