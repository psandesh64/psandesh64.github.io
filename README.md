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
