# 🌿 Giardino di Palazzo Morpurgo — App pilota (v2)

Web app cartocentrica per la fruizione divulgativa del Giardino di Palazzo Morpurgo a Udine, costruita a costo zero con Leaflet + OpenStreetMap + GitHub Pages.

**Versione 2**: ora i dati cartografici (perimetro + oggetti) sono caricati direttamente dai tuoi GeoJSON esportati da QGIS. Quando aggiungi/modifichi punti nel GIS, basta riesportare e sovrascrivere i file: l'app si aggiorna sola.

---

## 📁 STRUTTURA DEL PROGETTO

```
morpurgo-v2/
├── index.html              ← La pagina web
├── README.md               ← Questo file
├── data/
│   ├── perimetro.geojson   ← Poligono del giardino (da QGIS)
│   └── oggetti.geojson     ← Punti degli oggetti (da QGIS)
└── foto/
    ├── 1.jpg               ← Loggia (numero = COD_ID)
    ├── 5.jpg               ← Ninfa
    ├── A.jpg               ← Aiuole formali
    └── ...
```

---

## 🚀 COME USARLO

### **PASSO 1 — Esporta da QGIS (lo hai già fatto ✓)**

Il sistema attuale legge automaticamente:
- `data/perimetro.geojson` per il poligono del giardino
- `data/oggetti.geojson` per i punti degli oggetti

Devono essere in **EPSG:4326 / WGS84** (i tuoi file attuali sono in `urn:ogc:def:crs:OGC:1.3:CRS84` che è equivalente — perfetto).

### **PASSO 2 — Prepara le foto**

Il file `index.html` cerca le immagini nella cartella `foto/` con nome `{COD_ID}.jpg`. Quindi:

| COD_ID nel GeoJSON | Nome file richiesto |
|---|---|
| `1` (Loggia) | `foto/1.jpg` |
| `5` (Ninfa) | `foto/5.jpg` |
| `A` (Aiuole) | `foto/A.jpg` |

Dimensione consigliata: 800×500 pixel, peso < 200 KB. Usa [squoosh.app](https://squoosh.app) per ottimizzarle.

Se manca una foto, l'app non crasha: nasconde semplicemente l'immagine.

### **PASSO 3 — Testa in locale (con un piccolo trucco)**

⚠️ **Importante**: questa versione **non funziona aprendo `index.html` con doppio clic** perché il browser blocca il caricamento dei GeoJSON locali per motivi di sicurezza (CORS).

Hai 2 opzioni:

**Opzione A — Server locale con Python (raccomandata)**

Apri il terminale nella cartella del progetto e lancia:

```bash
python3 -m http.server 8000
```

Poi apri il browser su `http://localhost:8000`. Funziona tutto.

Per fermare: `Ctrl+C` nel terminale.

**Opzione B — Estensione VS Code "Live Server"**

In VS Code installa l'estensione **Live Server** di Ritwick Dey. Poi clicca col destro su `index.html` → "Open with Live Server". Si apre il browser e tutto funziona.

### **PASSO 4 — Pubblica su GitHub Pages**

Una volta online, il problema CORS non esiste più. Procedura uguale a prima:

1. Crea account su [github.com](https://github.com)
2. **+** in alto → **New repository** → nome `morpurgo` → Public → Create
3. Nel repo clicca **Add file** → **Upload files**
4. Trascina dentro: `index.html`, `README.md`, e le cartelle `data/` e `foto/` (con il loro contenuto)
5. **Commit changes**
6. **Settings** → **Pages** → Source: **Deploy from a branch** → branch `main` → cartella `/(root)` → Save
7. Aspetta 1-2 minuti, vai su **Settings → Pages**: trovi l'URL pubblico tipo `https://tuonome.github.io/morpurgo/`

Quel link è la tua app, accessibile da qualsiasi smartphone con browser.

---

## 🔧 COME L'APP USA I TUOI ATTRIBUTI

Il GeoJSON degli oggetti ha questi campi (li hai impostati tu in QGIS):

| Campo | Come viene usato nell'app |
|---|---|
| `NOME` | Titolo della scheda |
| `COD_ID` | Numero/lettera sul marker; nome del file foto |
| `LIV1` | Categoria principale (etichetta in alto, maiuscoletto) |
| `LIV2` | Categoria intermedia (riga in corsivo) |
| `LIV3` | Sottocategoria (continua la riga di LIV2) |
| `DESC_BREVE` | Testo della scheda (usato se manca descrizione estesa) |
| `SITO`, `COMUNE`, `NOTE` | Non mostrati ma presenti |

**Marker numerici vs lettera**: l'app distingue automaticamente. Numeri → verde scuro (elementi puntuali). Lettere → verde chiaro (ambiti / spazi verdi). Puoi cambiare i colori nel CSS in cima al file.

---

## 📝 DESCRIZIONI ESTESE — DOVE METTERLE

Il problema noto degli shapefile è il **limite di 254 caratteri** per campo testo, che tronca le descrizioni. La soluzione attuale è:

- I tuoi `DESC_BREVE` nel GeoJSON restano come backup
- Le **descrizioni complete** stanno dentro `index.html`, nella sezione `const descrizioniEstese = {...}` (cerca quel testo nel file)
- La chiave è il `COD_ID` dell'oggetto: `"1"` per la loggia, `"5"` per la Ninfa, `"A"` per le aiuole

Per **aggiungere** descrizioni estese a nuovi oggetti, basta aggiungere una riga:

```javascript
const descrizioniEstese = {
  "1": `Una loggia per chiudere il giardino ...`,
  "5": `Una Ninfa al centro dell'acqua ...`,
  "A": `Aiuole formali — perché formali, oggi ...`,
  "3": `LA TUA NUOVA DESCRIZIONE QUI`  // ← nuova
};
```

I backtick (` ` `) permettono di scrivere su più righe con interruzioni naturali.

**Alternativa più strutturata (futuro)**: spostare le descrizioni in un file `descrizioni.json` esterno, così le modifichi senza toccare il codice. Te lo posso preparare quando vuoi.

---

## 🐛 PROBLEMI NOTI DEI TUOI GEOJSON ATTUALI

Una piccola pulizia consigliata per la prossima esportazione da QGIS:

**1. Il campo `FOTO` contiene il codice + nome invece del nome file**

Ora hai: `"FOTO": "5, Scultura \"Ninfa\""`

Sarebbe meglio: `"FOTO": "5.jpg"` (oppure lascia il campo vuoto e l'app prende `COD_ID + .jpg`)

L'app v2 gestisce comunque entrambi i casi: se `FOTO` contiene una virgola, usa `COD_ID + .jpg` come fallback.

**2. Il valore `COMUNE` è "Udine" negli oggetti ma "UDINE" nel perimetro**

Non è un problema per l'app ma è bene uniformare in QGIS.

**3. Manca il campo `id` valorizzato**

Adesso è `null`. Per il pilota va bene, ma quando crescerai converrebbe valorizzarlo (es. progressivo univoco) per gestione futura.

---

## ⚠️ LIMITI ATTUALI

- **GPS impreciso al Morpurgo**: 5-15 m, in centro storico anche peggio. Il pallino blu serve a orientarti, non per "scattare" la scheda dell'oggetto vicino.
- **Niente offline**: serve connessione.
- **Niente multilingua**: solo italiano.
- **Niente analytics**: non sai quanti visitatori hai. Quando vorrai, aggiungiamo Plausible o Umami (gratis, GDPR-friendly).

---

## 💡 PROSSIMI PASSI

Ti propongo, in ordine di priorità:

1. **Aggiungi gli altri oggetti chiave del giardino**: peschiera, cancellata, siepe di tasso, eventuale parterre opposto. In QGIS digitalizzi 3-4 punti nuovi, riesporti.
2. **Carica 3 foto reali** (anche scattate al volo col telefono) per Loggia, Ninfa, Aiuole.
3. **Pubblica su GitHub Pages** e prova sul posto.
4. **Aggiungi audio** per i 3 oggetti (registrati con il telefono, 45 secondi a brano).
5. **Replica per un secondo giardino** (es. Giardino del Torso) per dimostrare la scalabilità.
6. **Presenta al Comune di Udine** con il link pubblico già funzionante.

---

## 📞 SUPPORTO

Per estensioni avanzate (audio, multilingua, AR, dashboard di gestione, sistema di analytics) valuta di partecipare a bandi regionali di digitalizzazione del patrimonio culturale.
