# Neulinger-vote
Vote for a new Product before launching

# QR Vote Pages (2 Sortimenten)

This folder contains two static pages you can host on **GitHub Pages** (or Netlify, Vercel, etc.).
Each page shows a *Danke* message and fires a background `POST` to your endpoint to count a scan.

## Files
- `vote_lebkuchen.html`
- `vote_plaetzchen.html`
- `assets/style.css`

## 1) Create a Google Sheet + Apps Script Web App (counts scans)
1. Create a new Google Sheet (e.g., `QR_Votes`). Add headers in row 1: `timestamp, sortiment, userAgent, ip`.
2. Tools → **Script editor** (Apps Script), paste this code:

```javascript
function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents || "{}");
    var sheet = SpreadsheetApp.openById("REPLACE_WITH_SHEET_ID").getSheetByName("Tabelle1"); // or your sheet name
    var ip = e.parameter && e.parameter.ip ? e.parameter.ip : "";
    sheet.appendRow([
      new Date(), 
      data.sortiment || "", 
      data.ua || "", 
      ip
    ]);
    return ContentService
      .createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON)
      .setHeader("Access-Control-Allow-Origin", "*")
      .setHeader("Access-Control-Allow-Methods", "POST, OPTIONS")
      .setHeader("Access-Control-Allow-Headers", "Content-Type");
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ ok: false, error: String(err) }))
      .setMimeType(ContentService.MimeType.JSON)
      .setHeader("Access-Control-Allow-Origin", "*");
  }
}

function doOptions() {
  return ContentService
    .createTextOutput("")
    .setMimeType(ContentService.MimeType.TEXT)
    .setHeader("Access-Control-Allow-Origin", "*")
    .setHeader("Access-Control-Allow-Methods", "POST, OPTIONS")
    .setHeader("Access-Control-Allow-Headers", "Content-Type");
}
```

3. Replace `REPLACE_WITH_SHEET_ID` with your Sheet ID (the long ID in the URL).
4. Deploy → **Manage deployments** → **Web app** → *Anyone with the link* → Deploy.
5. Copy the **Web app URL**.

## 2) Wire pages to your endpoint
- In both `vote_*.html`, replace the placeholder
  `REPLACE_WITH_YOUR_GOOGLE_APPS_SCRIPT_WEBAPP_URL`
  with your Web app URL.

## 3) Host on GitHub Pages
1. Create a new public repo (e.g., `qr-vote-pages`) on GitHub.
2. Upload the contents of this folder.
3. Settings → Pages → Build and deployment → Source: *Deploy from a branch* → select `main` / `/root`.
4. The site will be live at: `https://USERNAME.github.io/REPO/`.
   - Your two links:
     - `https://USERNAME.github.io/REPO/vote_lebkuchen.html`
     - `https://USERNAME.github.io/REPO/vote_plaetzchen.html`

## 4) Make QR codes
Use the small Python snippet in Colab (below) to generate QR PNGs for the two URLs:

```python
!pip -q install qrcode[pil]
import qrcode

BASE = "https://USERNAME.github.io/REPO"
urls = {
    "lebkuchen": f"{BASE}/vote_lebkuchen.html",
    "plaetzchen": f"{BASE}/vote_plaetzchen.html"
}
for key, url in urls.items():
    img = qrcode.make(url)
    img.save(f"{key}_qr.png")
    print("Saved", f"{key}_qr.png", "→", url)
```

## 5) (Optional) Live dashboard
Since all scans land in your Google Sheet, you can connect it to:
- Looker Studio (Google Data Studio) for a live pie chart.
- Python/Colab + Plotly for internal dashboards.

## Notes
- The pages will show *Danke* immediately regardless of network or CORS – the `fetch` uses `no-cors` fire-and-forget.
- If you prefer to verify the POST on the page, enable CORS in Apps Script (already set in the code) **and** remove `mode: "no-cors"` from the HTML.
- To reduce duplicate scans, Looker Studio or Python can group by IP + time window.
- You can add UTM tags or GitHub Pages query params if you want to A/B test placements.
