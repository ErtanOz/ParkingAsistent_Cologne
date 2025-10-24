# Parkhaus Monitoring — Köln (Open Data)

Ein leichtgewichtiges Web‑Dashboard zur **Echtzeit‑Anzeige der Parkhausbelegung** in Köln. 
Zeigt freie Plätze, Kapazität, Auslastung, Trend und Öffnungsstatus – mit **Suche**, **Sortierung**, **CSV‑Export**, **Demo‑Modus**, **Diagnose/Self‑Tests** und optionaler **LLM‑Kurz­zusammenfassung** (Gemini).

---

## ✨ Features

- **Live‑Daten** der Stadt Köln (`/externe-dienste/open-data/parking.php`)
- **Fallback‑Strategie:** Direkt → eigener CORS‑Proxy → Default‑Proxies
- **Demo‑Modus** mit Beispieldaten (ideal für `file://`, Offline, CORS‑Probleme)
- **UI‑Komfort:** Suche, Sortierung (freie Plätze, Auslastung, Name), „Jetzt aktualisieren“
- **Status‑Badges:** Live‑Status, „letzte Änderung“, Stale‑Hinweise
- **CSV‑Export:** Momentaufnahme der angezeigten Daten
- **Diagnose & Self‑Tests:** Key‑Mapping, CSV‑Checks, Auslastungs‑Berechnung
- **LLM‑Kurzfazit (optional):** Türkische Kurz­einschätzung per Button **„Genel Durum Özeti ✨“**

---

## 🧱 Tech‑Stack

- **Vanilla HTML/CSS/JS**, keine Build‑Schritte
- Responsives Grid & CSS‑Variablen, kein Framework erforderlich
- Fetch + AbortController
- Optional: **Google Gemini** über eigenes Backend/Proxy

---

## 📦 Struktur

```
/
└── index.html    # App (aus dieser Repo)
```

> Lege die Datei als `index.html` ins Repo‑Root, damit GitHub Pages sie direkt ausliefert.

---

## 🚀 Schnellstart (lokal)

**Variante A – Python Mini‑Server**
```bash
# Windows
py -m http.server 8000

# macOS / Linux
python3 -m http.server 8000
```
Dann im Browser: http://localhost:8000

**Variante B – Node http-server**
```bash
npm i -g http-server
http-server -p 8000
```

> Öffnest du `index.html` direkt als `file://…`, blockiert der Browser häufig `fetch` (CORS/Origin). 
> Nutze daher einen lokalen Server **oder** den **Demo‑Modus**.

---

## 🌐 Deployment (GitHub Pages)

1. Repo erstellen, `index.html` ins Root pushen.  
2. **Settings → Pages → Source:** `main` / root aktivieren.  
3. Seite unter `https://<user>.github.io/<repo>/` abrufen.

**Live‑Abruf & CORS:** Wenn der direkte Abruf des Köln‑Endpoints blockiert wird, nutze
- einen **eigenen CORS‑Proxy** (z. B. Serverless/Cloudflare Worker) und trage ihn im UI ein, oder
- den **Demo‑Modus** (automatisches Failover nach 3 Fehlversuchen).

---

## ⚙️ Konfiguration

Im Script‑Teil der `index.html`:
- `ENDPOINT` – Datenquelle (`https://www.stadt-koeln.de/externe-dienste/open-data/parking.php`)
- `REFRESH_MS` – Aktualisierungsintervall (Standard: 60 s)
- `DEFAULT_CORS_PROXIES` – Fallback‑Proxies
- **Gemini**: `apiUrl` zeigt aktuell auf die Google‑API; **nicht** mit Key im Frontend betreiben (siehe Sicherheit).

**Eingabefelder in der UI:**
- Suche (Name/ID), Sortierung, Live/Demo, *Optionaler* CORS‑Proxy, CSV‑Download, Self‑Tests

---

## 🔌 Datenquelle & Normalisierung

Die App normalisiert heterogene Feldnamen automatisch:

- **ID:** `id | identifier | parkhaus_id | number | nr`
- **Name:** `name | bezeichnung | title | description`
- **Frei:** `free | frei | available | aktuell_frei`
- **Kapazität:** `capacity | kapazitaet | gesamtplaetze | totalCapacity`
- **Trend:** `trend | tendenz | tendency`
- **Offen:** `open | state | geoeffnet | status`
- **Adresse:** `address | ort | location`

Berechnete Felder:
- **occupancy** = `1 - (free/capacity)` in %, inkl. Clamping 0–100

CSV‑Export (Semikolon‑getrennt): `id;name;free;capacity;occupancy;trend;isOpen;address`

---

## 🤖 (Optional) Gemini‑Kurzfazit

Der Button **„Genel Durum Özeti ✨“** öffnet ein Modal und erzeugt eine 2–3‑Satz‑Zusammenfassung der Lage (Türkisch).

> **Wichtig:** **Keinen** API‑Key im Frontend ausliefern!  
> Richte einen **Backend‑Proxy** (Serverless/Worker) ein und injiziere den Key als Secret. 
> Das Frontend ruft dann dein Backend (`/api/summary`) statt der Google‑URL auf.

Pseudo‑Beispiel (nicht produktionsreif):
```js
// /api/summary – Serverless‑Funktion
export default async function handler(req, res) {
  const { rows } = await req.json();
  const payload = { /* … baue Gemini‑Request … */ };

  const gRes = await fetch(
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=" + process.env.GEMINI_API_KEY,
    { method: "POST", headers: { "Content-Type": "application/json" }, body: JSON.stringify(payload) }
  );

  res.status(gRes.status).send(await gRes.text());
}
```

---

## 🛠️ Troubleshooting

- **Failed to fetch**: Häufig CORS/Origin oder Offline → lokalen Server starten oder Demo‑Modus nutzen.
- **Keine Live-Daten**: Der städtische Dienst liefert zeitweise keine freien Plätze (Wartungsarbeiten laut stadt-koeln.de). In diesem Fall zeigt das Dashboard Warnungen an – es sind dann leider keine Live-Werte verfügbar.
- **Windows & Python**: Wenn `python` nicht gefunden wird, probiere `py -m http.server 8000` oder installiere Python von https://www.python.org/downloads/
- **Langsame Updates**: `REFRESH_MS` anpassen, Proxy prüfen.

---

## ♿ Barrierefreiheit

- Semantik (`header`, `section`, `article`), ARIA‑Labels, `aria-live` für Toast
- Hoher Kontrast im Dark‑Theme, responsive Karten

---

## 🔒 Sicherheit

- Keine Third‑Party‑Libs im Frontend, keine Cookies.
- **LLM:** API‑Keys immer serverseitig geheim halten (Proxy/Worker).

---

## 🤝 Contributing

- Issues & PRs willkommen.
- Ideen: Kartenansicht (Leaflet/MapLibre), Favoriten/Alerts, i18n (DE/TR/EN‑Switch), LocalStorage‑Persistenz.

---

## 📄 Lizenz

MIT – siehe `LICENSE`.

---

## 📝 Changelog (Kurz)

- **v0.1.0:** Erste Version mit Live/Demo, Suche, Sortierung, CSV, Diagnose/Self‑Tests, optionalem Gemini‑Fazit.
