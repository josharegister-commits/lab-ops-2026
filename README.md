# Medtronic ENT — Jacksonville Lab Apps

Two static HTML apps that live in the same folder and read from the same Teamup calendar (`Med Ed > Jacksonville Lab` sub-calendar of the Medtronic ENT Education Teamup).

## Apps

### 1. ENT Immersion Lab Booking — `ENT_Immersion_Lab_Booking.html`

The public-facing booking page used by ENT sales reps to claim an "Open ENT Immersion Lab" slot. Reps pick from available dates, fill in their info, and the app updates the matching Teamup event title (`Open ENT Immersion Lab` → `ENT Immersion Lab — Dr. X — Rep Y`) so it lands in the manager dashboard with all the prep tasks attached.

### 2. Jacksonville Lab Manager Dashboard — `Jacksonville_Lab_Manager_Dashboard.html`

The internal ops dashboard. Pulls the Jacksonville sub-calendar live, classifies every event by type (Lab, Training, R&D, Specimen Delivery/Pickup, Cleanup, Install, etc.), and generates a multi-day prep checklist for each one. Includes a Medtronic catalog (209+ part numbers), pre/post-event restock email generator, daily ops checklists, two par-check systems, and a cadaver log.

## Role-based URLs

The dashboard reads `?role=...` from the URL to filter what each person sees.

| Person | URL suffix | What they see |
|---|---|---|
| Master (Josh) | *(none)* | Everything, with role badges per task |
| Ana Maria Rojas | `?role=anamaria` | Specimen ordering + meeting planning tasks only |
| Bert Clark | `?role=bert` | All lab manager tasks + daily ops + par checks + cadaver log |
| Latisha | `?role=latisha` | Only events with "latisha" in the Teamup title |

The master view also has a `?view=team` toggle for an event-pivot layout.

For one-click access, the `Jax Lab — *.html` files in this folder are tiny redirects to each role's URL.

## Deploy to GitHub Pages

This is a single-folder static site. Once on GitHub:

```bash
# from inside this folder
git init
git add .
git commit -m "initial commit"
git branch -M main
git remote add origin git@github.com:YOUR_USERNAME/REPO_NAME.git
git push -u origin main
```

Then on GitHub:
1. **Repo → Settings → Pages**
2. Source: `Deploy from a branch`, Branch: `main`, Folder: `/ (root)`
3. Save. Site appears at `https://YOUR_USERNAME.github.io/REPO_NAME/`

Each app is reachable at:
- `https://YOUR_USERNAME.github.io/REPO_NAME/Jacksonville_Lab_Manager_Dashboard.html`
- `https://YOUR_USERNAME.github.io/REPO_NAME/ENT_Immersion_Lab_Booking.html`

To make the dashboard the default landing page, rename it to `index.html`. Or add a small `index.html` that links to both.

**This repo MUST be private.** It contains:
- A Teamup API read+write token
- Internal Medtronic cost-center mappings (DSM names → CC numbers)
- Lab address and contact info
- Jacksonville sub-calendar ID

GitHub Pages from a private repo requires GitHub Pro, Team, or Enterprise.

## Updating

Make changes locally, then:

```bash
git add .
git commit -m "what you changed"
git push
```

GitHub Pages rebuilds the site automatically (usually within a minute). The dashboard caches nothing on the server side — every page load fetches fresh from Teamup.

## When someone leaves the team

The dashboard's "URL = password" model is not real auth. To revoke access cleanly:

1. **Rotate the Teamup API token** in the Teamup admin console. Anyone with a saved copy of the file loses calendar access. Update the new token in `Jacksonville_Lab_Manager_Dashboard.html` (search for `teamupApiToken`) and `ENT_Immersion_Lab_Booking.html` (same constant).
2. **Commit and push**. GitHub Pages picks up the new token within a minute.
3. **Optional**: change the repo URL or rename `Jacksonville_Lab_Manager_Dashboard.html` so old bookmarks 404.

For real authentication (so an ex-employee is locked out the moment you flip a switch), put **Cloudflare Access** in front of the GitHub Pages URL — it's free and ties access to email allow-lists. Whenever you want, I can walk through the setup.

## Where data lives

- **Calendar source of truth**: Teamup, sub-calendar `Med Ed > Jacksonville Lab` (id 13225880)
- **Checked tasks, par counts, cadaver log entries, DSM choices per event**: browser localStorage, key prefix `jaxLabDashboard.v1.*` — *per-device, does not sync*
- **Booking submissions**: written back to Teamup as event title updates

If you want cross-device sync for cadaver log + par counts, that needs a small backend (Cloudflare Workers + KV is the cheapest path). Not built yet.

## Configuration

Top of `Jacksonville_Lab_Manager_Dashboard.html`:

- `ROLES` — defines the 4 roles + what each can see
- `ADMIN_CONFIG` — Teamup keys, lab address, email contact, hidden event titles
- `DSMS` — list of District Sales Managers + cost centers
- `LAB_SUPPLY_CONTACT` — Ana Maria's email for the lab supply par check
- `TASK_TEMPLATES` — per-event-type checklists with role tags + offsets
- `LAB_VARIANTS` — General lab vs 1-Station Sinus Lab kits
- `MEDTRONIC_CATALOG` — 209+ part numbers
- `MEDTRONIC_PAR` — par levels per Medtronic SKU
- `LAB_SUPPLY_PAR` — par levels per non-Medtronic supply
- `EMAIL_TEMPLATES` — per-category, pre/post-event email body templates

All edits are direct text edits in the HTML file — no build step.

## Local development

Just open the HTML files in a browser:

```bash
open Jacksonville_Lab_Manager_Dashboard.html
```

The Teamup API supports CORS so it works fine from `file://`. No server needed.
