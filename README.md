# Dexfinity · Retention &amp; emailing audity

Live HTML audity účtov klientov — výkonnosť, RFM segmenty, retenčné kohorty, databáza, automatizácie, kalendár kampaní a porovnanie s odvetvovými štandardmi. Dáta priamo z používaných nástrojov (Leadhub, Ecomail, ďalšie).

**Live:** https://dxfnt.github.io/rtn_audit/

---

## Aktívni klienti

| Klient | Nástroj | Folder | Hlavný report |
|---|---|---|---|
| **Liliana.sk** — spodná bielizeň a plavky | Leadhub | `liliana/` | [liliana/index.html](./liliana/) |
| **SCANquilt.sk** — domáce textílie | Leadhub | `scanquilt/` | [scanquilt/index.html](./scanquilt/) |
| **Shapen Barefoot** — barefoot obuv, 7 trhov | Ecomail | `shapen/` | [shapen/emailing-revizia.html](./shapen/emailing-revizia.html) |

---

## Štruktúra repa

```
rtn_audit/
├── README.md                    ← tento súbor
├── index.html                   ← landing page (data-driven, JS array CLIENTS[])
├── liliana/
│   └── index.html               ← single-file audit
├── scanquilt/
│   └── index.html               ← single-file audit
└── shapen/
    ├── index.html               ← interný audit (frequency reframe)
    ├── emailing-revizia.html    ← klientska krátka verzia (A)
    ├── emailing-revizia-full.html ← klientska plná verzia (B)
    ├── revizia-deck.html        ← deck 1920×1080 (keyboard nav)
    └── anastasia-rebuttal.html  ← interný defensive briefing
```

Jednoduchí klienti (1 súbor) — folder len s `index.html`. Komplexnejší klienti (viacero verzií) — folder s viacerými HTML súbormi a entry v landing-u s viacerými link-pillami.

---

## Ako pridať nového klienta

### Krok 1: Pripraviť report

Použi DXFNT brand template (rovnaký ako majú Liliana/SCANquilt/Shapen):

- **Fonts:** Poppins (body) + Questrial (headings), z Google Fonts
- **Farby (CSS variables):**
  - `--dex-navy: #05024E`
  - `--dex-blue: #0065F7`
  - `--dex-teal: #16AB8B`
  - `--dex-orange: #ff7f00`
  - `--dex-light-bg: #F4F6FC`
- **Logo:** `https://www.dexfinity.com/wp-content/uploads/2021/03/Logo-Small-Light-Landscape@2x.png` (light) / `Logo-Small-Dark-Landscape@2x.png` (dark)
- **Tagline:** „Go Beyond" (vždy v footeri)

Najľahší spôsob — skopíruj `liliana/index.html` alebo `shapen/emailing-revizia.html` ako východisko.

### Krok 2: Vytvoriť folder

```bash
mkdir -p ./novyklient
mv myreport.html ./novyklient/index.html
```

Názov folder-u: **lowercase, bez diakritiky, bez medzier** (napr. `mojaznacka`, nie `Moja-Značka`).

### Krok 3: Pridať entry do landing-u

Otvor root `index.html`, nájdi `const CLIENTS = [...]` a pridaj nový objekt:

```js
{
  slug:     "novyklient",                     // názov folder-u
  client:   "Nový klient s.r.o.",             // display name
  industry: "Krátky popis odvetvia",          // 1 riadok
  tool:     "leadhub",                        // "leadhub" | "ecomail" | iný
  accent:   "teal",                           // "blue" | "teal" | "orange" | "navy"
  headline: "Hlavná teza auditu",
  summary:  "1–2 vety popisu pre kartu.",
  stats: [
    { num: "35,1 %", label: "Attribution" },
    { num: "19,1 %", label: "Miera návratu" },
    { num: "4 538",  label: "Zákazníkov (12 m)" },
    { num: "8",       label: "Automatizácií" }
  ],
  collected: "27. 5. 2026",
  links: [
    { href: "./novyklient/", label: "Otvoriť audit", kind: "primary" }
  ]
}
```

**Kind hodnoty pre links:**
- `"primary"` — zvýraznený modrý button (hlavný link)
- `""` — bežný outline pill
- `"internal"` — sivý pill (pre interné materiály, ktoré sa nezdieľajú širšiemu publiku)

### Krok 4: Push do `main`

```bash
git add novyklient/ index.html
git commit -m "Add audit for Nový klient"
git push origin main
```

GitHub Pages publikuje automaticky do ~1 minúty. URL bude:
- Landing: `https://dxfnt.github.io/rtn_audit/`
- Klient: `https://dxfnt.github.io/rtn_audit/novyklient/`

---

## Deploy bez gitu (cez REST API)

Ak nemáš lokálny clone, môžeš pushnúť cez GitHub REST API. PAT je uložený v `Dexfinity_sandbox/KNOWLEDGE/api-keys.env` ako `GITHUB_PAT`:

```bash
GH_TOKEN=$(grep '^GITHUB_PAT=' /Users/user/Documents/Claude/Projects/Dexfinity_sandbox/KNOWLEDGE/api-keys.env | cut -d= -f2)
FILE="/path/to/index.html"
TARGET="novyklient/index.html"

# Check if exists (need SHA for update)
SHA=$(curl -s -H "Authorization: Bearer $GH_TOKEN" \
  "https://api.github.com/repos/DXFNT/rtn_audit/contents/$TARGET" \
  | python3 -c "import sys,json;d=json.load(sys.stdin);print(d.get('sha','NEW'))")

# Push
python3 <<PYEOF
import json, base64, urllib.request
with open("$FILE", "rb") as f:
    content = base64.b64encode(f.read()).decode()
payload = {"message": "Add audit for novyklient", "content": content, "branch": "main"}
if "$SHA" != "NEW": payload["sha"] = "$SHA"
req = urllib.request.Request("https://api.github.com/repos/DXFNT/rtn_audit/contents/$TARGET",
    data=json.dumps(payload).encode(), method="PUT",
    headers={"Authorization": "Bearer $GH_TOKEN", "Accept": "application/vnd.github+json"})
print(json.loads(urllib.request.urlopen(req).read()).get("commit", {}).get("sha", "?")[:10])
PYEOF
```

---

## Mirror na S3

Reporty sú zrkadlené aj na S3 bucket `dexfinity` (eu-central-1) pre prípady keď klient nevie otvoriť GitHub Pages (firewall, atď.).

Cesta na S3: `s3://dexfinity/{klient}/{report}.html` → `https://dexfinity.s3.eu-central-1.amazonaws.com/{klient}/{report}.html`

Upload:
```bash
export PATH="$PATH:$HOME/.local/bin"
aws s3 cp ./novyklient/index.html s3://dexfinity/novyklient/index.html \
  --acl public-read --content-type "text/html; charset=utf-8"
```

---

## Metodika auditov

Každý audit zvyčajne pokrýva tieto oblasti (rozsah podľa nástroja klienta):

1. **Výkonnosť** — attribution, obrat z e-mailov, posledných 7/30 dní + YoY trend
2. **Automatizácie** — všetky scenáre, počet odoslaných, konverzie, ROI per pipeline
3. **Kampane** — top performers, témy ktoré fungujú, frequency a send patterns
4. **RFM segmentácia** — Champions, Loyal, At Risk, Hibernating, ... + revenue per segment
5. **Retenčné kohorty** — návratnosť zákazníkov v 30/60/90/180-dňových oknách
6. **Databáza** — veľkosť, growth, DOI, custom fields, segmenty
7. **Frekvencia a prekryvy** (ak relevantné) — koľko e-mailov dostane priemerný odberateľ
8. **Porovnanie s odvetvím** — Klaviyo, Google, Shopify, MailerLite benchmarks
9. **Akčný plán + harmonogram** — konkrétne úpravy s termínmi a očakávaným dopadom

---

## Brand spec

Všetky reporty musia dodržať DXFNT brand:

- **Typografia:** Poppins (300–700) + Questrial (regular). Google Fonts CDN.
- **Farby:**
  ```css
  --dex-navy: #05024E;     /* Hlavná tmavá */
  --dex-blue: #0065F7;     /* Akcent / primary */
  --dex-teal: #16AB8B;     /* Pozitívne / Win */
  --dex-orange: #ff7f00;   /* Pozor / sezónne */
  --dex-light-bg: #F4F6FC; /* Pozadie sekcií */
  ```
- **Loga:** light verzia na tmavé pozadie, dark verzia na svetlé.
- **Tagline:** „Go Beyond" — v každom footeri + zvyčajne v hero.
- **Spacing:** generous, dychá. Nie tightly packed.
- **Sticky nav** s linkmi na sekcie + brand logo vľavo.
- **Hero** s decorative circles, navy background.
- **Footer** dark navy s Dexfinity logom a Go Beyond tagline.

---

## Kontakt

- Otázky: hello@dexfinity.com
- GitHub: https://github.com/DXFNT/rtn_audit
- Projekt vedie: CreAI oddelenie (Matej Aštary)

**Go Beyond.**
