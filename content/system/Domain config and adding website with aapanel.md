+++
title = "Complete Guide: Deploying a React Portfolio on VPS with aaPanel & Cloudflare"
date = "2026-10-16"
author = "FAKE"
+++

# Complete Guide: Deploying a React Portfolio on VPS with aaPanel & Cloudflare

This guide walks you through connecting your custom domain to Cloudflare, pointing DNS records to your VPS, hosting your React production build on aaPanel, and configuring end-to-end SSL/TLS encryption.

---

## Step 1: Add Your Domain to Cloudflare

1. **Log in to Cloudflare:**
   - Go to [dash.cloudflare.com](https://dash.cloudflare.com/) and sign in.
2. **Add Domain:**
   - Click **Add a site** (or **Add domain**).
   - Enter `yourdomain` and select the **Free Plan**.
3. **Scan Records:**
   - Cloudflare will automatically scan existing DNS records.
   - Click **Continue** to proceed to DNS review.

---

## Step 2: Configure Accurate DNS Records

During domain migration, existing records from previous hosting providers (like old shared hosts) must be updated to your VPS IP.

1. In the **Review your DNS records** screen:
   - **Root Domain Record:**
     - **Type:** `A`
     - **Name:** `@` (or `yourdomain`)
     - **IPv4 Address:** `Your VPS Public IP`
     - **Proxy status:** **Proxied** (Orange cloud enabled)
     - **TTL:** `Auto`
   - **WWW Subdomain Record:**
     - **Type:** `CNAME`
     - **Name:** `www`
     - **Target:** `yourdomain`
     - **Proxy status:** **Proxied** (Orange cloud enabled)
     - **TTL:** `Auto`
2. **Delete Old / Conflicting Records:**
   - If any old `A` record points to an outdated IP (e.g. `103.65.139.18`), edit or delete it immediately.
3. Click **Continue to activation**.

---

## Step 3: Update Nameservers at Your Domain Registrar

Cloudflare will provide two authoritative nameservers (for example: `gina.ns.cloudflare.com` and `todd.ns.cloudflare.com`).

1. Log in to the registrar where you purchased `yourdomain` (e.g., Namecheap, Hostinger, GoDaddy, Porkbun).
2. Find the **DNS Management** or **Nameserver** section for `yourdomain`.
3. Switch nameservers from **Default Registrar DNS** to **Custom DNS**.
4. Enter the two Cloudflare nameservers provided.
5. Save changes.
6. Return to Cloudflare and click **Done, check nameservers**.

> **Note:** Nameserver propagation typically takes between 5 to 30 minutes, though global registrar propagation can take up to 24 hours.

---

## Step 4: Configure the Site in aaPanel

1. **Access aaPanel:**
2. **Add Website:**
   - Click **Website** in the left sidebar.
   - Under the **PHP Project** (or **Static**) tab, click **Add site**.
   - **Domain:**

     ```text
     yourdomain.com
     www.yourdomain.com
     ```

   - **Description:** `Website`
   - **Root Directory:** Default is `/www/wwwroot/yourdomain`
   - **FTP / Database:** Leave as `Do not create` (not needed for static frontends).
   - **PHP Version:** Select `Static` (or leave default; Nginx handles delivery directly).
   - Click **Submit**.

---

## Step 5: Build & Deploy Your React Portfolio

### 1. Build Locally

Inside your React project directory on your development machine:

```bash
# If using Vite
npm run build
# Output directory: ./dist

# If using Create React App
npm run build
# Output directory: ./build
```

### 2. Upload to VPS via aaPanel

1. In aaPanel, navigate to **Files** > `/www/wwwroot/domain.com`.
2. Delete the default files (`index.html`, `404.html`, `.htaccess`).
3. Click **Upload** and upload your production files:
   - Ensure `index.html` resides directly in `/www/wwwroot/domain.com/index.html`.
   - Ensure the `assets/` or `static/` folders sit alongside `index.html`.

### 3. Setup Single Page App (SPA) URL Rewrite

If your portfolio uses React Router (e.g. `/projects`, `/about`), refreshing any page will cause a `404 Not Found` without Nginx rewrite rules.

1. Go to **Website** > click on **aliridowan.com**.
2. Select **URL rewrite** from the left tab.
3. Add the following rule:

   ```nginx
   location / {
       try_files $uri $uri/ /index.html;
   }
   ```

4. Click **Save**.

---

## Step 6: Configure SSL/TLS (Cloudflare & aaPanel)

To ensure secure HTTPS without redirect loops (`ERR_TOO_MANY_REDIRECTS`), follow these configurations:

### Method A: Cloudflare Origin CA (Recommended for Best Security)

1. **In Cloudflare Dashboard:**
   - Go to **SSL/TLS** > **Origin Server**.
   - Click **Create Certificate**.
   - Keep default hostnames (`domain.com`, `*.domain.com`) and 15-year validity.
   - Click **Create**.
   - Cloudflare displays an **Origin Certificate** and a **Private Key**.
2. **In aaPanel:**
   - Go to **Website** > **aliridowan.com** > **SSL** tab.
   - Click **Business certificate** (or manual entry).
   - Paste the **Private Key** into the `Private key (KEY)` field.
   - Paste the **Origin Certificate** into the `Certificate (CRT/PEM)` field.
   - Click **Save and enable SSL**.
3. **Set Cloudflare SSL Mode:**
   - In Cloudflare, go to **SSL/TLS** > **Overview**.
   - Select **Full (strict)**.
   - Under **SSL/TLS** > **Edge Certificates**, enable **Always Use HTTPS**.

---

### Method B: Flexible SSL (Quick Setup)

If you do not want to install a certificate on aaPanel right away:

1. In Cloudflare, set **SSL/TLS mode** to **Flexible**.
2. Traffic between browser and Cloudflare is encrypted (HTTPS).
3. Cloudflare reaches your VPS over port 80 (HTTP).
4. *Important:* Do not enable force-HTTPS redirects on aaPanel when using Flexible mode, as this creates an infinite loop.

---

## Step 7: Verification & Testing Checklist

- [ ] **DNS Resolution:**

  ```bash
  curl -I https://aliridowan.com
  ```

  Check that the HTTP response returns `200 OK` and headers include `server: cloudflare`.
- [ ] **Proxy Verification:**
  Pinging `aliridowan.com` returns a Cloudflare anycast IP (e.g. `104.x.x.x` or `172.x.x.x`), successfully masking your origin VPS IP .
- [ ] **SPA Route Refresh:**
  Navigate to an internal route (e.g., `https://domain.com/about`) and reload to verify client routing handles the path seamlessly.
- [ ] **SSL Grade:**
  Confirm green padlock status on both `https://domain.com` and `https://www.domain.com`.

---

## Common Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| **Old host page displays** | Cloudflare `A` record points to old provider IP or local DNS cache. | Update Cloudflare `A` record to `server ip`, flush DNS (`ipconfig /flushdns` or browser cache). |
| **`ERR_TOO_MANY_REDIRECTS`** | Cloudflare is set to *Flexible*, while aaPanel Nginx forces HTTPS. | Change Cloudflare SSL to **Full** / **Full (strict)**, or disable origin-level HTTPS redirection. |
| **`404 Not Found` on refresh** | Single Page Application route missing Nginx rewrite. | Add `try_files $uri $uri/ /index.html;` in aaPanel **URL rewrite**. |
| **`521 Web Server Is Down`** | Nginx stopped or VPS firewall blocking ports 80/443. | Check Nginx status in aaPanel; run `ufw allow 80/tcp` and `ufw allow 443/tcp` on your VPS. |
