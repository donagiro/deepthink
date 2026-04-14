# DeepThink AI — Website

Static marketing site for [deepthinkai.org](https://deepthinkai.org/), a strategy and organizational excellence consultancy.

## Stack

| Layer | Technology |
|---|---|
| Markup | HTML5 (single page — `index.html`) |
| Styles | Vanilla CSS (`style.css`) |
| Fonts | Google Fonts — DM Sans + DM Serif Display |
| Contact form | Formspree (AJAX, form ID `mbdqlzqg`) |
| Favicon | Inline SVG (`favicon.svg`) |
| Hosting | Namecheap shared hosting — `~/public_html/` |
| DNS / CDN | Cloudflare (proxied, with Email Routing) |
| Deploy | GitHub Actions → rsync over SSH |

## Project structure

```
deepthink/
├── index.html          # Single-page site (all sections)
├── style.css           # All styles
├── favicon.svg         # SVG favicon (blue-purple gradient)
├── .htaccess           # Apache config: HTTPS redirect, caching, security headers
├── .github/
│   └── workflows/
│       └── deploy.yml  # Auto-deploy to cPanel on push to main
├── .cpanel.yml         # Legacy cPanel Git config (not used — see deploy.yml)
└── .hintrc             # HTML linting config
```

## Deployment

Every push to `main` triggers the GitHub Actions workflow in `.github/workflows/deploy.yml`.

**What it does:**
1. Checks out the repository
2. Installs the SSH private key from `CPANEL_SSH_KEY` secret
3. Adds the server's host key to `~/.ssh/known_hosts` via `ssh-keyscan` on port **21098** (Namecheap's non-standard SSH port)
4. Runs `rsync` over SSH (port 21098) to sync changed files to `~/public_html/` on the server
5. Excludes `.git/`, `.github/`, `.hintrc`, `.cpanel.yml`, and `README.md` from the transfer

**Required GitHub secrets** (set under Settings → Secrets → Actions):

| Secret | Value |
|---|---|
| `CPANEL_HOST` | `server315.web-hosting.com` |
| `CPANEL_USER` | cPanel username |
| `CPANEL_SSH_KEY` | SSH private key (matching public key added in cPanel SSH Access) |

> **Note:** Namecheap shared hosting uses SSH port **21098**, not the default 22. Both `ssh-keyscan` and `rsync` must specify `-p 21098` / `-e "ssh -p 21098"` respectively. This was the root cause of a prior deploy failure.

## Contact form (Formspree)

The form in `index.html` posts to `https://formspree.io/f/mbdqlzqg`. Submissions are forwarded to the admin email configured in the Formspree dashboard. The JavaScript handler in `index.html` detects whether the placeholder `YOUR_FORM_ID` is still present and disables AJAX submission if so — this is how you know the form ID is live.

Free tier: 50 submissions/month.

## Email

`info@deepthinkai.org` is routed to `doagiro@gmail.com` via **Cloudflare Email Routing**.

DNS records required (managed in Cloudflare):
- MX records pointing to Cloudflare's mail servers
- SPF TXT record authorizing Cloudflare to send
- DKIM TXT record

When adding or changing email routing, delete any conflicting MX/SPF records first — Cloudflare's "Add records and enable" button stays disabled until all conflicts are removed.

## DNS

Domain registrar: **Namecheap** (account: `donagiro`)  
Nameservers: **Cloudflare** (changed from Namecheap default; zone status: Active once propagated)  
Canonical URL: `https://deepthinkai.org/` (non-www enforced via `.htaccess` redirect)

## Apache configuration (`.htaccess`)

- HTTP → HTTPS redirect (301)
- www → non-www redirect (301)
- Gzip compression for HTML, CSS, JS, SVG, fonts
- Browser caching: HTML 1 hour, CSS/JS 1 month, fonts 1 year, images 6 months
- Security headers: `X-Frame-Options`, `X-Content-Type-Options`, `X-XSS-Protection`, `Referrer-Policy`, `Permissions-Policy`
- 404 → `/index.html` (single-page fallback)
- Block `.git` and `.github` directory exposure

## Hosting renewal

Namecheap shared hosting expires **May 29, 2026**. Auto-renew is currently **OFF** — enable it in the Namecheap account before that date to prevent the site going offline.

## Local development

No build step. Open `index.html` directly in a browser or use any static file server:

```bash
npx serve .
# or
python -m http.server 8080
```

To test form submissions locally, either use a real Formspree endpoint or temporarily disable the JS handler's ID check.
