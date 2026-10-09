# Deploy a Website on GitHub Pages with a Custom Domain

## Step 1: Prepare and Deploy on GitHub

1. Log into GitHub and create a new **Public** repository.
   - **Tip:** Name it `yourusername.github.io` to make it your primary website.
2. Upload your website files. Make sure your main homepage file is named exactly `index.html` and sits in the root folder.
3. Go to the repository **Settings** tab.
4. In the left sidebar, locate the **"Code, planning, and automation"** section and click **Pages**.
5. Under **"Build and deployment"**, set the source to **Deploy from a branch**, select your main branch (e.g., `main` or `master`), then click **Save**.

## Step 2: Configure Your Custom Domain on GitHub

1. Scroll down on the same Pages settings menu until you see the **Custom domain** section.
2. Type your domain name (e.g., `yourdomain.com` or `www.yourdomain.com`) and click **Save**.
3. This automatically generates a `CNAME` file in the root of your repository.

## Step 3: Point Your DNS Provider to GitHub

Log into the account where you bought your domain (e.g., GoDaddy, Namecheap, Cloudflare) and open **DNS Management / DNS Zone Editor**.

You need to configure two types of records:

### 1. Apex Domain (Root) Records

To make sure `yourdomain.com` loads properly, create four **A** records. Use `@` for the Host/Name and point them to GitHub's official IP addresses:

| Type | Host / Name | Value (IP Address) |
|------|-------------|--------------------|
| A    | @           | 185.199.108.153    |
| A    | @           | 185.199.109.153    |
| A    | @           | 185.199.110.153    |
| A    | @           | 185.199.111.153    |

### 2. Subdomain Record

To make sure the `www` version works (e.g., `www.yourdomain.com`), create a **CNAME** record:

| Type  | Host / Name | Value / Target           |
|-------|-------------|--------------------------|
| CNAME | www         | `yourusername.github.io.` |

> Replace `yourusername` with your actual GitHub username.

> **Note:** If there are any pre-existing default A or CNAME records pointing to your registrar's parking page, delete them so they don't conflict.

## Step 4: Enforce HTTPS (Security)

DNS changes can take anywhere from a few minutes up to **24 hours** to propagate.

Once the changes settle:

1. Go back to your GitHub **Pages** settings.
2. Refresh the page.
3. Check the box **Enforce HTTPS**.

This issues a free SSL certificate so visitors see a secure padlock icon next to your URL.

# Free Node.js Backend + PostgreSQL Hosting (Render + Neon)

Use **Render** for the Node backend and **Neon** for Postgres. Don't use Render's own Postgres: free Render Postgres databases expire 30 days after creation, with a 14-day grace period before the data is permanently deleted. ([livemy](https://livemy.app/blog/render-pricing))

## Free-tier limits to know

- **Render:** A Free web service spins down after 15 minutes without inbound traffic, and spin-up takes about one minute. You get 750 free instance hours per workspace per month; exhaust them and free web services are suspended until the next month. ([jwatte](https://jwatte.com/blog/render-com-platform-review/), [onrender](https://render-www.onrender.com/articles/platforms-with-a-real-free-tier-for-developers-in-2026))
- **Neon:** Free plan storage was recently doubled to 1 GB per project. Free compute suspends after five minutes of inactivity. Neon, Supabase and Aiven all run permanent free tiers with no credit card. ([releases.sh](https://releases.sh/collections/serverless-postgres/digest/2026-09-28), [jetadmin](https://www.jetadmin.io/blog/neon-pricing/), [swyftstack](https://swyftstack.com/blog/free-postgresql-hosting))

## Step 1: Create the database (Neon)

1. Sign up at neon.com and create a project. Pick the region closest to your Render region (e.g., Singapore).
2. Copy the **connection string** (`postgresql://user:pass@host/db?sslmode=require`). Use the **pooled** one if offered.
3. Create your tables in Neon's SQL Editor.

## Step 2: Prepare the Express app

Install the packages:

```bash
npm i express pg cors
```

Then:

```js
const express = require('express');
const cors = require('cors');
const { Pool } = require('pg');

const app = express();
app.use(cors({ origin: 'https://sandeshpradhan.com.np' }));
app.use(express.json());

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: { rejectUnauthorized: false },
});

app.get('/api/health', async (req, res) => {
  const r = await pool.query('SELECT NOW()');
  res.json(r.rows[0]);
});

app.listen(process.env.PORT || 3000);
```

**Requirements:**

- `package.json` must have `"start": "node index.js"`.
- Use `process.env.PORT`. Render sets the port; a hardcoded port fails.
- Never commit `.env` or the DB URL. Add `.env` to `.gitignore`.

## Step 3: Push to GitHub

Use a separate repo from your `psandesh64.github.io` site.

## Step 4: Deploy on Render

1. Sign up at render.com with GitHub.
2. Go to **New → Web Service** and pick the repo.
3. Set:
   - Runtime: `Node`
   - Build: `npm install`
   - Start: `npm start`
   - Instance Type: **Free**
4. Under **Environment**, add `DATABASE_URL` set to the Neon connection string.
5. Deploy. You'll get a URL like `https://yourapp.onrender.com`.

## Step 5: Connect the frontend

From your GitHub Pages site:

```js
fetch('https://yourapp.onrender.com/api/health').then(r => r.json())
```

> The CORS origin in Step 2 must match your site domain exactly.

## Optional: Custom API domain

Add `api.sandeshpradhan.com.np` in Render's settings, then add a DNS record:

| Type  | Host / Name | Value / Target         |
|-------|-------------|------------------------|
| CNAME | api         | `yourapp.onrender.com` |

> Check that the free tier allows this in the Render dashboard; sources conflict.

## Avoiding cold starts

The first request after idle takes about 1 min. One fix is a free uptime pinger (e.g., UptimeRobot) hitting `/api/health` every 10 min.

> A 31-day month is 744h, which fits under the 750h limit for **one** service only.
