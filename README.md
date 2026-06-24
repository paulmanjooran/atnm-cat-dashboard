# ATNM HSE CAT Dashboard — PDO ODC 2026–2027

Live competency assessment tracker for Al Tasnim Enterprises LLC, covering five PDO ODC clusters: Amal, Marmul Well Delivery, Marmul Projects, Greater Birba, and Nimr.

**Live dashboard:** https://paulmanjooran.github.io/atnm-cat-dashboard/

---

## Architecture

```
┌─────────────────────┐      JSONP       ┌──────────────────────┐      reads      ┌─────────────────┐
│  GitHub Pages        │  ◄───────────►   │  Google Apps Script   │  ◄───────────►  │  Google Sheet    │
│  index.html          │   (callback)     │  Code.gs (JSON API)   │                 │  CAT 2026–2027   │
│  charts + filters    │                  │  paged + cached       │                 │  5 cluster tabs  │
└─────────────────────┘                  └──────────────────────┘                 └─────────────────┘
```

- **Frontend** (`index.html`) is served instantly from GitHub's CDN — loads in ~3–5 seconds.
- **Backend** (Apps Script) returns pure JSON via JSONP, bypassing CORS for the `altasnim.com` Workspace domain.
- Table data loads in **pages of 1,000 rows** so no single call is too large.
- Dashboard aggregates and table pages are **cached for 30 minutes** (chunked to survive the 100 KB per-key cache limit).

---

## Files in This Repo

| File | Purpose |
|------|---------|
| `index.html` | The full dashboard — KPIs, 7 charts, filters, 4 data tabs, CSV export. Served by GitHub Pages. |
| `README.md` | This file. |

> The Apps Script backend (`Code.gs`, `SheetDashboard.gs`) lives in the bound Google Sheet, **not** in this repo.

---

## How It Works

1. On load, `index.html` calls the Apps Script Web App with `?action=dashboard&callback=...` → gets KPIs, cluster/assessor/gap aggregates → renders all charts.
2. It then calls `?action=table&page=0,1,2...` → stitches the record pages → fills the assessment register table.
3. Filters (cluster, status, criticality, type, gaps, search) recompute all charts and KPIs client-side — instantly, no server round-trip.

---

## Maintenance

### Update the dashboard data
Data is read live from the Google Sheet. To force a refresh of cached data:
1. Open the Sheet → **Extensions → Apps Script**
2. Run `clearCache()`
3. Reload the dashboard URL

Cache also auto-refreshes every 30 minutes via a time-based trigger.

### Change the API endpoint
If the Apps Script Web App is redeployed with a new URL, update the `API` constant near the top of the `<script>` section in `index.html`:
```javascript
const API = 'https://script.google.com/.../exec';
```

### Editing index.html
- ⚠️ Do **not** judge changes from GitHub's preview pane — JSONP is blocked in the sandboxed preview and will always show a load error there.
- Always test on the **live URL** in a normal/incognito browser tab.
- After committing, let **one** GitHub Pages build finish (Actions tab → green ✅) before testing. Rapid commits cancel each other's deploys.

---

## Embedding in Google Sites

Insert → Embed → Embed code:
```html
<iframe
  src="https://paulmanjooran.github.io/atnm-cat-dashboard/"
  width="100%"
  height="1000"
  frameborder="0"
  style="border:none; border-radius:8px; box-shadow:0 2px 12px rgba(0,0,0,0.12);"
  title="ATNM HSE CAT Dashboard"
  loading="lazy"
  allowfullscreen>
</iframe>
```

---

## Apps Script Backend Reference

Key functions in `Code.gs` (in the bound Sheet):

| Function | Role |
|----------|------|
| `doGet(e)` | API entry — returns JSON or JSONP based on `?callback=` |
| `getDashboardData()` | KPIs, cluster/assessor/critical/expat/gap aggregates |
| `getTableRecords(page)` | One page (1,000 rows) of the assessment register |
| `sendWeeklyEmail()` | Branded HTML summary, emailed Monday 08:00 |
| `setupWeeklyTrigger()` | Registers the Monday email trigger (run once) |
| `setupRefreshTrigger()` | Registers the 30-min cache refresh (run once) |
| `clearCache()` | Clears all cached dashboard + table data |

`SheetDashboard.gs` builds an in-spreadsheet visual dashboard (`📊 CAT DASHBOARD` tab) via the `📊 CAT Dashboard` menu.

---

## Data Source

Google Sheet: **ATNM HSE CAT PDO ODC 2026–2027**
Cluster tabs: Amal · Marmul Well Delivery · Marmul Projects · GB · Nimr

Each record tracks: employee, job title, critical position, expat/national, assessor, planned/actual dates, status, competency gaps, improvement plan, and reassessment status.

---

*Al Tasnim Enterprises LLC | Quality You Can Build On | التسنيم*
