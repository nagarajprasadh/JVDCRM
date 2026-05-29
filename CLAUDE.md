# Diamond CRM — Project Context

## What this project is
A fully self-contained, single-file browser CRM built for Jay Vijay Diamond Co., Ltd. / Saraff Global, Bangkok. No server, no build step, no dependencies to install. Everything runs from one HTML file.

## Files
- `index.html` — the entire application (HTML + CSS + JS, ~4,500 lines)
- `CLAUDE.md` — this file

## GitHub
- Repo: `https://github.com/nagarajprasadh/JVDCRM`
- Owner: `nagarajprasadh`, Repo: `JVDCRM`
- The file in the repo is `index.html`
- The local working file is also `index.html` (previously `diamond_crm.html` — mapping is now resolved)
- GitHub Pages serves the app at: `https://nagarajprasadh.github.io/JVDCRM`
- Token: stored in env var `GITHUB_TOKEN`

## How to push changes to GitHub
Always use the GitHub REST API (not git CLI) — the token is used directly:
```python
import base64, json, urllib.request, os

TOKEN = os.environ['GITHUB_TOKEN']

# 1. Get current SHA
req = urllib.request.Request(
    "https://api.github.com/repos/nagarajprasadh/JVDCRM/contents/index.html",
    headers={"Authorization": f"token {TOKEN}", "User-Agent": "DiamondCRM-Push"}
)
with urllib.request.urlopen(req) as r:
    sha = json.loads(r.read())['sha']

# 2. Push
with open('index.html', 'rb') as f:
    content_b64 = base64.b64encode(f.read()).decode('utf-8')

payload = json.dumps({
    "message": "Your commit message here",
    "content": content_b64,
    "sha": sha
}).encode('utf-8')

req2 = urllib.request.Request(
    "https://api.github.com/repos/nagarajprasadh/JVDCRM/contents/index.html",
    data=payload, method="PUT",
    headers={"Authorization": f"token {TOKEN}", "Content-Type": "application/json", "User-Agent": "DiamondCRM-Push"}
)
with urllib.request.urlopen(req2) as r:
    d = json.loads(r.read())
    print("SUCCESS:", d['commit']['sha'])
```

## Architecture
- **Data storage**: Browser `localStorage` under key `diamondCRM_v2`
- **Auth**: `sessionStorage` under key `diamondCRM_auth`, cleared on tab close
- **Password hashing**: DJB2 algorithm (`hashPwd` function)
- **No build process**: edit `index.html` directly, push to GitHub

## Default login credentials
| Username | Password | Role |
|----------|----------|------|
| admin | admin1234 | Admin |
| arjun | arjun1234 | User |
| priya | priya1234 | User |
| kanokwan | kanok1234 | User |
| david | david1234 | User |
| nadia | nadia1234 | User |

## Key features implemented
- Customers & Leads — full CRUD, card/table view, filters, bulk select/delete
- Sales Pipeline — 7-stage Kanban board
- Excel/CSV import with duplicate detection
- Business Card Scanner — OCR (Tesseract.js), multi-card image auto-split, handwritten notes
- Photo upload on customer cards with duplicate photo detection (perceptual hash)
- Invoices & Payments — full invoice lifecycle
- Campaigns module
- Reports — Overview, 6 pre-built PDF reports, custom Report Builder with saved configs
- Users & Access — role-based access (Admin / User)
- Settings — VIP thresholds, export/import JSON backup
- Help & FAQ — full embedded user manual
- Last Contacted Date — auto-updates on save, colour-coded freshness badge on cards
- Referred By — free text field, in import template, in Report Builder
- Overdue follow-up filter (>7d / >14d / >30d / >60d / Never)
- Column chooser for table view (19 available columns)
- Hover tooltips on all data elements

## Key JavaScript globals
```javascript
let db = { customers, salespeople, campaigns, businessCards, invoices, savedReports, users, settings }
let currentUser       // set after login
let selectedCusts     // Set of selected customer IDs
let custTableCols     // Array of column keys shown in table view
const ALL_TABLE_COLS  // All available column definitions with labels and tooltips
let custFilter        // { search, status, stage, vip, sp, overdue, view, sort }
let custViewMode      // 'card' | 'table'
```

## External libraries (CDN, no install needed)
- Tesseract.js v5 — OCR for business cards
- PDF.js v3.11.174 — PDF rendering
- SheetJS (XLSX) — Excel import/export
- Playfair Display font (Google Fonts)

## Syntax rules (important)
- Always validate JS with acorn after edits: `node -e "const {parse}=require('./node_modules/acorn'); const src=require('fs').readFileSync('index.html','utf8'); parse(src.slice(src.indexOf('<script>')+8, src.lastIndexOf('</script>')), {ecmaVersion:2022}); console.log('OK');"`
- Never use Unicode em-dash `—` or en-dash `–` inside JavaScript string literals — use `'--'` and `'-'` instead
- Avoid nested template literals more than 2 levels deep in JS
- The `renderReports` function has 3 sub-renderers: `renderReportOverview()`, `renderPdfReportsTab()`, `renderReportBuilder()`

## How to set up acorn for syntax checking
```bash
npm init -y && npm install acorn
```

## Nagaraj's preferences
- Operations, HR and Finance manager at a large fintech company, Bangkok
- Wants all changes pushed to GitHub in the same step as the edit
- Does not want to manually rename files — the index.html mapping is handled in code
- Reports should be PDF-exportable
- Free-text fields preferred over dropdowns where possible
