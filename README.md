# duitsnap.com – static website

Plain HTML plus one stylesheet. There is no build step, no JavaScript, no cookies, no analytics and no external fonts or CDNs.

```
website/
├── index.html            → https://duitsnap.com/
├── privacy/index.html    → https://duitsnap.com/privacy/   (EN / Bahasa Melayu / 简体中文, anchors #en #ms #zh)
├── support/index.html    → https://duitsnap.com/support/   (EN / Bahasa Melayu / 简体中文)
├── 404.html              → shown for unknown paths (Cloudflare Pages and GitHub Pages both use it)
├── styles.css
└── assets/
    ├── icon.png              (1024 × 1024 master copy of the app icon)
    ├── icon-512.png          (used on the pages, rounded with CSS)
    ├── apple-touch-icon.png  (180 × 180)
    ├── favicon-32.png
    └── favicon-16.png
```

## Before publishing: fill these in

Search the folder for `[` to find every placeholder:

| Placeholder | Where | Replace with |
|---|---|---|
| `[YOUR FULL NAME]` | all pages (footer), privacy policy §1 and §18 in all three languages | Your legal name, exactly as on your Apple Developer account |
| `[EMAIL PROVIDER]` | privacy policy §11 (EN/MS/ZH) | Whoever hosts `yikai@duitsnap.com`, e.g. "Google Workspace", "Zoho Mail", or "Cloudflare Email Routing, forwarding to Gmail" |
| `[HOSTING PROVIDER]` | privacy policy §12 (EN/MS/ZH) | "Cloudflare Pages" or "GitHub Pages" |
| App Store badge `href="#"` | `index.html` | The real App Store URL after approval (see "After launch") |

```sh
grep -rn "\[YOUR FULL NAME\]\|\[EMAIL PROVIDER\]\|\[HOSTING PROVIDER\]\|href=\"#\"" website/
```

If you change the effective date (26 September 2026), change it in all three language sections.

## Preview locally

The pages use root-relative links (`/styles.css`, `/privacy/`), so double-clicking the HTML file will not load the styles. Serve the folder instead:

```sh
cd website
python3 -m http.server 8000
# open http://localhost:8000/
```

---

## Option A (recommended): Cloudflare Pages

Free, fast in Malaysia, automatic HTTPS, and it works with an apex domain (`duitsnap.com`) once Cloudflare manages the domain's DNS.

### A1. Deploy the files

1. Create a free account at <https://dash.cloudflare.com>.
2. **Workers & Pages** → **Create** → **Pages** → **Upload assets** (Direct Upload).
3. Project name `duitsnap` → drag in the **contents** of the `website/` folder (so `index.html` sits at the top level; you can leave out `README.md`) → **Deploy**.
4. Check `https://duitsnap.pages.dev/`, `/privacy/` and `/support/`.
   To update later: open the project → **Create deployment** → upload the folder again. Alternatively, connect a GitHub repo so every push deploys automatically.

### A2. Move DNS to Cloudflare (needed for the apex domain)

1. In Cloudflare: **Add a domain** → `duitsnap.com` → **Free** plan. Cloudflare scans your existing DNS records.
2. **Check the imported records before continuing, especially email.** Keep every `MX` record and every `TXT` record for SPF (`v=spf1…`), DKIM and DMARC exactly as they were, so `yikai@duitsnap.com` keeps working.
   - **If your email is Squarespace/Google "email forwarding"** (not a real mailbox), that forwarding **stops working** when the nameservers leave Squarespace. Set up **Cloudflare Email Routing** instead (Email → Email Routing → forward `yikai@duitsnap.com` to your personal inbox) and let it add its own MX/TXT records.
   - If you use Google Workspace or Zoho, just make sure their MX/TXT records were imported.
3. Cloudflare shows two nameservers (e.g. `xxx.ns.cloudflare.com`). Change them at your registrar (see "Where are my DNS settings?" below).
4. Wait until Cloudflare says the domain is **Active** (minutes to a few hours; rarely up to 24 h).

### A3. Attach the domain to the Pages project

1. Pages project → **Custom domains** → **Set up a custom domain** → `duitsnap.com` → Activate. Cloudflare creates the DNS record and certificate for you.
2. Repeat for `www.duitsnap.com`, then add a redirect so `www` goes to the apex: **Rules → Redirect Rules → Create rule → template "Redirect from WWW to root"**.
3. SSL/TLS → Edge Certificates → turn on **Always Use HTTPS**.

---

## Option B: GitHub Pages (keeps DNS at Squarespace)

Use this if you'd rather not move nameservers, for example to keep Squarespace email forwarding.

1. Create a **public** GitHub repository (e.g. `duitsnap-site`) and put the **contents** of `website/` at the repository root. GitHub Pages can only publish from the root or from `/docs`.
2. Repo → **Settings → Pages** → Source: **Deploy from a branch** → `main` / `(root)` → Save.
3. Same page → **Custom domain**: `duitsnap.com` → Save (this adds a `CNAME` file).
4. At your DNS host (Squarespace, see below), **delete any existing `A`, `AAAA` or `CNAME` records for `@` and `www`** (for example Squarespace parking or "Squarespace Defaults" records), then add:

   | Type | Host | Value |
   |---|---|---|
   | A | @ | 185.199.108.153 |
   | A | @ | 185.199.109.153 |
   | A | @ | 185.199.110.153 |
   | A | @ | 185.199.111.153 |
   | AAAA | @ | 2606:50c0:8000::153 |
   | AAAA | @ | 2606:50c0:8001::153 |
   | AAAA | @ | 2606:50c0:8002::153 |
   | AAAA | @ | 2606:50c0:8003::153 |
   | CNAME | www | `YOUR-GITHUB-USERNAME.github.io` |

   Leave MX and TXT (email) records alone.
5. Once the DNS check passes, tick **Enforce HTTPS**. The certificate can take up to about an hour.
6. Recommended: GitHub → your profile **Settings → Pages → Add a verified domain** (adds a TXT record) so nobody else can claim `duitsnap.com` on GitHub Pages.

---

## Where are my DNS settings? (domain bought through Google)

**Google Domains no longer exists.** In 2023 all Google Domains registrations moved to **Squarespace Domains**, and Google's newer domain purchases (Google Cloud Domains) are also registered through Squarespace. So:

- **Bought at domains.google / Google Domains, or migrated from it:** sign in at **<https://account.squarespace.com/domains>** using **"Continue with Google"** with the same Google account you used to buy the domain. Click **duitsnap.com**, then:
  - **DNS records:** **DNS** → **DNS Settings** → **Custom records** → **Add record**. There may be preset blocks such as "Squarespace Defaults" or "Google Workspace". Delete a preset only if it conflicts with the records you are adding (for example `@` / `www` for GitHub Pages).
  - **Nameservers (Cloudflare option):** **DNS** → **Domain Nameservers** → **Use custom nameservers** → enter the two Cloudflare nameservers → Save.
  - **Email forwarding** (if you use it) is under **Email** → **Email Forwarding**. It only works while Squarespace nameservers are in use.
- **Bought in Google Cloud Console (Cloud Domains):** Console → **Network services → Cloud Domains** → your domain → **Edit DNS details**. Either manage records in **Cloud DNS**, or choose **Use custom name servers** and enter Cloudflare's.

DNS changes normally show up within minutes to a few hours. Check with `dig duitsnap.com +short` or <https://dnschecker.org>.

---

## Publishing checklist

- [ ] Placeholders replaced (table above), and `grep` finds nothing.
- [ ] Site deployed; `https://duitsnap.com/`, `https://duitsnap.com/privacy/` and `https://duitsnap.com/support/` all load **over HTTPS** with the padlock and no login.
- [ ] `http://` and `www.` versions redirect to `https://duitsnap.com/…`.
- [ ] The privacy page anchors work: `/privacy/#ms` and `/privacy/#zh`.
- [ ] A test email to `yikai@duitsnap.com` arrives, and your reply is delivered (not marked as spam; SPF/DKIM are set by your email provider).
- [ ] Viewed on an iPhone in light and dark mode, and with larger text (Settings → Display & Brightness → Text Size).
- [ ] The privacy policy wording matches the app build you submit (no features the build lacks, e.g. confirm the daily reminder ships).
- [ ] The same URLs are entered in App Store Connect (Support, Marketing, Privacy Policy).

## After launch

1. Replace the placeholder badge in `index.html` with Apple's official **"Download on the App Store"** badge (SVG from <https://developer.apple.com/app-store/marketing/guidelines/> → *App Store badges*). Save it as `assets/app-store-badge.svg` and link it to your app URL, e.g. `https://apps.apple.com/my/app/duitsnap/idXXXXXXXXXX`. Apple requires the official badge artwork. Don't redraw it.
2. Uncomment/add `<meta name="apple-itunes-app" content="app-id=XXXXXXXXXX">` in the `<head>` of `index.html` to show the Safari smart app banner.
3. If you change the privacy policy later: update all three languages, change the effective date, redeploy, and update the App Store privacy label **before** releasing an app version that changes data handling.
