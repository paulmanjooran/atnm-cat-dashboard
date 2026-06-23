# ATNM CAT Dashboard — GitHub Pages Setup

## Files in this repo
- `index.html` — full dashboard, loads instantly from GitHub CDN

---

## Step 1 — Create GitHub Repo

1. Go to https://github.com/new
2. Repository name: `atnm-cat-dashboard` (or any name)
3. Set to **Private** (recommended — internal HSE data)
4. Click **Create repository**

---

## Step 2 — Upload index.html

1. In the new repo, click **Add file → Upload files**
2. Upload `index.html`
3. Commit message: `Initial CAT dashboard`
4. Click **Commit changes**

---

## Step 3 — Enable GitHub Pages

1. Go to repo **Settings → Pages**
2. Source: **Deploy from a branch**
3. Branch: **main** | Folder: **/ (root)**
4. Click **Save**
5. Wait ~60 seconds — your URL will be:
   `https://YOUR-USERNAME.github.io/atnm-cat-dashboard/`

---

## Step 4 — Update Apps Script doGet()

Replace the existing `doGet()` in your `Code.gs` with:

```javascript
function doGet(e) {
  const action = (e && e.parameter && e.parameter.action) || 'dashboard';
  let result;
  try {
    if (action === 'table') {
      result = { ok: true, data: getTableRecords() };
    } else {
      result = { ok: true, data: getDashboardData() };
    }
  } catch (err) {
    result = { ok: false, error: err.message };
  }
  return ContentService
    .createTextOutput(JSON.stringify(result))
    .setMimeType(ContentService.MimeType.JSON);
}
```

Then **redeploy**: Deploy → Manage Deployments → Edit → New Version → Deploy

**IMPORTANT — Access setting:**
- Who has access: **Anyone** (not "Anyone with Google account")
- This is required for the GitHub Pages HTML to call the API

---

## Step 5 — Update API URL in index.html

In `index.html`, find line:
```javascript
const API = 'https://script.google.com/a/macros/altasnim.com/s/AKfyc...exec';
```
Replace the URL with your current deployed Web App URL if it changed after redeployment.

---

## Step 6 — Google Sites Embed

Use this embed code in Google Sites (Insert → Embed → Embed code):

```html
<iframe
  src="https://YOUR-USERNAME.github.io/atnm-cat-dashboard/"
  width="100%"
  height="950"
  frameborder="0"
  style="border-radius:8px;">
</iframe>
```

---

## Performance Comparison

| Before | After |
|--------|-------|
| 30 seconds | 2–4 seconds |
| Apps Script renders full HTML | GitHub CDN serves HTML instantly |
| Single blocking call | Two async JSON calls (staged) |
| No caching at HTML level | Browser caches HTML |

---

## To Update Dashboard After Sheet Changes

If you update the spreadsheet structure or want fresh data:
1. Open Apps Script → run `clearCache()`
2. Reload the GitHub Pages URL — it will fetch fresh JSON

The dashboard auto-refreshes cached data every 15 minutes via the Apps Script trigger already set up.

---

*Al Tasnim Enterprises LLC | Quality You Can Build On | التسنيم*
*ATNM HSE CAT PDO ODC 2026–2027*
