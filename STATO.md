# Stato — Asta Fantacalcio

Due app da unire in un tool solo per l'asta su **Fantalab** (8 squadre, 500 crediti,
Classic + modificatore difesa, modificatore gol diverso da Rivoluzione Fantacalcio):
- `index.html` — "Asta Assistant", live durante l'asta (budget, prezzo suggerito, registro)
- `scout.html` — cheat sheet pre-asta (sfoglia/filtra/prezzi attesi)

Uso: solo dal telefono, un dispositivo alla volta (Fantalab è un'altra app, usata da PC).

## Fatto

- **23/09/2026 — Supabase sganciato**: il progetto Supabase ("Asta Fantacalcio",
  `swoibefjqkpgijbzrdtd`) aveva due tabelle (`asta_config`, `asta_movimenti`), entrambe
  vuote e con RLS disattivata (mai usate per un'asta vera). Tabelle eliminate. Rimossi
  client, chiavi e polling da entrambe le app: ora usano solo `localStorage`, condiviso
  tra le due pagine sullo stesso origin. Commit `ab32288`, pushato su GitHub.
  Il progetto Supabase resta attivo (senza tabelle) — cancellarlo del tutto va fatto a
  mano dalla dashboard, nessuno strumento automatico lo fa.

## Aperto — piano di unificazione (deciso per proposte il 23/09/2026)

1. ~~Un'unica app, due schede~~ — **fatto** (sopra, index.html+scout.html uniti).
2. Dati live da Jarvis: leggere via URL grezzo da GitHub `dati/statistiche.json`,
   `dati/titolari.json`, `dati/infortuni.json`, `dati/modello.json` del repo JARVIS
   (pubblici, fuori dal lucchetto) invece di uno scraping proprio. **Non iniziato.**
3. ~~Supabase da decidere~~ — **fatto**, sganciato (sopra).
4. Diversificazione manuale: lista "giocatori già miei altrove" compilata a mano (niente
   collegamento con le rose di Rivoluzione Fantacalcio, che sono in un repo diverso e
   cifrate). **Non iniziato.**
5. Level-up per uso durante un'asta vera: ricerca sempre a fuoco (**fatto**), undo sempre
   a portata (**fatto**, già in index), avviso sfondamento budget di ruolo (**fatto**,
   colpo d'occhio a 3 livelli sopra). Manca ancora: colpo d'occhio "chi comprare ora"
   all'apertura dell'app (oggi si apre sulla lista giocatori del ruolo, non su un
   riepilogo/suggerimento).
6. ~~Default della nuova asta: 8 partecipanti (nomi provvisori), 500 crediti~~ — **fatto**,
   7 avversari placeholder ("Avversario 1"..."Avversario 7", rinominabili/rimovibili) +
   budget 500 di default (8 squadre totali = te + 7 avversari, coerente con Fantalab).

## Decisioni

- Niente sincronizzazione multi-dispositivo: chiarito il 23/09/2026 che le due app si
  usano solo dal telefono, mai insieme a un altro dispositivo — da qui la scelta di
  sganciare Supabase invece di sistemarne le policy di sicurezza.

- **23/09/2026 — architettura dell'unificazione (punto 1 del piano), decisa prima del
  mockup**: oggi le due app hanno due dataset giocatore separati — Live (`index.html`)
  solo quotazione ufficiale, Prepara (`scout.html`) anche FVM, tag, titolarità/infortuni,
  trend — e Prepara è di sola consultazione (legge lo stato ma non scrive). Deciso:
  1. **Un solo dataset giocatore**, quello ricco di oggi di Prepara (FVM, tag, titolarità,
     infortuni, trend): anche il prezzo consigliato di Live lo userà (es. sconto se
     infortunato, premio se titolare fisso), non solo la quotazione ufficiale.
  2. **Un solo posto per agire**: toccare un giocatore in "Prepara" apre la stessa
     schermata di dettaglio/acquisto di "Live" (oggi Prepara è solo consultazione).
  3. **Impostazioni/Registro/Rivali** raggiungibili da entrambe le schede, non solo da
     Live come oggi.
  Prossimo passo: mockup coerente con queste tre decisioni.

- **23/09/2026 — punto 1 implementato (codice vero, non mockup)**: `index.html` e
  `scout.html` uniti in un solo `index.html` (`scout.html` ora è solo un redirect, per
  chi lo avesse salvato). Verificato in locale con un server statico:
  - **Dataset unico**: 532 giocatori, stesso set di id nelle due liste originali (nessun
    disallineamento), ora con tutti i campi (quotazione ufficiale, FVM, titolarità,
    infortuni, trend) in un solo oggetto.
  - **Prezzo consigliato aggiornato**: non più solo quotazione ufficiale. Peso di
    riferimento = media tra quotazione ufficiale e valore atteso da FVM
    (`fvm * budget/1000`); poi sconto se non titolare fisso (`cambio` ×0,85, `riserva`
    ×0,6) e forte sconto se infortunato (×0,4). **I tre moltiplicatori sono un punto di
    partenza ragionevole, non testato su un'asta vera** — da aggiustare quando si vede
    come si comporta.
  - **Scheda azione unica**: toccare un giocatore in Prepara o in Live apre sempre la
    stessa scheda (prezzo consigliato, quotazione, FVM, titolarità, banner infortunio,
    Preso a X / Prezzo diverso / Preso da altri, alternative simili). Provato: acquisto
    da Prepara aggiorna budget/slot/lista; ricerca in Live su un infortunato mostra il
    banner e il prezzo già scontato.
  - **Registro/Rivali/Impostazioni** in header, raggiungibili da entrambe le schede
    (erano già fuori dal contenuto specifico di tab, ora la barra tab sta sotto).
  Non ancora toccato: la grafica (resta quella scura verde/oro originale, il mockup
  liquid glass provato prima resta da valutare a parte); i punti 2, 4, 5, 6 del piano.

- **23/09/2026 — livello serio, prime 4 priorità funzionali** (l'utente ha guardato
  l'app unita e l'ha trovata scomoda, non solo esteticamente: prima di continuare col
  piano abbiamo fermato e riscritto l'interazione). Fatto e verificato dal vivo in
  locale:
  1. **Ricerca globale unica**, sopra le due schede: cerchi un nome da Prepara o da Live
     senza cambiare scheda, i risultati sostituiscono il contenuto e tornano quando
     cancelli. Cambiare scheda pulisce la ricerca.
  2. **Scheda giocatore come overlay** (foglio dal basso, sfondo scurito): non sposta più
     lo scroll della pagina — si apre sopra, si chiude e resti esattamente dove eri.
  3. **Acquisto in un tocco**: ogni riga/card ha un bottone "✓ prezzo" col prezzo
     consigliato già calcolato — un tocco registra l'acquisto senza aprire la scheda.
     Tutta la card resta comunque cliccabile per aprire il dettaglio completo.
  4. **Impostazioni/Registro/Rivali come overlay** identici alla scheda giocatore
     (stesso meccanismo `openSheet`/`closeSheet`), non spingono più la pagina.
  Non ancora fatto: percentuali con slider, colpo d'occhio visivo (avvisi colorati oltre
  al rosso quando sfori), la grafica vera e propria — questi restano il prossimo giro.
  Commit `d4fbcbd`, pushato su GitHub.

- **23/09/2026 — percentuali con slider**: i 4 campi "% budget" per ruolo in Impostazioni
  (Portieri/Difensori/Centrocampisti/Attaccanti) sono ora slider (`type="range"`) invece
  di campi numerici, con il valore mostrato accanto in tempo reale mentre si trascina.
  Il ribilanciamento automatico delle altre tre percentuali (per restare a 100%) avviene
  già durante il trascinamento, non solo al rilascio — pensato per un tocco rapido durante
  un'asta vera. Provato in locale: trascinamento, ribilanciamento live, "Ripristina
  percentuali di partenza". Logica di calcolo invariata, solo l'input.

- **23/09/2026 — colpo d'occhio visivo**: le card ruolo nella barra in basso (e il totale
  crediti) hanno ora un terzo stato oltre a verde (completo) e rosso (sforato): **arancio
  ("tight")** quando il budget rimasto sul ruolo è positivo ma sotto il costo minimo per
  completarlo (somma delle quotazioni ufficiali più basse tra i giocatori ancora liberi di
  quel ruolo, per gli slot che mancano) — un avviso di rischio prima che sia già tardi.
  Nuova funzione `roleFloorCost(role, neededSlots)`. Provato in locale forzando lo stato
  (via console) nei tre casi: verde/arancio/rosso confermati sia sulle card ruolo che sul
  totale in basso a destra.
  Commit `e52e885`, pushato su GitHub.

- **23/09/2026 — default nuova asta (punto 6 del piano)**: `DEFAULT_STATE.participants`
  ora parte con 7 avversari placeholder ("Avversario 1"..."Avversario 7") invece di lista
  vuota — 8 squadre totali contando te, come su Fantalab. Budget 500 era già di default.
  Si applica solo a un'installazione nuova (localStorage vuoto) — "Azzera asta" non tocca
  i partecipanti, solo acquisti/registro. Nomi restano modificabili/rimovibili dal
  pannello Impostazioni come prima. Provato in locale con localStorage pulito.
  Non ancora committato su git.
