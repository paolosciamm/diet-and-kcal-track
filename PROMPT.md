# Prompt: Kcal & Weight Tracker (single-file HTML)

Agisci come senior product designer e front-end developer. Crea **una singola pagina web `index.html`** (HTML + CSS + JavaScript vanilla, tutto inline salvo Chart.js da CDN, nessun build step, nessun backend) per tracciare le calorie giornaliere e il dimagrimento. Deve funzionare aprendo il file direttamente nel browser, con connessione internet disponibile.

## 1. Persistenza dei dati (JSON come memoria)
- Nessun database: la "memoria" è un file JSON.
- Pulsante **Esporta**: scarica `kcal-tracker-YYYY-MM-DD.json` con tutti i dati.
- Pulsante **Importa**: carica un JSON (anche con drag & drop), lo valida e chiede se **sostituire** o **unire** (in caso di conflitto sullo stesso giorno vince il dato importato, mostra un riepilogo prima di confermare).
- Per comodità, salva anche una copia in `localStorage` come cache automatica, ma il JSON resta la fonte di verità. Mostra un indicatore discreto "modifiche non esportate" e un promemoria di esportare.
- Schema JSON versionato, esempio:

```json
{
  "version": 1,
  "settings": {
    "dailyDeficitGoal": 500,
    "baselineBurn": 1800,
    "includeBaselineInDeficit": true,
    "kcalPerKg": 7000,
    "startWeight": 82,
    "targetWeight": 75,
    "targetDate": null
  },
  "days": {
    "2026-11-14": {
      "breakfast": 350,
      "lunch": 700,
      "dinner": 650,
      "snacks": 200,
      "burned": 450,
      "weight": 80.4,
      "note": ""
    }
  }
}
```

- Un giorno senza dati **non esiste** nel JSON (nessun valore a 0 inventato): "non inserito" è diverso da "0 kcal".

## 2. Sezione "Oggi / Diario"
- Navigazione per data (frecce giorno precedente/successivo, selettore data, pulsante "Oggi"), si può compilare qualsiasi giorno passato.
- Quattro card ben distinte per **Colazione, Pranzo, Cena, Spuntini**, ciascuna con icona e un **unico campo numerico kcal** grande e leggibile (solo kcal, niente nomi di alimenti né macro), pulsanti rapidi +50/+100/+250 e possibilità di digitare una somma (es. `350+120`). Anche gli spuntini hanno una voce unica.
- Card **Sport**: kcal totali bruciate nel giorno.
- Campo **Peso** (kg, 1 decimale) opzionale per il giorno.
- Riepilogo in alto con anello di progresso (stile Apple Activity): **deficit del giorno vs obiettivo di deficit giornaliero**, più assunte, bruciate e metabolismo basale usato. Aggiornamento live mentre si digita, salvataggio automatico.

## 3. Sezione "Grafici"
- Grafico a barre/linee delle **kcal assunte** e **kcal bruciate** per giorno (due serie, colori distinti, legenda, tooltip al tocco con i valori esatti), con in più il deficit giornaliero e la linea dell'obiettivo di deficit.
- Grafico separato del **peso**: traccia **solo le pesate effettivamente inserite**, ciascun punto collocato sul suo giorno nell'asse temporale (nessuna interpolazione sui giorni senza pesata, ma linea di collegamento tra i punti), con linea del peso obiettivo. Ha la stessa finestra temporale e gli stessi controlli espandi/comprimi degli altri grafici.
- Controlli:
  - **Finestra temporale**: 7 / 14 / 30 / 90 giorni / Tutto, più intervallo personalizzato.
  - **Espandi / Comprimi**: bottone per ingrandire il grafico (altezza maggiore/schermo intero) o ridurlo.
  - **Toggle "Solo giorni con dati"**: se attivo mostra solo i giorni in cui è stato inserito qualcosa (asse X compatto), se spento mostra tutti i giorni del periodo con i buchi evidenziati.
  - Pan/zoom opzionale sulla finestra.
- Usa **Chart.js** caricato da CDN (versione fissata, es. `chart.js@4.4.x`, con eventuale adapter date) dando per scontata la connessione internet. Grafici responsive, ottimizzati per schermo verticale di telefono, con tooltip toccabili e tema coerente con dark/grey mode.

## 4. Sezione "Stima dimagrimento"
- Selettore di **finestra variabile** (ultimi 7/14/30 giorni, intervallo personalizzato, dall'inizio).
- Calcola e mostra:
  - Totale kcal assunte, totale kcal bruciate, **deficit calorico** totale e **medio giornaliero** nel periodo (solo sui giorni con dati; indica chiaramente quanti giorni sono stati conteggiati su quanti), confrontato con l'**obiettivo di deficit giornaliero** (scostamento in kcal e in %).
  - **Confronto con i 7000 kcal = 1 kg**: barra/anello di progresso verso il prossimo kg ("hai accumulato 4.350 / 7.000 kcal, 62%"), **kg stimati persi** (deficit ÷ 7000), e una proiezione ("a questo ritmo: ~0,4 kg/settimana, kg obiettivo raggiunto il …").
  - **Confronto stima vs realtà**: kg stimati dal deficit accanto alla variazione di peso realmente misurata nello stesso periodo (prima e ultima pesata nella finestra), con la differenza.
  - Se è impostato un peso obiettivo: kg mancanti e, con data obiettivo opzionale, il deficit giornaliero necessario per rispettarla confrontato con quello attuale. Senza data, mostra solo la data stimata di arrivo al ritmo attuale.
- Se l'opzione `includeBaselineInDeficit` è attiva, il deficit = (**metabolismo basale impostato** + sport) − assunte; se spenta, deficit = bruciate (sport) − assunte come da definizione semplice. Spiega la differenza in un tooltip "i" e rendi il toggle visibile anche in questa sezione.
- Gestisci i casi limite: surplus calorico (mostra "aumento stimato" con colore/segno diversi), periodo senza dati, un solo giorno.

## 5. Impostazioni
Pannello per: **obiettivo di deficit giornaliero (kcal)**, **metabolismo basale (kcal/giorno, modificabile in qualsiasi momento)** con toggle per includerlo nel deficit, peso iniziale, peso obiettivo con **data obiettivo opzionale**, valore kcal/kg (default 7000), cancellazione di tutti i dati (con conferma). Lingua italiana, unità kg e kcal fisse.

## 6. Design (stile Apple, minimalista)
- **Dark mode di default** con tema grigio (grey mode) alternativo e toggle; rispetta `prefers-color-scheme` alla prima apertura.
- Palette sobria su grigi neutri (es. sfondo #000/#1c1c1e, card #2c2c2e), un solo colore d'accento per sezione (arancio = assunte, verde = bruciate, blu = peso), uso del colore solo per dare significato.
- Font di sistema (`-apple-system, SF Pro, Inter, system-ui`), gerarchia tipografica chiara, numeri grandi con `font-variant-numeric: tabular-nums`.
- Card con raggio ampio (16–20px), ombre morbide o bordi sottili, effetto vetro (`backdrop-filter`) sulla barra di navigazione, spaziature generose, transizioni e animazioni sottili (anelli, barre, cambio tab) con rispetto di `prefers-reduced-motion`.
- Navigazione a tab/segmented control: **Diario · Grafici · Stima · Impostazioni**.
- **Pensata per l'uso da telefono** (iPhone, Safari): layout verticale a colonna singola, tab bar in basso, rispetto delle safe area (`env(safe-area-inset-*)`), niente hover-only, target touch ≥ 44px, input numerici con tastiera numerica (`inputmode="decimal"`), meta viewport e `apple-mobile-web-app-capable`. Su desktop basta che resti ben leggibile (colonna centrata, larghezza max ~480px).
- L'import/export JSON usa il download e il file picker standard del browser (compatibile con Safari iOS, es. salvataggio in app File), nessuna File System Access API.
- Accessibilità: contrasto adeguato, focus visibili, label corrette, navigazione da tastiera.

## 7. Qualità del codice e consegna
- Un unico file `index.html`, codice ordinato e commentato a sezioni (stato, storage, calcoli, rendering, UI).
- Funzioni di calcolo pure e separate dal rendering; validazione input (numeri non negativi, date valide, limiti ragionevoli).
- Includi dati demo caricabili con un pulsante ("Carica dati di esempio") per testare grafici e stime.
- Prima di consegnare, verifica nel browser: import/export round-trip, giorni mancanti, finestre diverse, vista mobile (priorità) e desktop, dark e grey mode.
