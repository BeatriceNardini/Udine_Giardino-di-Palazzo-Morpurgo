# 🌿 Giardino di Palazzo Morpurgo — App pilota (v3)

Web app cartocentrica per la fruizione divulgativa del Giardino di Palazzo Morpurgo a Udine.
Tecnologie: Leaflet + OpenStreetMap + GitHub Pages. **Costo: zero.**

---

## 🆕 Novità della v3

| Caratteristica | Come funziona |
|---|---|
| **Pannello informativo** | Premi ⓘ in alto a destra per aprire descrizione complessiva e storia del giardino |
| **Galleria foto generale** | Sezione del pannello informativo con foto del giardino |
| **Galleria foto per oggetto** | Ogni oggetto può avere più foto, mostrate nella scheda completa |
| **Rinumerazione automatica dei marker** | I COD_ID catalografici (es. "5") diventano numeri progressivi a video (1, 2, 3...) |
| **Sottotitolo in corsivo grassetto** | Sotto il titolo dell'oggetto |
| **Mappa sempre visibile** | Anche zoomando oltre il livello dei tile OSM (max nativo 19) la mappa non sparisce |
| **Testi divulgativi separati** | Modificabili senza toccare il GIS (file `descrizioni.json` + `giardino.json`) |

---

## 📁 STRUTTURA DEL PROGETTO

```
morpurgo-v3/
├── index.html
├── README.md
├── data/
│   ├── perimetro.geojson    ← Poligono del giardino (da QGIS)
│   ├── oggetti.geojson      ← Punti degli oggetti (da QGIS)
│   ├── descrizioni.json     ← Testi divulgativi degli oggetti
│   └── giardino.json        ← Descrizione + storia + galleria del giardino
└── foto/
    ├── giardino/            ← Foto generali del giardino
    └── oggetti/             ← Foto dei singoli elementi
```

---

## 🔧 COME L'APP USA I TUOI DATI

### 1. Dati cartografici (dal GIS)

I file `perimetro.geojson` e `oggetti.geojson` sono **gli esportati dal tuo QGIS**.
Quando aggiungi/sposti un punto, **riesporti il GeoJSON** e l'app si aggiorna sola.

Campi attesi nel GeoJSON degli oggetti:
- `COD_ID`: codice catalografico (può essere numero o lettera)
- `NOME`: nome catalografico
- `LIV1`, `LIV2`, `LIV3`: gerarchia ICCD
- `DESC_BREVE`: descrizione breve (fallback se manca in descrizioni.json)

### 2. Testi divulgativi (file separati)

**`data/descrizioni.json`** — per gli oggetti.
La chiave di ogni voce è il **COD_ID** dell'oggetto.

```json
"5": {
  "titolo": "Scultura \"Ninfa\"",
  "sottotitolo": "Una Ninfa al centro dell'acqua",
  "breve": "Testo breve per il pop-up sulla mappa (1-2 frasi).",
  "completa": "Testo lungo per la scheda 'Leggi tutto'.",
  "galleria": [
    { "file": "ninfa-attuale.jpg", "didascalia": "La Ninfa oggi" },
    { "file": "ninfa-1954.jpg", "didascalia": "Foto storica del 1954" }
  ]
}
```

**`data/giardino.json`** — per il pannello informativo a scomparsa.

```json
{
  "descrizione_completa": "Testo che descrive il giardino oggi...",
  "storia": "Testo che racconta la storia del giardino...",
  "galleria": [
    { "file": "vista-generale.jpg", "didascalia": "Vista d'insieme" }
  ]
}
```

### 3. Foto

- **Foto del giardino in generale** → `foto/giardino/`
- **Foto degli oggetti** → `foto/oggetti/`

Vedi i file `LEGGIMI.txt` nelle due cartelle per dettagli su nomi e formati.

---

## 🔢 RINUMERAZIONE AUTOMATICA — come funziona

Nel GIS i tuoi oggetti hanno COD_ID catalografici (es. `1`, `5`, `12`, `A`, `B`).
Sono **numeri ICCD** che corrispondono alla scheda PG ufficiale.

Sulla mappa l'utente vede invece una **numerazione pulita e progressiva**:
- Gli oggetti puntuali (COD_ID numerico) vengono **rinumerati 1, 2, 3...** in ordine crescente
- Gli ambiti (COD_ID alfabetico) mantengono la lettera (A, B, C...)

**Esempio:** se nel GIS hai punti con COD_ID `1`, `5`, `12`, sulla mappa l'utente vede `1`, `2`, `3`.

I tuoi codici catalografici restano nel GeoJSON, ma non sono mai visibili al pubblico.

---

## 🗺️ MAPPA SEMPRE VISIBILE — come abbiamo risolto

OpenStreetMap fornisce le tile (i quadrati che compongono la mappa) fino allo zoom 19.
Oltre quel livello, le tile non esistono e la mappa sparirebbe.

L'app v3 dice a Leaflet: *"se l'utente zooma oltre il 19, prendi la tile del 19 e ingrandiscila tu"*.
Risultato: la mappa diventa un po' sfocata ai zoom massimi (è inevitabile, le tile sono raster) ma **rimane sempre visibile**.

Lo zoom massimo consentito è 22.

---

## 🚀 PROCEDURA DI TEST E PUBBLICAZIONE

### Test in locale

⚠️ Apertura con doppio clic **non funziona** (CORS blocca i JSON locali).

Apri terminale nella cartella del progetto:

```bash
python3 -m http.server 8000
```

Poi browser su `http://localhost:8000`.

### Pubblicazione su GitHub Pages

1. Account su [github.com](https://github.com) → New repository → `morpurgo` → Public
2. Upload di tutti i file
3. Settings → Pages → Source: Deploy from branch → main → / (root) → Save
4. Aspetta 1-2 minuti → URL pubblico tipo `https://tuonome.github.io/morpurgo/`

---

## ✏️ COME MODIFICARE I TESTI

**Caso 1 — Cambi il nome catalografico, sposti un punto, aggiungi un oggetto**:
- Lavori in QGIS
- Riesporti `oggetti.geojson`
- Sovrascrivi il file
- Commit + push

**Caso 2 — Limi una descrizione divulgativa, cambi un sottotitolo, aggiungi una foto a un oggetto**:
- Apri `data/descrizioni.json`
- Modifichi
- Commit + push

**Caso 3 — Cambi la descrizione complessiva del giardino o la storia**:
- Apri `data/giardino.json`
- Modifichi
- Commit + push

In tutti i casi, dopo il push: 1-2 minuti e il sito pubblico è aggiornato.

---

## 📷 GESTIONE FOTO

### Foto del giardino in generale
Cartella: `foto/giardino/`
Dichiarate in: `data/giardino.json` → sezione `galleria`

### Foto dei singoli oggetti
Cartella: `foto/oggetti/`
Dichiarate in: `data/descrizioni.json` → sezione `galleria` di ogni oggetto

### Cosa appare dove
- **Sulla mappa (scheda pop-up)**: la PRIMA foto della galleria dell'oggetto
- **Nella modale "Leggi tutto"**: tutte le foto, con didascalia
- **Nel pannello ⓘ del giardino**: le foto generali, in griglia

### Foto storiche
Puoi mescolare foto attuali e storiche. Indica la fonte nella didascalia.
**Importante**: per le foto dei Civici Musei di Udine o altre istituzioni serve permesso scritto.

### Formati consigliati
- JPG, sotto 250 KB
- Foto copertina: 800×500 px
- Foto galleria: 600×400 px
- Foto del giardino: 1000×700 px
- Strumenti gratuiti: [squoosh.app](https://squoosh.app), ImageOptim

---

## 💡 PROSSIMI PASSI

1. Aggiungi gli altri oggetti del giardino in QGIS (peschiera, cancellata, siepe di tasso, ecc.)
2. Scatta o procura le foto e mettile nelle cartelle giuste
3. Compila descrizioni.json per ogni nuovo oggetto
4. Pubblica su GitHub Pages
5. Prova sul posto
6. Quando funziona: presenta al Comune di Udine

---

## 🛠️ ESTENSIONI POSSIBILI IN FUTURO

- Audio per ogni oggetto (registrazioni 45-60 secondi)
- Multilingua (italiano, inglese, sloveno, tedesco)
- Modalità "percorso guidato" con sequenza ordinata
- Caccia al tesoro botanica per famiglie
- Analytics privacy-friendly (Plausible, Umami)
- Lightbox per le foto a schermo intero
- Estensione ad altri giardini di Udine
